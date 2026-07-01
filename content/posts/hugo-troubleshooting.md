---
title: "Hugo + PaperMod 新手踩坑全记录（Windows环境）"
date: 2026-06-28T20:00:00+08:00
draft: false
tags: ["Hugo", "PaperMod", "建站教程", "踩坑记录"]
categories: ["技术笔记"]
description: "从零搭建Hugo博客过程中遇到的所有问题、原因分析与解决方案汇总，适配Windows环境与PaperMod主题"
showToc: true
---

## 前言
本文记录了在 Windows 系统上从零搭建 Hugo 静态博客、使用 PaperMod 主题过程中遇到的全部问题与对应解决方案，所有命令和路径均经过实际验证，适合新手参考避坑。

测试环境：
- 操作系统：Windows 11
- Hugo 版本：v0.163.3 Extended
- 博客主题：PaperMod
- 项目路径：`D:\Hugo\hugoWeb\myblog`

---

## 一、站点初始化相关

### 1.1 如何在指定文件夹打开 PowerShell
**问题现象**：不知道怎么在目标文件夹里快速打开终端，每次都要手动 cd 切路径。

**解决方法**：
1. 进入目标文件夹（如 `D:\Hugo\hugoWeb`）
2. 在空白处按住 `Shift` 键，同时点击鼠标右键
3. 选择「在此处打开 PowerShell 窗口」

**备用方法**：直接点击文件夹顶部地址栏，输入 `powershell` 回车，同样可以原地打开终端。

### 1.2 站点目录如何规划更合理
**问题现象**：不知道网站文件放哪里好，直接建在D盘根目录显得杂乱。

**推荐方案**：
```
D:\Hugo\hugoWeb\    # 所有Hugo项目的总目录
└── myblog\         # 第一个博客项目
    ├── content\    # 文章内容
    ├── themes\     # 主题文件
    ├── assets\     # 自定义样式资源
    └── hugo.toml   # 站点配置
```

**优点**：
- 文件集中管理，备份迁移方便
- 后续新建第二个站点只需在同级目录创建，互不干扰
- 避免零散文件散落在磁盘根目录

### 1.3 创建站点的标准命令
```powershell
# 进入总目录后执行
hugo new site myblog

# 进入站点根目录
cd myblog
```

---

## 二、Git 相关问题

### 2.1 fatal: not a git repository
**问题现象**：执行 `git submodule add` 安装主题时报错：
```
fatal: not a git repository (or any of the parent directories): .git
```

**原因**：`git submodule` 是 Git 仓库的附属功能，必须在已初始化的 Git 仓库中才能执行。刚用 `hugo new site` 创建的只是普通文件夹，不属于 Git 仓库。

**解决方法**：先初始化本地 Git 仓库，再安装主题：
```powershell
# 1. 初始化Git仓库（仅需执行一次）
git init

# 2. 再拉取主题
git submodule add https://github.com/adityatelange/hugo-PaperMod themes/PaperMod
```

### 2.2 LF will be replaced by CRLF 警告
**问题现象**：拉取主题时出现警告：
```
warning: in the working copy of '.gitmodules', LF will be replaced by CRLF the next time Git touches it
```

**原因**：
- `LF` 是 Linux/macOS 系统的换行符格式
- `CRLF` 是 Windows 系统的换行符格式
- Git 在 Windows 下会自动做换行符转换，保证跨平台兼容

**结论**：这只是无害提示，**完全不影响功能，不需要任何处理**，主题已经完整下载成功。

---

## 三、配置文件常见报错

### 3.1 languageCode 废弃警告
**问题现象**：新建文章时出现警告：
```
WARN  deprecated: project config key languageCode was deprecated in Hugo v0.158.0
```

**原因**：Hugo v0.158.0 之后，`languageCode` 配置项已被弃用，改用 `locale`。

**解决方法**：打开 `hugo.toml`，将
```toml
languageCode = 'zh-CN'
```
替换为：
```toml
locale = 'zh-CN'
```

### 3.2 TOML 解析错误：expected newline but got U+00E7
**问题现象**：启动 `hugo server` 时报错：
```
ERROR unmarshal failed: toml: expected newline but got U+00E7 'ç'
```

**原因**：TOML 配置文件语法不规范，通常是以下原因之一：
1. 注释行没有加 `#` 开头，中文被当成配置代码解析
2. 使用了中文全角标点符号（引号、等号、括号等）
3. 复制粘贴带入了不可见特殊字符

**解决方法**：
1. 所有注释必须以英文 `#` 开头
2. 所有标点符号使用英文半角
3. 不确定问题位置时，直接全量替换为经过语法校验的干净配置

**基础配置参考模板**：
```toml
# 站点基础配置
baseURL = 'http://localhost:1313/'
locale = 'zh-CN'
title = '我的Hugo博客'
theme = 'PaperMod'
enableRobotsTXT = true
hasCJKLanguage = true
paginate = 5

# 输出格式（搜索功能必需）
[outputs]
  home = ["HTML", "RSS", "JSON"]

# 主题核心参数
[params]
  defaultTheme = "auto"
  showToc = true
  tocOpen = true
  tocSide = "right"
```

---

## 四、PaperMod 功能配置问题

### 4.1 文章目录不显示
**问题现象**：打开文章看不到侧边目录。

**常见原因与解决**：
1. **全局开关未开启**
   在 `hugo.toml` 的 `[params]` 下添加：
   ```toml
   showToc = true
   tocOpen = true
   tocSide = "right"
   ```

2. **文章没有多级标题**
   目录是自动抓取 `## 二级标题`、`### 三级标题` 生成的，如果文章只有一级标题，目录就会是空的。

3. **单篇文章单独关闭了目录**
   检查文章头部配置，确保没有 `showToc: false`。

### 4.2 搜索功能配置
**完整配置步骤**：
1. 在 `hugo.toml` 中开启 JSON 输出（必需）
   ```toml
   [outputs]
     home = ["HTML", "RSS", "JSON"]
   ```

2. 在 `content/` 根目录新建 `search.md`
   ```markdown
   ---
   title: "文章搜索"
   layout: "search"
   url: "/search/"
   placeholder: "输入关键词搜索文章..."
   ---
   ```

3. 在导航菜单添加入口
   ```toml
   [[menu.main]]
     identifier = "search"
     name = "🔍 搜索"
     url = "/search/"
     weight = 5
   ```

**注意**：`layout: "search"` 是核心，调用 PaperMod 内置的搜索模板，缺少会导致页面空白。

### 4.3 文章不显示在首页
**常见原因**：
- 文章头部 `draft: true`，表示草稿状态，默认不会被构建
- 文章放错了目录，必须在 `content/posts/` 下才能被识别为博客文章

**解决方法**：将 `draft` 改为 `false`，确认文件路径正确。

---

## 五、样式自定义问题

### 5.1 自定义 CSS 会不会和主题冲突
**结论**：**不会冲突**。

PaperMod 官方预留了 `assets/css/extended/` 目录，放在这里的 CSS 文件会自动追加加载，属于叠加式生效，不是覆盖主题原有样式。

**核心原理**：
- 主题先加载自身核心样式
- 再加载 `extended` 目录下的所有自定义样式
- 相同属性下自定义样式优先级更高，只会覆盖你明确写的属性
- 主题升级不会影响你的自定义文件

**正确路径**：
```
myblog/
└── assets/
    └── css/
        └── extended/
            └── custom.css
```

### 5.2 方格纸背景效果实现
#### 方案A：全站淡方格背景
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

#### 方案B：仅文章正文方格（笔记本效果）
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

.dark .post-content {
  background-color: var(--theme);
  background-image:
    linear-gradient(rgba(85, 85, 85, 0.4) 1px, transparent 1px),
    linear-gradient(90deg, rgba(85, 85, 85, 0.4) 1px, transparent 1px);
}
```

**参数调节说明**：
- `background-size`：控制格子大小，数值越大格子越大
- `rgba` 第四个值：控制线条深浅，范围 0~1，越小越淡
- `background-color`：控制纸张底色

---

## 六、新手常见认知问题

### 6.1 写文章每次都要在 PowerShell 操作吗
**不需要**。

PowerShell 只用来做三件事：
1. 初始化站点（仅一次）
2. 启动本地预览服务 `hugo server`（开一次即可）
3. 最终打包生成静态文件 `hugo`（发布时用）

日常写文章全程在编辑器（记事本、VS Code）中完成，只要预览服务开着，保存文件后浏览器会自动刷新。

### 6.2 使用开源主题是"偷"吗
**完全不是**。

PaperMod 采用 MIT 开源协议，作者主动公开代码，明确允许所有人免费使用、修改、商用，这是开源社区的正常共享模式，和"盗取"完全无关。

`git submodule` 是官方标准的下载同步方式，比手动下载更规范，后续还能一键升级主题。

### 6.3 可以完全自己设计网站页面吗
**完全可以**。

Hugo 支持两种模式：
1. **套用主题+自定义修改**：新手首选，在成熟主题基础上微调样式
2. **完全自研模板**：不使用任何第三方主题，手写 `layouts/` 目录下的所有 HTML 模板、CSS、JS，100% 掌控页面设计

循序渐进路线：先用成熟主题跑通流程 → 逐步自定义样式 → 熟练后尝试自研组件 → 最终完全自制主题。

---

## 七、常用命令速查
```powershell
# 创建新站点
hugo new site 站点名

# 新建文章
hugo new content posts/文章名.md

# 启动本地预览
hugo server

# 启动预览并显示草稿文章
hugo server -D

# 打包生成静态文件（发布用）
hugo

# 更新主题到最新版本
git submodule update --remote themes/PaperMod
```