---
title: "Hugo + PaperMod 从本地搭建到 GitHub Pages 上线全流程踩坑记录"
date: 2026-07-01T22:00:00+08:00
draft: false
tags: ["Hugo", "PaperMod", "GitHub Pages", "建站踩坑"]
categories: ["技术笔记"]
description: "Windows环境下从零搭建Hugo博客、使用PaperMod主题，到部署GitHub Pages全过程遇到的问题与解决方案汇总"
showToc: true
---

## 前言

最近从零开始用 Hugo + PaperMod 搭建个人博客，从本地环境配置、主题功能调试，到最终部署到 GitHub Pages，一路上踩了不少新手典型坑。

本文把所有遇到的问题、原因分析和最终解决方案完整记录下来，既是自己的复盘笔记，也能给同样在折腾 Hugo 的新手避坑。

**测试环境**

- 操作系统：Windows 11
- Hugo 版本：v0.160+ Extended
- 博客主题：PaperMod
- 部署平台：GitHub Pages
- 项目路径：`D:\Hugo\hugoWeb\myblog`

---

## 一、本地配置文件踩坑

### 1.1 languageCode 废弃警告

**问题现象**

执行 `hugo new content` 新建文章时，出现黄色警告：

```plaintext
WARN deprecated: project config key languageCode was deprecated in Hugo v0.158.0 and will be removed in a future release. Use locale instead.
```

**原因分析**

Hugo 从 v0.158.0 版本开始，弃用了旧的 `languageCode` 配置项，统一改用 `locale` 来设置站点语言。旧写法目前还能运行，但后续版本会彻底移除。

**解决方案**

打开 `hugo.toml`，直接替换配置项：

```toml
# 旧写法（已废弃）
# languageCode = 'zh-CN'

# 新写法
locale = 'zh-CN'
```

### 1.2 TOML 解析报错：expected newline but got U+00E7

**问题现象**

执行 `hugo server` 启动失败，报错：

```plaintext
ERROR command error: failed to load config: "D:\Hugo\hugoWeb\myblog\hugo.toml:12:21": unmarshal failed: toml: expected newline but got U+00E7 'ç'
```

**原因分析**

TOML 配置语法非常严格，出现这个报错基本是以下三种情况之一：

1. 注释行没有加 `#` 开头，中文文字被当成配置代码解析
2. 使用了中文全角的引号、等号、括号等标点
3. 复制粘贴时带入了不可见的特殊乱码字符

**解决方案**

- 所有注释必须以英文半角 `#` 开头
- 所有配置符号都使用英文半角
- 排查不出问题时，直接替换为干净的基础配置，再逐行新增参数

> **避坑心得**：TOML 对格式要求比 YAML 更严格，写配置时尽量不要在一行里混写代码和中文说明。

---

## 二、PaperMod 主题功能踩坑

### 2.1 文章侧边目录不显示

**问题现象**

按照教程开启了目录配置，打开文章却看不到侧边目录，或者目录是空的。

**原因与解决**

目录不显示通常是三个原因，逐一排查即可：

**1. 全局开关没开启**

在 `hugo.toml` 的 `[params]` 下确认添加了配置：

```toml
showToc = true
tocOpen = true
tocSide = "right"
```

**2. 文章没有多级标题**

目录是自动抓取文章内的 `##` 二级标题、`###` 三级标题 生成的。如果整篇文章只有一级标题，目录自然是空的。

解决：给文章补充分层标题结构。

**3. 单篇文章单独关闭了目录**

检查文章头部 Front Matter，确认没有 `showToc: false` 配置。

### 2.2 搜索功能无结果

**问题现象**

创建了搜索页面，但输入关键词没有任何结果。

**原因与解决**

PaperMod 的搜索依赖 JSON 格式的文章索引，最常见的是漏了输出配置：

- 检查 `hugo.toml` 是否开启了 JSON 输出：

```toml
[outputs]
  home = ["HTML", "RSS", "JSON"]
```

这一行是搜索功能的核心，缺失就不会生成索引文件。

- 检查 `content/search.md` 的布局是否正确：

```markdown
---
title: "文章搜索"
layout: "search"
url: "/search/"
---
```

`layout: "search"` 必须正确，否则调用不到内置搜索模板。

### 2.3 自定义 CSS 会和主题冲突吗

**结论：完全不会冲突。**

PaperMod 官方预留了 `assets/css/extended/` 目录，放在这里的 CSS 文件会自动追加加载，属于叠加式生效，不是覆盖主题原有样式。

**加载逻辑：**

1. 先加载主题自身的核心样式
2. 再加载 `extended` 目录下所有自定义样式
3. 相同属性下自定义样式优先级更高，只会覆盖你明确写的属性

这是官方推荐的标准自定义方式，升级主题也不会丢失修改，非常安全。

### 2.4 方格纸背景效果实现

折腾了两种方格纸背景方案，分享给喜欢这种风格的人：

**方案 A：全站淡方格背景**

低调有质感，文章卡片保持纯白，层次感强：

```css
/* 浅色模式 */
body {
  background-color: #fafafa;
  background-image:
    linear-gradient(rgba(200, 200, 200, 0.25) 1px, transparent 1px),
    linear-gradient(90deg, rgba(200, 200, 200, 0.25) 1px, transparent 1px);
  background-size: 24px 24px;
  background-attachment: fixed;
}

/* 保证内容卡片底色纯净 */
.main .post-entry,
.main .post-content,
.main .archives,
.main .search {
  background: var(--theme);
}

/* 暗黑模式适配 */
.dark body {
  background-color: #1a1a1a;
  background-image:
    linear-gradient(rgba(70, 70, 70, 0.35) 1px, transparent 1px),
    linear-gradient(90deg, rgba(70, 70, 70, 0.35) 1px, transparent 1px);
}
```

**方案 B：仅文章正文方格（笔记本效果）**

适合写学习笔记，有手写本的感觉：

```css
.post-content {
  background-color: #ffffff;
  background-image:
    linear-gradient(rgba(210, 210, 210, 0.4) 1px, transparent 1px),
    linear-gradient(90deg, rgba(210, 210, 210, 0.4) 1px, transparent 1px);
  background-size: 20px 20px;
  padding: 24px 28px;
  border-radius: 8px;
  box-sizing: border-box;
}
```

---

## 三、Git 操作踩坑

### 3.1 安装主题报错：不是 Git 仓库

**问题现象**

执行 `git submodule add` 安装 PaperMod 时报错：

```plaintext
fatal: not a git repository (or any of the parent directories): .git
```

**原因分析**

`git submodule` 是 Git 仓库的附属功能，必须在已初始化的 Git 仓库中才能运行。`hugo new site` 只是创建了普通文件夹，不是 Git 仓库。

**解决方案**

先初始化本地仓库，再安装主题：

```powershell
# 初始化Git仓库（仅需执行一次）
git init

# 再拉取主题
git submodule add https://github.com/adityatelange/hugo-PaperMod themes/PaperMod
```

### 3.2 推送报错：src refspec main does not match any

**问题现象**

执行 `git push -u origin main` 时报错：

```plaintext
error: src refspec main does not match any
error: failed to push some refs to 'https://github.com/Shelyosa/Shelyosa.github.io.git'
```

**原因分析**

本地仓库还没有任何提交记录，不存在 `main` 分支，自然无法推送。刚 `git init` 完的空仓库，必须先 `commit` 一次才会生成本地分支。

**解决方案**

按顺序执行完整流程：

```powershell
# 所有文件加入暂存
git add .

# 生成本地提交记录，创建main分支
git commit -m "初始化博客项目"

# 确保本地分支名为main
git branch -M main

# 推送到远程仓库
git push -u origin main
```

### 3.3 Actions 构建失败：找不到主题

**问题现象**

代码推送成功，但 GitHub Actions 构建报错，提示找不到 PaperMod 主题。

**原因分析**

主题是通过子模块（submodule）方式安装的，如果只推送了主仓库代码，没有同步子模块信息，远程构建时就拉不到主题文件。

**解决方案**

本地执行同步后重新推送：

```powershell
git submodule update --init --recursive
git add .gitmodules themes/PaperMod
git commit -m "同步PaperMod主题子模块"
git push
```

同时确保 workflow 配置里开启了子模块拉取：

```yaml
- name: Checkout
  uses: actions/checkout@v4
  with:
    submodules: true
    fetch-depth: 0
```

---

## 四、GitHub Pages 部署踩坑

### 4.1 找不到 Pages 设置入口

**问题现象**

新版 GitHub 界面改版后，Settings 页面内容很长，一开始找不到 Pages 在哪里。

**精准定位路径**

1. 进入仓库，点击顶部导航栏最右侧的 **Settings**
2. 左侧侧边栏往下滚动，在 **Code and automation** 分类下
3. 找到 **Pages** 选项，在 Codespaces 下方

### 4.2 首页正常，所有子页面跳 localhost

**问题现象**

部署完成后，首页 `https://shelyosa.github.io` 能正常打开，但点击任何文章、导航，都会跳转到 `http://localhost:1313/xxx`，显示拒绝连接。

**原因分析**

这是 Hugo 部署最经典的坑：`hugo.toml` 里的 `baseURL` 还是本地地址。

Hugo 生成所有站内链接（导航、文章卡片、目录跳转、图片路径）时，都会以 `baseURL` 为基准拼接。配置写的是 `localhost`，线上所有链接就都指向了本地地址。首页能打开只是因为你手动输入了正确域名。

**解决方案**

修改 `hugo.toml` 为线上正式地址：

```toml
baseURL = 'https://shelyosa.github.io/'
```

⚠️ 末尾的斜杠 `/` 不能少，否则图片、样式、搜索依然会 404。

修改后提交推送，等待 Actions 重新构建即可。

**本地预览技巧**

以后本地开发不用来回改 `baseURL`，直接用启动参数临时指定：

```powershell
hugo server --baseURL=http://localhost:1313/
```

配置文件永久保留线上地址，避免部署时忘记修改。

### 4.3 页面样式错乱、图片 404

如果部署后页面没有样式、图片加载失败，优先检查两点：

- `baseURL` 是否正确，末尾是否带斜杠
- 图片引用路径是否以 `/` 开头，例如 `/images/xxx.jpg`，不要写相对路径

---

## 五、常用命令速查

```powershell
# 创建新站点
hugo new site 站点名

# 新建文章
hugo new content posts/文章名.md

# 启动本地预览
hugo server

# 启动预览并显示草稿文章
hugo server -D

# 本地打包生成静态文件
hugo --minify

# 更新主题到最新版本
git submodule update --remote themes/PaperMod

# 提交代码并推送
git add .
git commit -m "更新内容"
git push
```