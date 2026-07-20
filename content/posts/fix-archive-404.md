+++
title = "Hugo 博客归档页面 404 修复——从问题定位到完美解决"
date = "2026-07-20"
tags = ["Hugo", "PaperMod", "404修复", "归档页面", "建站踩坑"]
categories = ["技术笔记"]
description = "记录 Hugo 博客归档页面出现 404 错误的完整修复过程，包括问题定位、原因分析、解决方案和最终效果。"
draft = false
+++

## 一、问题背景

在完成博客 UI 改造后，点击导航栏的「归档」按钮，页面显示 404 错误，无法访问归档页面。

这个问题看似简单，但涉及 Hugo 的路由机制、布局模板系统和内容组织方式，值得记录下来。

---

## 二、问题现象

### 2.1 导航栏配置

查看 `hugo.toml` 配置文件，归档菜单项配置如下：

```toml
[[menu.main]]
  identifier = "archives"
  name = "归档"
  url = "/archives/"
  weight = 3
```

导航栏显示正常，点击后跳转到 `http://localhost:1313/archives/`。

### 2.2 访问结果

访问 `/archives/` 时，页面显示 404 错误：

```
404 | This is not the page you were looking for
```

---

## 三、问题排查

### 3.1 检查内容目录

首先检查 `content/` 目录下是否有对应的归档页面文件：

```
content/
├── posts/          # 文章目录
├── tools/          # 工具页面
├── about.md        # 关于页面
└── search.md       # 搜索页面
```

**发现问题**：`content/` 目录下没有 `archives.md` 或 `archives/_index.md` 文件！

### 3.2 理解 Hugo 路由机制

Hugo 的路由规则是：

| 内容文件路径 | 生成 URL |
|-------------|---------|
| `content/about.md` | `/about/` |
| `content/search.md` | `/search/` |
| `content/posts/hello.md` | `/posts/hello/` |
| `content/archives.md` | `/archives/` |

**结论**：导航栏配置的 `/archives/` 需要在 `content/` 下有对应的 `archives.md` 文件。

### 3.3 对比其他正常页面

检查正常工作的页面：

```
content/about.md    → /about/    ✓
content/search.md   → /search/   ✓
content/archives.md → /archives/ ✗ (文件不存在)
```

**根因确认**：缺少 `content/archives.md` 文件。

---

## 四、解决方案

### 4.1 创建归档页面文件

在 `content/` 目录下创建 `archives.md`：

```markdown
---
title: "文章归档"
layout: "archives"
description: "按年月汇总所有博文，快速查阅历史笔记"
summary: "archives"
placeholder: "归档页面"
---
```

**关键配置说明**：

| 字段 | 作用 |
|------|------|
| `title` | 页面标题，显示在浏览器标签页和页面顶部 |
| `layout` | 指定使用的布局模板，这里用 `archives` |
| `description` | 页面描述，显示在标题下方 |
| `summary` | 用于搜索功能的摘要标识 |
| `placeholder` | 占位符文本 |

### 4.2 创建归档布局模板

在 `layouts/_default/` 目录下创建 `archives.html`：

```html
{{- define "main" }}

<header class="page-header">
  <h1>{{ .Title }}</h1>
  {{- if .Description }}
  <div class="post-description">{{ .Description }}</div>
  {{- end }}
</header>

{{- $pages := where site.RegularPages "Type" "in" site.Params.mainSections }}
{{- if not .Site.Params.disableArchiveGrouping }}
  {{- $pagesByYear := $pages.GroupByDate "2006" }}
  {{- range $pagesByYear }}
    <div class="archive-year">
      <h2 class="archive-year-title">{{ .Key }} 年 <span class="archive-count">({{ len .Pages }} 篇)</span></h2>
      {{- $pagesByMonth := .Pages.GroupByDate "January" }}
      {{- range $pagesByMonth }}
        <div class="archive-month">
          <h3 class="archive-month-title">{{ .Key }}</h3>
          <ul class="archive-list">
            {{- range .Pages }}
              <li class="archive-item">
                <span class="archive-date">{{ .Date.Format "01-02" }}</span>
                <a href="{{ .Permalink }}" class="archive-link">{{ .Title }}</a>
              </li>
            {{- end }}
          </ul>
        </div>
      {{- end }}
    </div>
  {{- end }}
{{- else }}
  <ul class="archive-list">
    {{- range $pages }}
      <li class="archive-item">
        <span class="archive-date">{{ .Date.Format "2006-01-02" }}</span>
        <a href="{{ .Permalink }}" class="archive-link">{{ .Title }}</a>
      </li>
    {{- end }}
  </ul>
{{- end }}

{{- end }}
```

**模板逻辑说明**：

1. **获取文章列表**：`where site.RegularPages "Type" "in" site.Params.mainSections` 获取所有文章
2. **按年份分组**：`GroupByDate "2006"` 将文章按年份分组
3. **按月份分组**：`GroupByDate "January"` 将每年文章按月份分组
4. **渲染列表**：显示日期和标题，点击跳转到文章详情页

### 4.3 添加归档样式

在 `assets/css/extended/archive.css` 中添加样式：

```css
/* 归档年份标题 */
.archive-year-title {
    font-size: 1.6rem;
    font-weight: 600;
    color: var(--text-1);
    margin-bottom: 1.2rem;
    padding-bottom: 0.6rem;
    border-bottom: 2px solid var(--accent);
    display: inline-block;
}

.archive-count {
    font-size: 0.9rem;
    font-weight: 400;
    color: var(--text-2);
}

/* 归档月份标题 */
.archive-month-title {
    font-size: 1.1rem;
    font-weight: 500;
    color: var(--text-2);
    margin-bottom: 0.8rem;
}

/* 归档列表 */
.archive-list {
    list-style: none;
    padding: 0;
    margin: 0;
}

.archive-item {
    display: flex;
    align-items: baseline;
    padding: 0.5rem 0;
    border-bottom: 1px solid var(--line);
    transition: background 0.2s ease;
}

.archive-item:hover {
    background: var(--accent-soft);
    padding-left: 0.5rem;
    border-radius: var(--radius-sm);
}

.archive-date {
    font-size: 0.85rem;
    color: var(--text-2);
    min-width: 3.5rem;
    font-family: 'JetBrains Mono', monospace;
}

.archive-link {
    color: var(--text-1);
    text-decoration: none;
    font-size: 1rem;
    transition: color 0.2s ease;
}

.archive-link:hover {
    color: var(--accent);
}
```

**样式特点**：

- 年份标题带橙色下划线，突出层级
- 月份标题用灰色弱化，不抢视觉焦点
- 文章列表 hover 时显示橙色背景，增强交互感
- 日期用等宽字体，对齐更整齐

### 4.4 确认 CSS 引入

检查 `custom.css` 是否引入了 `archive.css`：

```css
@import url("fonts.css");
@import url("code.css");
@import url("quote.css");
@import url("home.css");
@import url("archive.css");  /* ← 已引入 */
@import url("friend.css");
@import url("404.css");
@import url("dark.css");
@import url("banner.css");
@import url("toc.css");
```

确认已引入，无需额外修改。

---

## 五、验证效果

### 5.1 重启 Hugo 服务

```powershell
hugo server
```

### 5.2 访问归档页面

打开浏览器访问 `http://localhost:1313/archives/`，页面正常显示：

```
文章归档
按年月汇总所有博文，快速查阅历史笔记

2026 年 (8 篇)
──────────────
July
  07-20  Hugo 博客 UI 改造——从扁平到精致的 PaperMod 主题美化实战
  07-17  Hugo 博客踩坑记录——样式不一致、语法高亮与 404 问题
  07-01  Git 提交回退与误删恢复完全指南
  07-01  Hugo + PaperMod 从本地搭建到 GitHub Pages 上线全流程踩坑记录

June
  06-28  Hugo 博客部署到 GitHub Pages 完整教程
  06-28  Hugo + PaperMod 新手踩坑全记录（Windows环境）
  06-28  Hugo + PaperMod 新手踩坑全记录（续）
  06-15  Hugo 静态博客网站搭建全过程学习记录
```

### 5.3 功能验证

| 测试项 | 预期结果 | 实际结果 |
|--------|---------|---------|
| 点击导航栏「归档」 | 跳转到归档页面 | ✓ 正常跳转 |
| 页面显示年份分组 | 按年份分组显示 | ✓ 2026 年 (8 篇) |
| 页面显示月份分组 | 每年内按月份分组 | ✓ July、June |
| 点击文章标题 | 跳转到文章详情 | ✓ 正常跳转 |
| 深色模式适配 | 样式正常显示 | ✓ 无异常 |
| 响应式布局 | 窄屏正常显示 | ✓ 自适应 |

---

## 六、经验总结

### 6.1 Hugo 页面创建规则

| 页面类型 | 文件位置 | URL |
|---------|---------|-----|
| 单页 | `content/xxx.md` | `/xxx/` |
| 列表页 | `content/xxx/_index.md` | `/xxx/` |
| 文章页 | `content/posts/xxx.md` | `/posts/xxx/` |
| 自定义布局 | `layouts/_default/xxx.html` | 通过 front matter 的 `layout` 指定 |

### 6.2 常见 404 原因

1. **内容文件缺失**：导航栏配置的 URL 没有对应的内容文件
2. **布局模板缺失**：front matter 指定的 `layout` 没有对应的模板文件
3. **路径拼写错误**：URL 路径与文件路径不匹配
4. **draft 状态**：文章 front matter 中 `draft = true`，生产模式不显示

### 6.3 排查步骤

```
1. 检查导航栏配置的 URL
   ↓
2. 检查 content/ 下是否有对应文件
   ↓
3. 检查文件 front matter 的 layout 字段
   ↓
4. 检查 layouts/ 下是否有对应模板
   ↓
5. 重启 hugo server 验证
```

---

## 七、文件清单

| 文件 | 路径 | 作用 |
|------|------|------|
| `archives.md` | `content/archives.md` | 归档页面内容文件 |
| `archives.html` | `layouts/_default/archives.html` | 归档页面布局模板 |
| `archive.css` | `assets/css/extended/archive.css` | 归档页面样式 |

---

## 八、后续优化建议

1. **添加文章统计**：显示总文章数、总字数、阅读时长
2. **添加标签云**：在归档页面顶部显示热门标签
3. **添加搜索框**：方便快速查找文章
4. **添加年份折叠**：多年文章时，默认折叠旧年份
5. **添加 RSS 订阅**：归档页面提供 RSS 订阅链接

---

## 九、总结

归档页面 404 问题的根因是**缺少内容文件**，解决方案很简单：

1. 创建 `content/archives.md` 内容文件
2. 创建 `layouts/_default/archives.html` 布局模板
3. 添加 `assets/css/extended/archive.css` 样式文件

整个过程不到 10 分钟，但理解 Hugo 的路由机制和布局系统对后续开发很有帮助。

如果你也遇到类似的 404 问题，可以按照本文的排查步骤逐步定位，大部分问题都能快速解决。
