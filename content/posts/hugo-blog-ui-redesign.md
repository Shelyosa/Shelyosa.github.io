+++
title = "Hugo 博客 UI 改造——从扁平到精致的 PaperMod 主题美化实战"
date = "2026-07-20"
tags = ["Hugo", "PaperMod", "UI设计", "CSS", "前端美化"]
categories = ["技术笔记"]
description = "对 Hugo + PaperMod 博客主题进行全面的 UI/UX 改造，解决层次扁平、留白失控、目录简陋等问题，实现卡片分层、滚动高亮、深色模式优化等精致效果。"
draft = false
+++

## 一、改造背景

之前博客用的是 PaperMod 主题的默认样式，虽然简洁，但有几个明显的视觉短板：

- **层次完全扁平**：标题、摘要、正文、分割线全是同一底色，没有卡片分层，视觉糊成一团
- **留白失控**：标题、段落、区块之间间距太小，文字拥挤压抑
- **侧边目录简陋**：纯黑无背景、无缩进区分、无滚动高亮，和主内容割裂
- **文字对比度失衡**：正文纯白刺眼，小标题和正文灰度区分弱
- **分割线生硬**：纯黑横线，没有质感
- **导航栏遮挡内容**：点击目录跳转时，标题被固定导航栏遮住

这次改造的目标是：**在保留 PaperMod 简洁风格的基础上，增加层次感、提升阅读舒适度、完善交互细节。**

---

## 二、改造方案总览

| 模块 | 问题 | 解决方案 |
|------|------|----------|
| 全局配色 | 纯黑底色刺眼 | 浅色暖白 + 深色护眼灰黑 |
| 文章主体 | 无层次感 | 卡片化容器 + 阴影 |
| 标题层级 | 区分度弱 | 一级标题左侧竖线装饰 |
| 分割线 | 生硬黑线 | 渐变淡出效果 |
| 行内代码 | 无样式 | 主题色半透明背景 |
| 表格 | 粗糙 | 圆角 + 表头高亮 |
| 侧边目录 | 简陋无交互 | 卡片化 + 滚动自动高亮 |
| 导航栏 | 磨砂玻璃多余 | 不透明实色 + 边界感 |
| 目录跳转 | 被导航栏遮挡 | `scroll-margin-top` 偏移 |
| 响应式 | 窄屏体验差 | 双栏→单栏自适应 |

---

## 三、全局变量统一

所有配色和尺寸通过 CSS 变量集中管理，浅色和深色模式各一套，方便统一调整。

### 浅色模式（暖白）

```css
:root {
    --bg-page: #fffcfa;          /* 暖白底色 */
    --surface-card: #ffffff;     /* 卡片白色 */
    --text-1: #201a16;           /* 正文深棕黑 */
    --text-2: #6b6560;           /* 辅助灰棕 */
    --accent: #b8773f;           /* 暖橙主色调 */
    --accent-soft: rgba(184, 119, 63, 0.12);
    --main-width: 960px;         /* 统一宽度 */
    --header-height: 64px;       /* 导航栏高度 */
    --gap-block: 2.8rem;         /* 大区块间距 */
    --gap-paragraph: 1.4rem;     /* 段落间距 */
}
```

### 深色模式（护眼灰黑）

```css
[data-theme="dark"] {
    --bg-page: #0d0d10;          /* 深灰黑，非纯黑 */
    --surface-card: #17171c;     /* 卡片深灰 */
    --text-1: #e4e7ed;           /* 柔和浅灰文字 */
    --text-2: #9ca3af;           /* 辅助灰 */
    --accent: #73a9ff;           /* 柔和蓝色主调 */
    --accent-soft: rgba(115, 169, 255, 0.12);
}
```

> **关键决策**：深色模式不用纯黑 `#000`，而是 `#0d0d10` 深灰黑，长时间阅读更护眼。主色调浅色用暖橙、深色用蓝色，各自适配。

---

## 四、文章卡片化分层

### 改造前

文章内容直接贴在页面背景上，没有边界，视觉糊成一团。

### 改造后

给文章主体套一层卡片容器，有背景色、边框、圆角和阴影，立刻产生层次感。

```css
body:not(.list) .post-single {
    margin-top: 1rem;
    background: var(--surface-card);
    border: 1px solid var(--line);
    border-radius: var(--radius-md);
    box-shadow: var(--shadow-card);
    padding: 2rem 2.4rem;
}
```

效果：页面底色 → 文章卡片 → 正文内容，三层明暗区分。

---

## 五、正文排版优化

### 标题装饰

一级标题左侧加竖线装饰，拉开层级感：

```css
.post-content h1 {
    font-size: 1.8rem;
    border-left: 4px solid var(--accent);
    padding-left: 0.9rem;
}
```

### 段落间距

```css
.post-content p,
.post-content li {
    font-size: 17px;
    line-height: 1.75;
    margin-bottom: 1.4rem;
}
```

### 分割线渐变

用渐变替代生硬黑线，两端淡出：

```css
.post-content hr {
    border: none;
    height: 1px;
    background: linear-gradient(90deg, transparent, rgba(23, 19, 17, 0.15), transparent);
    margin: 2.8rem 0;
}
```

### 行内代码

```css
.post-content :not(pre) > code {
    background: var(--accent-soft);
    color: var(--accent);
    padding: 0.2rem 0.5rem;
    border-radius: 5px;
    font-family: 'JetBrains Mono', monospace;
}
```

### 表格美化

```css
.post-content table {
    border-radius: var(--radius-md);
    overflow: hidden;
    background: var(--accent-soft);
}

.post-content th {
    background: rgba(184, 119, 63, 0.15);
    color: var(--accent);
}
```

---

## 六、导航栏修复

### 问题

之前给导航栏加了 `backdrop-filter: blur(12px)` 磨砂玻璃效果，滚动时导航栏下方会出现半透明模糊区域，反而不好看。

### 解决

去掉磨砂效果，改为不透明实色，干净利落：

```css
.header {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    z-index: 9999;
    background: #fffcfa;  /* 不透明实色 */
}
```

### 目录跳转遮挡问题

点击右侧目录跳转时，标题会被固定导航栏遮住。通过 `scroll-margin-top` 解决：

```css
.post-content h1,
.post-content h2,
.post-content h3,
.post-content h4,
.post-content h5,
.post-content h6 {
    scroll-margin-top: calc(var(--header-height) + 1rem);
}
```

这样跳转后标题会停在导航栏下方 `64px + 1rem` 的位置，既能看到标题，导航栏也有清晰的边界感。

---

## 七、侧边目录改造

### 改造前

纯黑背景、无缩进、无高亮、无滚动条美化，和主内容完全割裂。

### 改造后

卡片化容器 + 滚动自动高亮 + 缩进层次 + 滚动条美化。

```css
.toc-container {
    position: fixed;
    right: 20px;
    top: calc(var(--gap) * 3);
    width: 280px;
    max-height: calc(100vh - var(--gap) * 4);
    overflow-y: auto;
    background: var(--surface-card);
    border: 1px solid var(--line);
    border-radius: var(--radius-md);
    padding: 1.2rem;
    box-shadow: 0 3px 14px rgba(0, 0, 0, 0.12);
}
```

### 滚动自动高亮

在 `toc.html` 中添加滚动监听 JS，当页面滚动到某个章节时，自动高亮对应的目录项：

```javascript
(function () {
    var tocLinks = document.querySelectorAll('.toc-container #TableOfContents a');
    var headings = [];
    tocLinks.forEach(function (link) {
        var id = link.getAttribute('href');
        if (id && id.startsWith('#')) {
            var target = document.getElementById(id.substring(1));
            if (target) headings.push({ el: target, link: link });
        }
    });

    function updateActive() {
        var scrollPos = window.scrollY + 120;
        var active = null;
        for (var i = headings.length - 1; i >= 0; i--) {
            if (headings[i].el.offsetTop <= scrollPos) {
                active = headings[i];
                break;
            }
        }
        tocLinks.forEach(function (l) {
            l.parentElement.classList.remove('active');
        });
        if (active) active.link.parentElement.classList.add('active');
    }

    window.addEventListener('scroll', updateActive, { passive: true });
    updateActive();
})();
```

高亮样式：

```css
.toc-container #TableOfContents li.active > a {
    color: var(--accent);
    border-left: 3px solid var(--accent);
    padding-left: 0.6rem;
    font-weight: 500;
}
```

---

## 八、响应式适配

| 屏幕宽度 | 布局策略 |
|----------|----------|
| `> 1440px` | 双栏布局（文章 + 侧边目录） |
| `769px ~ 1100px` | 目录移到文章顶部折叠显示 |
| `< 768px` | 单栏布局，字号缩小，元信息纵向排列 |

```css
@media screen and (max-width: 768px) {
    body:not(.list) .post-single {
        padding: 1.2rem 1rem;
    }
    .post-title {
        font-size: 1.6rem;
    }
    .post-content p,
    .post-content li {
        font-size: 16px;
    }
    .post-meta {
        flex-direction: column;
        gap: 0.4rem;
    }
}
```

---

## 九、效果对比

### 改造前

- 一片死黑，层次模糊
- 标题和正文灰度区分弱
- 目录简陋无交互
- 分割线生硬

### 改造后

- **分层清晰**：页面底色 → 文章卡片 → 正文内容，三层明暗
- **阅读舒适**：17px 字号、1.75 行高、柔和配色
- **导航感强**：目录有底色、缩进、滚动高亮
- **细节精致**：统一圆角、渐变分割线、代码/表格半透明底色
- **交互完善**：目录跳转不遮挡、平滑滚动、hover 过渡动效

---

## 十、文件清单

| 文件 | 路径 | 改动 |
|------|------|------|
| `custom.css` | `assets/css/extended/custom.css` | 全局变量、卡片分层、正文排版、分割线、表格、响应式 |
| `dark.css` | `assets/css/extended/dark.css` | 深色模式全套配色 |
| `toc.css` | `assets/css/extended/toc.css` | 目录卡片化、滚动高亮、缩进层次 |
| `toc.html` | `layouts/partials/toc.html` | 添加滚动监听 JS |

> **重要原则**：所有修改都在 `assets/css/extended/` 目录下完成，**不直接修改主题文件**。主题作为 Git 子模块，直接修改无法同步到远程仓库。

---

## 十一、总结

这次改造的核心思路是：**在 PaperMod 的简洁基础上做加法，而不是推翻重来。**

通过 CSS 变量统一管理配色、卡片化增加层次、`scroll-margin-top` 解决导航遮挡、滚动监听实现目录高亮，用最小的改动量实现了显著的视觉提升。

如果你也在用 Hugo + PaperMod，可以直接把上面的 CSS 复制到你的 `assets/css/extended/` 目录下，重启 `hugo server` 即可看到效果。
