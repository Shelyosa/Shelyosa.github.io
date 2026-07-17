+++
title = "Hugo 博客踩坑记录——样式不一致、语法高亮与 404 问题"
date = "2026-07-17"
tags = ["Hugo", "踩坑记录", "PaperMod", "建站教程"]
categories = ["技术笔记"]
description = "记录 Hugo 博客搭建过程中遇到的三个典型问题：本地与线上样式不一致、代码块语法高亮、新页面 404，以及对应的解决方案。"
draft = false
+++

## 一、问题背景

最近在完善 Hugo 博客的过程中，连续遇到了几个典型问题，记录一下解决过程，方便以后复盘，也给遇到类似问题的朋友一些参考。

---

## 二、问题一：文章容器宽度，本地和线上不一致

### 现象

本地运行 `hugo server` 查看文章，内容区域很宽，看起来舒服；但部署到 GitHub Pages 后，同一篇文章的内容区域变窄了，左右留白很多。

### 排查过程

打开浏览器的开发者工具，检查 HTML 元素的 CSS 变量，发现关键差异：

| CSS 变量 | 本地 | GitHub Pages |
|---|---|---|
| `--main-width` | **960px** | **720px** |

PaperMod 主题通过 CSS 变量 `--main-width` 控制文章主容器的最大宽度。本地是 960px，线上是 720px，说明两处使用的 CSS 不一致。

### 根因

查看 git diff 后发现，之前在 `themes/PaperMod/assets/css/core/theme-vars.css` 中直接修改了 `--main-width` 的值。

**但是！** 主题目录是 Git 子模块（submodule），对主题文件的修改无法被提交到博客仓库。所以 GitHub Pages 部署时使用的仍然是主题原始的 `theme-vars.css`（720px），而本地则是修改后的版本（960px）。

### 解决方案

正确的做法是**不要直接修改主题文件**，而是通过 Hugo 的扩展 CSS 机制来覆盖变量。

在 `assets/css/extended/custom.css` 中添加：

```css
:root {
  --main-width: 960px;
  --nav-width: 1380px;
}
```

然后还原主题文件的修改：

```bash
cd themes/PaperMod
git checkout assets/css/core/theme-vars.css
```

最后重新构建并推送：

```bash
hugo
git add .
git commit -m "fix: 统一文章容器宽度"
git push
```

### 经验总结

> **永远不要直接修改主题目录下的文件**。主题作为子模块，修改无法同步到远程仓库。正确的自定义方式是利用 Hugo 的 `assets/css/extended/` 目录。

---

## 三、问题二：深色模式代码块看不清

### 现象

Markdown 便签本工具在深色模式下，代码块字体灰蒙蒙的，和背景对比度不够，看起来费劲。

### 解决方案

与其手动调颜色，不如直接集成专业的语法高亮库——**highlight.js**。

#### 第一步：引入 CDN

在模板中引入 highlight.js 的核心库和主题样式：

```html
<script src="https://cdn.jsdelivr.net/gh/highlightjs/cdn-release@11.9.0/build/highlight.min.js"></script>
<link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/highlightjs/cdn-release@11.9.0/build/styles/github-dark.min.css" id="hljs-theme">
```

#### 第二步：在 marked 中启用高亮

```javascript
preview.innerHTML = marked.parse(raw, {
  gfm: true,
  breaks: true,
  highlight: function(code, lang) {
    if (lang && hljs.getLanguage(lang)) {
      try {
        return hljs.highlight(code, { language: lang }).value;
      } catch (e) {}
    }
    return hljs.highlightAuto(code).value;
  }
});
```

#### 第三步：明暗主题自动切换

通过 MutationObserver 监听 HTML 的 `data-theme` 属性变化，自动切换 highlight.js 的 CSS 主题：

```javascript
function setHighlightTheme(dark) {
  document.getElementById('hljs-theme').href = dark
    ? 'https://.../github-dark.min.css'
    : 'https://.../github.min.css';
}
```

### 效果对比

| | 之前 | 之后 |
|---|---|---|
| 代码块颜色 | 单色灰白 | 关键字蓝色、字符串绿色、数字黄色 |
| 深色模式 | 对比度不足 | 清晰可辨 |
| 主题切换 | 不支持 | 自动跟随 |

---

## 四、问题三：新页面 404 拒绝访问

### 现象

新建了一个待办清单页面，访问时提示 404，同时 Hugo 本地服务器崩溃退出。

### 排查过程

查看终端输出，发现两个报错：

1. **TOML 解析错误**：`unmarshal failed: toml: expected newline but got...`
2. **Hugo 内部 panic**：`panic: Shift: unknown type *hugolib.pageMetaSource`

### 根因一：layout 配置错误

在 Markdown 文件头（front matter）中，错误地使用了：

```toml
layout = 'tools/todo'   ❌
```

Hugo 中 layout 的值应该是模板文件名（不含扩展名），而 section 路径会自动根据内容所在目录推断。内容文件在 `content/tools/` 下，模板在 `layouts/tools/todo.html`，所以正确的写法是：

```toml
layout = 'todo'   ✅
```

### 根因二：Hugo 热更新 bug

第一次保存文件时 TOML 解析失败，紧接着第二次保存又触发了重构。Hugo 在处理失败状态时发生了 panic 崩溃。这是 Hugo v0.163 的一个已知缺陷——连续文件变更时，第一次失败会导致第二次重建时 panic。

### 解决方案

1. 修正 front matter 中的 layout 字段
2. **重启 Hugo 服务**（这是唯一让 server 恢复的方法）

```bash
# 先停掉旧的，再启动新的
hugo server --bind 0.0.0.0 --baseURL http://localhost:1313/
```

### 经验总结

> 遇到 Hugo server panic 崩溃，先检查 front matter 语法，修正后重启 server 即可。layout 的值只需要写文件名，不要带路径。

---

## 五、本地预览 vs 部署上线

这次还学到了一个重要的工作流程概念：

| 环节 | 命令 | 作用 |
|---|---|---|
| **本地开发** | `hugo server` | 实时预览，改完即看，热更新 |
| **构建静态文件** | `hugo` | 生成 `public/` 目录 |
| **部署上线** | `git push` | 推送到 GitHub，触发 Pages 自动部署 |

本地 `hugo server` 是开发利器，所有修改都可以先在本地验证，满意了再推送到线上，省去反复等待 GitHub Pages 部署的时间。

---

## 六、总结

| 问题 | 根因 | 一句话解法 |
|---|---|---|
| 样式不一致 | 直接修改了主题文件 | 把 CSS 覆盖写到 `assets/css/extended/custom.css` |
| 代码块看不清 | 缺少语法高亮 | 集成 highlight.js |
| 新页面 404 | layout 写错 + Hugo panic | 修正 front matter + 重启 server |

希望这篇记录能帮到同样在用 Hugo + PaperMod 的朋友们。遇到问题不要慌，先看终端日志，再检查文件语法，大部分问题都能定位到根因。
