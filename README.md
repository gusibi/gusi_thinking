# hugo_blog

这是一个基于 [Hugo](https://gohugo.io/) 和 [PaperMod](https://github.com/adityatelange/hugo-PaperMod) 的中文技术博客，使用 Cloudflare Pages 部署。

博客源代码仓库：<https://github.com/gusibi/hugo_blog>

## 一、项目目录和部署原理

Hugo 不需要运行在服务器上。部署时先执行 Hugo，把 Markdown 文章和主题生成静态文件，再把生成结果上传到 Cloudflare Pages：

~~~text
Markdown + PaperMod 主题
              │
              ▼
       hugo --gc --minify
              │
              ▼
          public/
              │
              ▼
       Cloudflare Pages
~~~

本项目的关键配置如下：

| 配置 | 当前值 |
| --- | --- |
| Hugo 配置文件 | `config.toml` |
| 主题 | `PaperMod` |
| 主题来源 | Git submodule：`themes/PaperMod` |
| 构建输出目录 | `public/` |
| 生产分支 | `master` |
| 生产域名 | `https://blog.gusibi.site` |

`public/` 是 Hugo 自动生成的目录，已经被 `.gitignore` 忽略，不需要提交到源代码仓库。Cloudflare Pages 会在每次部署时重新生成它。

## 二、准备环境

需要准备：

1. 一个 GitHub 账号。
2. 一个 Cloudflare 账号。
3. 本机安装 Hugo Extended 版。
4. 本机安装 Git。

检查 Hugo 版本：

~~~bash
hugo version
~~~

当前项目本地使用的版本是 `0.164.0`。如果本地版本不同，建议先安装相同版本，避免本地和 Cloudflare 的构建结果不一致。

## 三、首次获取项目

如果本机还没有项目，执行：

~~~bash
git clone git@github.com:gusibi/hugo_blog.git
cd hugo_blog
git submodule update --init --recursive
~~~

`PaperMod` 是 Git submodule。执行 `git submodule update --init --recursive` 可以确保主题文件被下载，否则 Hugo 可能报错：`theme "PaperMod" not found`。

如果项目已经存在，只需要进入项目目录并同步主题：

~~~bash
cd hugo_blog
git submodule update --init --recursive
~~~

## 四、本地预览博客

启动本地开发服务器：

~~~bash
hugo server
~~~

浏览器打开 <http://localhost:1313>。

如果需要预览草稿文章：

~~~bash
hugo server --buildDrafts
~~~

停止服务器可以在终端按 `Ctrl+C`。

## 五、本地构建检查

在提交到 GitHub 前，先执行一次正式构建：

~~~bash
hugo --gc --minify
~~~

构建成功后，生成的网站文件位于：

~~~text
public/
~~~

也可以用本地服务器检查生成结果：

~~~bash
hugo server
~~~

当前配置使用了较新的 Hugo 版本，构建时可能会看到配置弃用警告。这些警告目前不会阻止构建；只要最后没有以 `Error` 结尾，构建就是成功的。

## 六、提交并推送到 GitHub

修改文章或配置后，先检查变更：

~~~bash
git status
~~~

只添加确认过的文件，不要把本地备份文件和 `public/` 提交进去。例如提交一篇文章：

~~~bash
git add content/zh/post/你的文章文件.md
git commit -m "docs: add a new post"
git push origin master
~~~

如果修改的是 README：

~~~bash
git add README.md
git commit -m "docs: update deployment guide"
git push origin master
~~~

Cloudflare Pages 会监听 GitHub 仓库的 `master` 分支。推送成功后，它会自动开始构建和部署。

## 七、创建 Cloudflare Pages 项目

### 1. 进入 Pages

登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)，进入：

~~~text
Workers & Pages
  → Create application
  → Pages
  → Connect to Git
~~~

不要把 Hugo 当作需要运行 Worker 代码的项目。Hugo 只需要构建静态文件，使用 Pages 即可。

### 2. 连接 GitHub

1. 授权 Cloudflare 访问 GitHub。
2. 选择仓库 `gusibi/hugo_blog`。
3. 选择 **Begin setup** 或 **Begin deployment**。

### 3. 填写构建配置

使用下面的配置：

| Cloudflare 配置项 | 填写内容 |
| --- | --- |
| Project name | 例如 `hugo-blog` |
| Production branch | `master` |
| Root directory | 留空，使用仓库根目录 |
| Framework preset | `Hugo` |
| Build command | `git submodule update --init --recursive && hugo --gc --minify` |
| Build output directory | `public` |

如果当前控制台没有显示 `Hugo` 预设，选择 `None` 或 `Other`，然后手动填写 **Build command** 和 **Build output directory** 即可。Cloudflare Pages 的构建目录必须与 Hugo 的输出目录保持一致。

### 4. 设置 Hugo 版本

在 Pages 项目中进入：

~~~text
Settings
  → Environment variables
~~~

添加以下变量：

~~~text
变量名：HUGO_VERSION
变量值：0.164.0
~~~

Production 和 Preview 环境都建议添加。如果以后升级本地 Hugo，也要同步更新这个变量。

### 5. 开始部署

点击 **Save and Deploy**。

Cloudflare 会依次完成：

1. 拉取 GitHub 仓库。
2. 初始化 `PaperMod` 主题子模块。
3. 安装指定版本的 Hugo。
4. 执行构建命令。
5. 上传 `public/` 目录。
6. 分配一个 `*.pages.dev` 测试地址。

部署完成后，打开 Cloudflare 提供的 `pages.dev` 地址，检查首页、文章、搜索、归档和图片是否正常。

## 八、绑定自己的域名

部署成功后，在 Pages 项目中进入：

~~~text
Custom domains
  → Set up a custom domain
~~~

填写：

~~~text
blog.gusibi.site
~~~

如果 `gusibi.site` 已经托管在 Cloudflare DNS，按照控制台提示确认即可。否则需要在 DNS 服务商处添加指向 Cloudflare Pages 项目地址的 CNAME 记录。

项目根目录中的 `CNAME` 文件可以保留，但 Cloudflare Pages 的自定义域名仍然需要在 Pages 控制台中配置，不能只依赖 `CNAME` 文件。

当前 `config.toml` 中已经配置：

~~~toml
baseURL = "https://blog.gusibi.site"
~~~

因此正式部署时不需要额外修改 `baseURL`。

## 九、以后如何发布文章

1. 在 `content/zh/post/` 下新增或修改文章。
2. 本地执行 `hugo server --buildDrafts` 预览。
3. 发布前确认文章的 front matter 中不是 `draft: true`。
4. 执行本地构建检查：

   ~~~bash
   hugo --gc --minify
   ~~~

5. 提交并推送到 `master`：

   ~~~bash
   git add content/zh/post/你的文章文件.md
   git commit -m "docs: publish a new post"
   git push origin master
   ~~~

6. Cloudflare Pages 自动构建并发布新版本。

如果使用 Pull Request，Cloudflare Pages 还会生成 Preview Deployment，可以先检查预览版本，再合并到 `master`。

## 十、可选：不使用 GitHub，手动部署

如果不想使用 GitHub 自动部署，也可以本地构建后用 Wrangler 上传：

~~~bash
hugo --gc --minify
npx wrangler login
npx wrangler pages deploy public --project-name=hugo-blog
~~~

这种方式每次都需要手动执行部署命令。Git 集成部署和 Direct Upload 是两种不同的 Pages 部署方式，建议首次部署时选择其中一种并持续使用。

## 十一、常见问题

### 1. `theme "PaperMod" not found`

主题子模块没有下载。执行：

~~~bash
git submodule update --init --recursive
~~~

并确认 Cloudflare 的构建命令包含：

~~~bash
git submodule update --init --recursive && hugo --gc --minify
~~~

### 2. 构建成功但页面是 404

检查 **Build output directory** 是否填写为：

~~~text
public
~~~

不要填写 `content`、`themes` 或仓库根目录。

### 3. 推送代码后没有自动部署

检查以下内容：

1. 推送的分支是否是 `master`。
2. Cloudflare Pages 项目是否连接了正确的 GitHub 仓库。
3. Cloudflare Dashboard 的部署记录中是否被设置为暂停自动部署。

### 4. 自定义域名没有生效

确认已经在 Pages 项目的 **Custom domains** 中添加 `blog.gusibi.site`，并检查 DNS 记录是否指向当前 Pages 项目。

### 5. 不要直接使用旧的 `deloy.sh`

当前脚本中使用了 `hugo -t jane`，但项目实际主题是 `PaperMod`；同时它还会把 `public/` 当作单独仓库推送。使用 Cloudflare Pages 时，应让 Pages 直接构建当前源代码仓库，不需要单独维护 `public` 仓库。

## 参考资料

- [Cloudflare Pages：部署 Hugo](https://developers.cloudflare.com/pages/framework-guides/deploy-a-hugo-site/)
- [Cloudflare Pages：构建配置](https://developers.cloudflare.com/pages/configuration/build-configuration/)
- [Cloudflare Pages：Git 集成](https://developers.cloudflare.com/pages/get-started/git-integration/)
- [Cloudflare Pages：自定义域名](https://developers.cloudflare.com/pages/configuration/custom-domains/)
- [Hugo：Host on Cloudflare](https://gohugo.io/host-and-deploy/host-on-cloudflare/)
- [Hugo Quick Start](https://gohugo.io/getting-started/quick-start/)
