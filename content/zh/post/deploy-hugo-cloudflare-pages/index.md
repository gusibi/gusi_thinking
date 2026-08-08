---
date: '2026-08-08T20:30:00+08:00'
lastmod: '2026-08-08T20:30:00+08:00'
draft: false
title: 使用 GitHub Actions 将 Hugo 博客部署到 Cloudflare Pages
slug: deploy-hugo-cloudflare-pages
description: 使用 GitHub Actions 自动构建 Hugo 博客，并通过 Wrangler 部署到 Cloudflare Pages。
tags:
- Hugo
- Cloudflare
- GitHub Actions
- CI/CD
categories:
- 技术
- 工具
comment: false
toc: true
---

我之前使用 Cloudflare Pages 部署 Hugo 博客时，最开始考虑的是直接使用 Cloudflare Pages 的 Git 集成：把 GitHub 仓库连接到 Cloudflare，之后每次推送代码就自动构建。

这种方式很简单，但构建过程完全由 Cloudflare 控制。如果希望自己控制 Hugo 版本、主题子模块初始化、构建命令和部署时机，也可以把整个过程放到 GitHub Actions 中完成。

本文记录我现在采用的方案：

~~~text
push 到 master
      │
      ▼
GitHub Actions
      │
      ├── 拉取源码和 PaperMod 子模块
      ├── 安装 Hugo Extended
      ├── 构建 public/
      └── 使用 Wrangler 部署到 Cloudflare Pages
~~~

## 这套方案适合什么情况？

如果只是想实现“推送代码后自动发布”，Cloudflare Pages 自带的 Git 集成已经够用。

GitHub Actions 更适合下面这些情况：

- 希望固定 Hugo 版本，保证本地和 CI 构建结果一致；
- 需要在部署前加入测试、格式检查或链接检查；
- 希望自己控制哪些分支可以部署；
- 已经把 CI/CD 统一放在 GitHub Actions 中；
- 希望构建过程和 Cloudflare 解耦。

需要注意的是，不要同时开启 Cloudflare Pages 的 Git 自动部署和 GitHub Actions 部署。否则一次 push 可能触发两次构建和两次发布。

## 项目当前情况

这个博客项目有几个和部署相关的配置：

- Hugo 配置文件是 `config.toml`；
- 使用 `PaperMod` 主题；
- `PaperMod` 通过 Git submodule 管理；
- Hugo 构建输出目录是 `public/`；
- 生产分支是 `master`；
- 本地使用 Hugo Extended `0.164.0`；
- 正式域名是 `https://blog.gusibi.site`。

Hugo 的 `public/` 目录是构建产物，不需要提交到仓库。GitHub Actions 每次运行时都会重新生成它。

## 一、准备 Cloudflare Pages 项目

先在 Cloudflare 中创建一个 Pages 项目。项目名可以使用：

~~~text
hugo-blog
~~~

这个名称稍后会写在 GitHub Actions 的部署命令里。

如果从零开始配置，建议创建 Direct Upload 类型的 Pages 项目，因为构建和部署都由 GitHub Actions 完成。

如果已经通过 Cloudflare 的 `Connect to Git` 创建了项目，先关闭 Cloudflare Pages 自带的自动部署，避免和 GitHub Actions 重复部署。

创建项目后，需要先配置自定义域名：

~~~text
blog.gusibi.site
~~~

域名配置在 Cloudflare Pages 项目的 `Custom domains` 中完成，和 GitHub Actions 的构建流程是两件独立的事情。

## 二、准备 Cloudflare API Token

GitHub Actions 运行在 GitHub 的服务器上，不能使用本机的 `wrangler login` 登录状态，因此需要使用 API Token。

在 Cloudflare 中进入：

~~~text
Account API Tokens
→ Create Token
→ Custom Token
~~~

创建 Token 时选择最小权限：

~~~text
Account → Cloudflare Pages → Edit
~~~

同时准备 Cloudflare Account ID。

不要把 Token 写进 YAML 文件，也不要提交到 Git 仓库。Token 应该只放在 GitHub Secrets 中。

## 三、配置 GitHub Secrets

进入 GitHub 仓库：

~~~text
Settings
→ Secrets and variables
→ Actions
~~~

添加以下两个 Repository secrets：

~~~text
CLOUDFLARE_API_TOKEN
CLOUDFLARE_ACCOUNT_ID
~~~

本文使用的 workflow 默认这两个变量已经配置完成。

## 四、创建 GitHub Actions Workflow

在项目根目录创建文件：

~~~text
.github/workflows/pages-deployment.yml
~~~

内容如下：

~~~yaml
name: Deploy Hugo to Cloudflare Pages

on:
  push:
    branches:
      - master
  workflow_dispatch:

permissions:
  contents: read
  deployments: write

jobs:
  deploy:
    name: Build and deploy
    runs-on: ubuntu-latest
    timeout-minutes: 15

    steps:
      - name: Checkout source
        uses: actions/checkout@v6
        with:
          submodules: recursive

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: "0.164.0"
          extended: true

      - name: Build site
        run: hugo --gc --minify

      - name: Deploy to Cloudflare Pages
        uses: cloudflare/wrangler-action@v4
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          command: pages deploy public --project-name=hugo-blog
          gitHubToken: ${{ secrets.GITHUB_TOKEN }}
~~~

如果 Cloudflare Pages 项目名称不是 `hugo-blog`，只需要修改这一行：

~~~yaml
command: pages deploy public --project-name=你的项目名
~~~

## 五、提交并触发部署

将 workflow 提交到 `master`：

~~~bash
git add .github/workflows/pages-deployment.yml
git commit -m "ci: deploy Hugo to Cloudflare Pages"
git push origin master
~~~

推送之后，打开 GitHub 仓库的 `Actions` 页面，可以看到 `Deploy Hugo to Cloudflare Pages` workflow。

它会按以下顺序执行：

1. 拉取博客源代码；
2. 递归拉取 `PaperMod` 主题子模块；
3. 安装 Hugo Extended `0.164.0`；
4. 执行 `hugo --gc --minify`；
5. 生成 `public/` 目录；
6. 使用 Wrangler 上传 `public/`；
7. 创建 Cloudflare Pages 部署记录。

部署成功后，Cloudflare 会把新版本发布到 Pages 项目绑定的域名。

## 六、本地构建和 GitHub Actions 构建保持一致

提交之前，可以先在本机执行：

~~~bash
git submodule update --init --recursive
hugo --gc --minify
~~~

如果本地构建成功，通常 GitHub Actions 也可以顺利构建。

本项目的构建命令是：

~~~bash
hugo --gc --minify
~~~

这里没有使用 `-b $CF_PAGES_URL`，因为 `config.toml` 已经配置了正式域名：

~~~toml
baseURL = "https://blog.gusibi.site"
~~~

这样生成的 canonical URL 和 sitemap 会继续使用正式域名。

## 七、以后如何发布文章

以后发布文章只需要：

~~~bash
git add content/zh/post/你的文章.md
git commit -m "docs: publish a new post"
git push origin master
~~~

GitHub Actions 会自动完成构建和部署。

如果文章还在写作中，可以在 front matter 中保留：

~~~yaml
draft: true
~~~

发布前改为：

~~~yaml
draft: false
~~~

## 常见问题

### 1. `theme "PaperMod" not found`

说明主题子模块没有被拉取。检查 workflow 中是否有：

~~~yaml
with:
  submodules: recursive
~~~

同时确认仓库中存在 `.gitmodules` 和 `themes/PaperMod`。

### 2. `Authentication error` 或 `Unauthorized`

通常是以下原因：

- `CLOUDFLARE_API_TOKEN` 写错；
- `CLOUDFLARE_ACCOUNT_ID` 写错；
- API Token 没有 `Account → Cloudflare Pages → Edit` 权限；
- Token 属于另一个 Cloudflare 账号。

### 3. `Project not found`

检查 workflow 中的项目名：

~~~yaml
--project-name=hugo-blog
~~~

它必须和 Cloudflare Pages 控制台中的项目名完全一致。

### 4. 一次 push 触发两次部署

说明 Cloudflare Pages 的 Git 自动部署和 GitHub Actions 同时开启了。保留 GitHub Actions 后，关闭 Cloudflare Pages 的自动部署。

### 5. Workflow 没有运行

确认：

- workflow 文件位于 `.github/workflows/`；
- 文件扩展名是 `.yml` 或 `.yaml`；
- push 的分支是 `master`；
- GitHub Actions 没有被仓库设置禁用。

## 总结

Hugo 本身只负责把文章生成静态文件，GitHub Actions 负责自动化构建，Wrangler 负责把 `public/` 上传到 Cloudflare Pages。

最终的部署链路是：

~~~text
文章修改
  → git push origin master
  → GitHub Actions
  → Hugo build
  → Wrangler pages deploy
  → Cloudflare Pages
~~~

相关文档：

- [Cloudflare Pages：使用 CI/CD 部署](https://developers.cloudflare.com/pages/how-to/use-direct-upload-with-continuous-integration/)
- [Cloudflare Wrangler Action](https://github.com/cloudflare/wrangler-action)
- [Cloudflare Pages：部署 Hugo](https://developers.cloudflare.com/pages/framework-guides/deploy-a-hugo-site/)
- [GitHub Actions 文档](https://docs.github.com/en/actions)
