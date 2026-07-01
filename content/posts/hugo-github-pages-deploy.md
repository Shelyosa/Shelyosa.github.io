---
title: "Hugo 博客部署到 GitHub Pages 完整教程（GitHub Actions 自动化）"
date: 2026-06-28T21:00:00+08:00
draft: false
tags: ["Hugo", "GitHub Pages", "部署", "GitHub Actions", "自动化"]
categories: ["技术笔记"]
description: "从零开始将 Hugo 博客部署到 GitHub Pages，使用 GitHub Actions 实现自动化构建与部署，包含完整配置与操作步骤"
showToc: true
---

好的，我们来一步步操作，把你的 Hugo 博客部署到 GitHub Pages。这里主要用到 **GitHub Actions** 来实现自动化，你以后更新文章时，只需要把改动推送到 GitHub，网站就会自动重新构建和部署。

### 📝 准备工作

在开始前，请确保你已经有：

1.  **一个 [GitHub 账号](https://github.com/signup)**。
2.  **一个本地的 Hugo 网站项目**，并且已经通过 `hugo server` 在本地预览过，确保一切正常。
3.  **安装了 Git**，并且熟悉基本的 `add`、`commit`、`push` 等命令。

### 🚀 详细部署步骤

整个过程可以分为以下几个核心步骤：

#### **第一步：在 GitHub 上创建一个新的仓库**

1.  登录你的 GitHub 账号，点击页面右上角的 **“+”** 号，选择 **“New repository”**。
2.  在 **“Repository name”** 一栏，**必须**填写为 `你的GitHub用户名.github.io`。
    *   例如，你的用户名是 `john-doe`，仓库名就应该是 `john-doe.github.io`。
3.  将仓库设置为 **Public**（公开）。因为 GitHub Pages 免费版不支持私有仓库部署。
4.  **不要**勾选 “Add a README file”、“Add .gitignore” 或 “Choose a license”，保持仓库完全空白。
5.  点击 **“Create repository”** 完成创建。

#### **第二步：将本地项目与 GitHub 仓库关联**

1.  打开你的终端（命令行），进入本地 Hugo 项目的根目录。
2.  执行以下命令，将本地项目初始化为一个 Git 仓库（如果已经初始化过可以跳过）：
    ```bash
    git init
    ```
3.  将你的 GitHub 仓库添加为本地仓库的远程源（`origin`）：
    ```bash
    git remote add origin https://github.com/你的用户名/你的用户名.github.io.git
    ```

#### **第三步：配置 GitHub Actions 自动化部署**

这是最关键的一步，我们通过 GitHub Actions 创建一个工作流，让它在代码推送后自动构建并部署网站。

1.  在项目的根目录下，创建 GitHub Actions 的配置文件夹：
    ```bash
    mkdir -p .github/workflows
    ```
2.  在该文件夹下创建一个 `hugo.yaml` 文件：
    ```bash
    touch .github/workflows/hugo.yaml
    ```
3.  用代码编辑器打开 `hugo.yaml`，将下面的配置内容**完整复制**进去：

    ```yaml
    name: Build and deploy

    on:
      push:
        branches:
          - main  # 当代码推送到 main 分支时触发
      workflow_dispatch: # 允许你从 GitHub 页面手动触发

    permissions:
      contents: read
      pages: write
      id-token: write

    concurrency:
      group: pages
      cancel-in-progress: false

    defaults:
      run:
        shell: bash

    jobs:
      build:
        runs-on: ubuntu-latest
        env:
          HUGO_VERSION: 0.160.0 # 建议使用你本地安装的 Hugo 版本
        steps:
          - name: Checkout
            uses: actions/checkout@v6
            with:
              submodules: recursive
              fetch-depth: 0

          - name: Setup Hugo
            uses: actions/setup-go@v6
            with:
              go-version: '1.26.1' # 你的 Hugo 版本可能依赖的 Go 版本，可以保持默认

          - name: Setup Pages
            id: pages
            uses: actions/configure-pages@v6

          - name: Build with Hugo
            env:
              HUGO_CACHEDIR: ${{ runner.temp }}/hugo_cache
              HUGO_ENVIRONMENT: production
            run: |
              hugo \
                --minify \
                --baseURL "${{ steps.pages.outputs.base_url }}/"

          - name: Upload artifact
            uses: actions/upload-pages-artifact@v3
            with:
              path: ./public

      deploy:
        environment:
          name: github-pages
          url: ${{ steps.deployment.outputs.page_url }}
        runs-on: ubuntu-latest
        needs: build
        steps:
          - name: Deploy to GitHub Pages
            id: deployment
            uses: actions/deploy-pages@v4
    ```
    > **提示**：你可以根据自己本地 Hugo 的版本，修改 `HUGO_VERSION` 的值。

#### **第四步：推送代码，触发自动部署**

1.  将你刚才新增的 `.github` 文件夹添加到 Git 暂存区：
    ```bash
    git add .github
    ```
2.  提交这次更改：
    ```bash
    git commit -m "Add GitHub Actions workflow for Hugo deployment"
    ```
3.  将所有代码推送到 GitHub 仓库的 `main` 分支：
    ```bash
    git push -u origin main
    ```

#### **第五步：启用并访问你的网站**

1.  推送完成后，回到你的 GitHub 仓库页面。
2.  点击上方的 **“Settings”** 标签，然后在左侧菜单栏找到 **“Pages”**。
3.  在 **“Build and deployment”** 部分，确保 **“Source”** 已经设置为 **“GitHub Actions”**。
4.  稍等片刻（通常1-2分钟），页面会刷新并显示一个链接，类似 `https://你的用户名.github.io/`。点击它，你的网站就上线了！

### 🔄 后续如何更新内容？

以后每次你写完新文章，或者修改了任何文件，只需要在本地执行以下 Git 命令，推送后网站就会自动更新：

```bash
git add .
git commit -m "更新了博客内容"
git push
```

### 💡 一些提示

*   **关于自定义域名**：如果你想使用自己的域名（如 `example.com`），可以在仓库的 `Settings` -> `Pages` 下找到 **“Custom domain”** 进行设置。
*   **关于 Hugo 版本**：上面给的 `hugo.yaml` 示例中 `HUGO_VERSION: 0.160.0` 是我写的示例版本。为了避免本地预览和线上部署效果不一致，建议将配置文件里的版本号改为和你本地安装的 Hugo 版本一致。
*   **关于 PaperMod 主题**：如果你是通过 `git submodule` 添加的主题，那么 `actions/checkout@v6` 这一步中的 `submodules: recursive` 参数会自动拉取主题代码，你无需额外操作。

按照这些步骤操作，你的网站应该很快就能上线了。如果在过程中遇到任何问题，随时可以再来问我。