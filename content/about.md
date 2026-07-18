+++
title = "关于"
date = "2026-07-18"
description = "这个博客的故事——用 Hugo 和 PaperMod 搭建的个人天地"
draft = false
+++

## 📖 博客的由来

这个博客最初的想法很简单：**想有一个属于自己的地方写东西**。

之前用过很多平台——CSDN、掘金、知乎……但它们要么有广告，要么排版不够自由，要么写着写着就觉得不像自己的地盘。于是决定自己动手搭一个。

## 🔧 技术选型

### Hugo — 世界上最快的建站框架

{{< rawhtml >}}
<p style="margin-bottom: 0;">
<a href="https://gohugo.io/" target="_blank" rel="noopener" style="display: inline-flex; align-items: center; gap: 6px;">
  <img src="https://gohugo.io/favicon.ico" width="18" height="18" alt="Hugo" style="margin: 0;">
  <strong>Hugo</strong>
</a>
</p>
{{< /rawhtml >}}

Hugo 是一个用 Go 语言编写的静态网站生成器，号称"世界上最快的建站框架"。选择它的理由：

- **极速编译** — 上千篇文章也能毫秒级生成
- **纯静态** — 不需要服务器、不需要数据库，生成 HTML 文件直接托管
- **Markdown 原生支持** — 写文章就是写 Markdown，专注内容
- **活跃的主题生态** — 社区贡献了大量高质量主题

> 🏗️ Hugo 官网：https://gohugo.io/
>
> 📦 GitHub 仓库：<https://github.com/gohugoio/hugo>

### PaperMod — 简洁优雅的主题

{{< rawhtml >}}
<p style="margin-bottom: 0;">
<a href="https://github.com/adityatelange/hugo-PaperMod" target="_blank" rel="noopener" style="display: inline-flex; align-items: center; gap: 6px;">
  <img src="https://github.com/adityatelange/hugo-PaperMod/raw/master/icons/logo-64x64.png" width="18" height="18" alt="PaperMod" style="margin: 0;">
  <strong>PaperMod</strong>
</a>
</p>
{{< /rawhtml >}}

PaperMod 是 Hugo 生态中最受欢迎的主题之一，以其简洁、快速、可定制著称：

- 极简设计，专注阅读体验
- 支持明暗主题自动切换
- 内置搜索、归档、标签等功能
- 轻量无依赖，开箱即用

> 🔗 PaperMod 主题：<https://github.com/adityatelange/hugo-PaperMod>

## 🚀 搭建历程

整个搭建过程记录在了这几篇文章里：

| 文章 | 内容 |
|---|---|
| [搭建全过程学习记录]({{< relref "/posts/第一篇随笔" >}}) | 从零开始的完整搭建流程 |
| [新手踩坑全记录]({{< relref "/posts/hugo-troubleshooting" >}}) | Windows 环境下的典型问题 |
| [部署到 GitHub Pages]({{< relref "/posts/hugo-github-pages-deploy" >}}) | 使用 GitHub Actions 自动化部署 |
| [踩坑记录（续）]({{< relref "/posts/hugo-troubleshooting-part2" >}}) | 部署、图片、评论系统 |
| [踩坑记录（三）]({{< relref "/posts/hugo-troubleshooting-part3" >}}) | 样式不一致、语法高亮与 404 问题 |

## 🛠️ 工具集

除了写文章，博客里还集成了一些实用小工具：

- **[📝 Markdown 便签本]({{< relref "/tools/markdown" >}})** — 在线 Markdown 实时预览编辑器
- **[✅ 待办清单]({{< relref "/tools/todo" >}})** — 轻量级待办事项管理

---

### 💬 关于我

一名热爱技术的开发者，喜欢捣鼓各种有意思的东西。这个博客会记录技术笔记、踩坑经验，以及一些随想。

如果你有什么想法或建议，欢迎在文章评论区留言交流 🙌
