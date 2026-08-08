---
date: '2026-06-12T15:41:22+08:00'
lastmod: '2026-06-12T15:41:22+08:00'
draft: false
title: 独立开发者免费资源汇总表
slug: indie-dev-free-resources
description: '| 功能分类    | 服务名称            | 链接                                         |
  价格  | 特点           | 一句话介绍                                      '
tags:
- clippings
- 免费资源
categories:
- 资源
comment: false
toc: false
---

## 一、开发工具与服务

| 功能分类    | 服务名称            | 链接                                         | 价格  | 特点           | 一句话介绍                                                                                                                                   | 备注        |
| ------- | --------------- | ------------------------------------------ | --- | ------------ | --------------------------------------------------------------------------------------------------------------------------------------- | --------- |
| 代码托管    | GitHub          | https://github.com                         | $0  | 代码托管、版本控制    | 托管代码和管理版本的主流网站，可以协作开发、提 PR、做 Code Review。 [docs.github](https://docs.github.com/en/get-started/start-your-journey/about-github-and-git) | 基础功能免费    |
| 代码审查    | CodeRabbit      | https://coderabbit.ai                      | $0  | AI 代码审查      | 给你的代码和 PR 做 AI 审查，指出 bug、坏味道并给建议。 [dev](https://dev.to/pullflow/coderabbit-ai-code-reviews-that-ship-code-faster-3nln)                  | 提高代码质量    |
| 网络连接    | Tailscale       | https://tailscale.com                      | $0  | 安全网络连接       | 让多台设备通过互联网组成一个私有局域网，像在同一个 Wi‑Fi 下一样访问服务。 [en.wikipedia](https://en.wikipedia.org/wiki/Tailscale)                                        | 设备间安全通信   |
| 监控告警    | Grafana         | https://grafana.com                        | $0  | 监控和可视化       | 把数据库、日志、监控数据连起来做漂亮的大盘和图表，方便观察服务健康状况。 [devops](https://devops.com/grafana-labs-advances-open-source-visualization-and-observability/)    | 数据监控分析    |
| 产品分析    | PostHog         | https://posthog.com                        | $0  | 产品使用分析       | 统计用户在你网站/应用里的点击、转化、留存，用来优化产品体验。 [youtube](https://www.youtube.com/watch?v=WPjJLpNxI6s)                                                  | 用户行为分析    |
| 图片/视频处理 | Cloudinary      | https://cloudinary.com                     | $0  | 媒体处理、云存储、CDN | 负责存图、剪裁、压缩、加水印并通过 CDN 分发，省掉自己写媒体处理逻辑。 [cloudinary](https://cloudinary.com/guides/automations/media-management-software)                 | 25GB 存储/月 |
| 技术栈     | Next.js         | https://nextjs.org                         | $0  | 全栈开发框架       | 基于 React 的 Web 框架，既能写前端页面也能写 API，支持 SSR、静态导出等。 [vercel](https://vercel.com/frameworks/nextjs)                                           |           |
| ORM     | Drizzle ORM     | https://orm.drizzle.team                   | $0  | 轻量级 ORM      | 在 TS 里用类型安全的方式读写数据库，比直接写 SQL 更安全好维护。 [refine](https://refine.dev/blog/drizzle-react/)                                                   |           |
| 认证      | Better Auth     | https://better-auth.com                    | $0  | 全面鉴权库        | 帮你处理登录、注册、会话等鉴权逻辑，减少自己手写安全相关代码。 [better-auth](https://better-auth.com/docs/comparison)                                                  |           |
| 邮件模板    | React Email     | https://react.email                        | $0  | 邮件模板系统       | 用 React 组件写邮件模板，让邮件 UI 和网站 UI 一样可复用、易维护。                                                                                                |           |
| 文档      | Fumadocs        | https://fumadocs.com                       | $0  | 文档生成         | 帮你基于 MDX/Next.js 快速搭一个美观、可搜索的产品/技术文档站。                                                                                                  |           |
| 国际化     | next-intl       | https://next-intl-docs.vercel.app          | $0  | 国际化支持        | 在 Next.js 里处理多语言文案、路由本地化，让网站轻松支持多种语言。                                                                                                   |           |
| 主题      | next-themes     | https://github.com/pacocoursey/next-themes | $0  | 暗色主题         | 一行配置就能给网站加上深色/浅色主题切换支持。                                                                                                                 |           |
| 分析      | Umami           | https://umami.is                           | $0  | 开源网站分析       | 自己部署的轻量统计工具，看访问量、来源等，又不追踪用户隐私。                                                                                                          |           |
| 分析      | Plausible       | https://plausible.io                       | $0  | 隐私友好分析       | 类似 Google Analytics 的网站统计，但界面更简单、更加注重隐私。                                                                                                |           |
| UI 框架   | Tailwind CSS    | https://tailwindcss.com                    | $0  | CSS 框架       | 提供一堆实用的类名，让你不用写很多自定义 CSS 就能快速搭出页面。 [w3schools](https://www.w3schools.com/git/git_intro.asp)                                             |           |
| UI 组件   | shadcn/ui       | https://ui.shadcn.com                      | $0  | 可定制组件        | 提供一套可复制源码的 React 组件，例如对话框、表单、菜单等，适合做统一设计系统。                                                                                             |           |
| 状态管理    | Zustand         | https://zustand-demo.pmnd.rs               | $0  | 轻量级状态管理      | 管理 React 全局状态的超轻库，比 Redux 简单很多，学习成本低。                                                                                                   |           |
| 数据获取    | TanStack Query  | https://tanstack.com/query                 | $0  | 数据获取和缓存      | 负责管理前端请求状态、缓存和刷新策略，解决“加载中/错误/刷新”等繁琐逻辑。                                                                                                  |           |
| 表单      | React Hook Form | https://react-hook-form.com                | $0  | 表单处理         | 把表单输入、校验、提交封装成 Hook，既高性能又易于和 TS/验证库配合。                                                                                                  |           |
| 类型安全    | TypeScript      | https://www.typescriptlang.org             | $0  | 类型系统         | 在 JS 上增加类型检查，帮助提前发现错误，提升开发体验和可维护性。 [datacamp](https://www.datacamp.com/blog/github-products)                                            |           |
| 验证      | Zod             | https://zod.dev                            | $0  | 模式验证         | 定义数据结构并校验输入是否符合要求，常用于接口入参、表单校验等。                                                                                                        |           |
| 格式化     | Biome           | https://biomejs.dev                        | $0  | Lint 和格式化    | 集成代码格式化和静态检查，一次配置就能统一团队代码风格。                                                                                                            |           |
| 动画      | Framer Motion   | https://www.framer.com/motion              | $0  | 动画库          | 给 React 组件加各种平滑动画效果，比如过渡、拖拽、手势等。                                                                                                        |           |

***

## 二、服务器 / 托管服务

| 服务名称             | 链接                           | 价格           | 配置/额度                | 特点                     | 一句话介绍                                                                                         | 适合场景    |
| ---------------- | ---------------------------- | ------------ | -------------------- | ---------------------- | --------------------------------------------------------------------------------------------- | ------- |
| AWS              | https://aws.amazon.com/free  | $0/年         | 新用户免费套餐              | 送半年+半年 VPS，Mac 实例带 GUI | 全球最大的云平台，新人有免费额度，可以先玩 EC2、S3 等常见服务。 [datacamp](https://www.datacamp.com/blog/github-products) | 最适合起步   |
| 腾讯云              | https://cloud.tencent.com    | ¥20/月        | 轻量服务器                | 预配置 OpenClaw 环境        | 国内访问速度友好、价格合适，用来部署 Web/后端很常见。                                                                 | 国内用户    |
| Hostinger        | https://www.hostinger.com    | $10/月 (~¥70) | VPS                  | 预配置 OpenClaw 环境        | 面向个人站长的便宜 VPS，适合部署博客、小应用等。                                                                    | 海外用户    |
| Vercel           | https://vercel.com           | $0           | 100GB 带宽/月，1M 边缘请求   | Next.js 官方支持           | 专门用来托管前端和全栈框架代码，“连 Git 仓库，一推就上线”。 [vercel](https://vercel.com/frameworks/nextjs)              | 非商业项目   |
| Cloudflare Pages | https://pages.cloudflare.com | $0           | 无限请求和带宽              | 静态网站托管                 | 把静态站点托管在 Cloudflare 边缘节点，访问速度快且基本免费。                                                          | 静态网站    |
| Zeabur           | https://zeabur.com           | $5/月起        | 按量付费                 | 支持多种语言和框架              | 输入 Git 仓库就能部署后端、前端、数据库，很适合独立开发者。                                                              | 容器部署    |
| Railway          | https://railway.app          | $5/月额度       | 按资源使用付费              | 支持 PostgreSQL 和 Docker | 通过 Web 界面快速创建服务和数据库，适合 PoC 和中小项目。                                                             | 灵活 PaaS |
| Fly.io           | https://fly.io               | <$5 免费       | Shared-1x 256MB      | 全球部署                   | 把应用打包成镜像后一键部署到全球多个机房，适合需要全球访问的服务。                                                             | 小额免费    |
| Dokploy          | https://dokploy.com          | 自托管          | 自建 PaaS 平台           | 一键部署、自动备份              | 自己有服务器时，用它搭一个类似 Railway 的“私有云平台”。                                                             | 技术用户    |
| Coolify          | https://coolify.io           | 自托管          | 开源部署平台               | 自托管方案                  | 开源的自托管 PaaS，可以用浏览器操作把项目部署到自家服务器。                                                              | 技术用户    |
| Oracle           | https://cloud.oracle.com     | $0           | 可申请两台免费服务器           | 可申请两台免费服务器             | 公有云服务，服务器可免费更换固定 IP                                                                           | 小额用户    |
| Appwrite         | https://appwrite.io/         | $0           | 5G 带宽，2G 存储 75K 活跃用户 | 免费资源足够                 | Appwrite 是一个面向开发者的开源后端即服务平台，提供认证、数据库、文件存储、云函数和实时通信等能力，帮助更快搭建 Web 和移动应用后端                      | 小额用户    |

***

## 三、AI 模型 / API 服务

| 服务名称            | 链接                            | 价格     | 特点           | 一句话介绍                                     | 稳定性 | 备注          |
| --------------- | ----------------------------- | ------ | ------------ | ----------------------------------------- | --- | ----------- |
| Anyrouter       | https://anyrouter.com         | $0     | 提供 Opus 但不稳定 | 聚合多个大模型的网关，可以统一一个 API 访问不同模型。             | 低   | 适合测试        |
| 公益站（Linux.do 等） | https://linux.do              | 几元/天   | 逆向 Opus      | 社区里有人提供低价大模型 API，一般适合个人体验使用。              | 中   | Linuxdo/闲鱼找 |
| ZenMux          | https://zenmux.com            | $12/月起 | 企业级聚合平台      | 面向企业的多模型接入平台，主打稳定性和统一账单管理。                | 高   | 多模型切换方便     |
| Claude          | https://console.anthropic.com | 按量     | 官方 API       | Anthropic 官方提供的 Claude API，适合正式产品接入和严肃场景。 | 高   | 适合生产环境      |

***

## 四、数据库服务

| 服务名称          | 链接                                             | 价格  | 免费额度            | 特点                    | 一句话介绍                                                      | 备注         |
| ------------- | ---------------------------------------------- | --- | --------------- | --------------------- | ---------------------------------------------------------- | ---------- |
| Supabase      | https://supabase.com                           | $0  | 500MB 存储，5GB 带宽 | PostgreSQL + 实时功能     | “开源 Firebase 替代品”，内置认证、存储、实时订阅等一整套后端能力。                    | 2 个项目限制    |
| Neon          | https://neon.tech                              | $0  | 0.5GB 存储，10 个项目 | Serverless PostgreSQL | 无需自己运维的云 Postgres，按使用量计费，支持分支等现代特性。                        | 分支功能       |
| PlanetScale   | https://planetscale.com                        | $0  | 5GB 存储，10 亿读/月  | MySQL 兼容              | 基于 Vitess 的云 MySQL，适合需要高扩展性又不想管运维的 Web 项目。                 | 独立开发者推荐    |
| TiDB Cloud    | https://tidbcloud.com                          | $0  | 免费 tier         | 分布式数据库                | 云上的分布式 NewSQL 数据库，兼容 MySQL 协议，适合大数据量场景。                    | 企业级特性      |
| DynamoDB      | https://aws.amazon.com/dynamodb                | $0  | 25GB 存储+1GB 传输  | AWS NoSQL             | AWS 的托管 NoSQL 数据库，适合键值和文档型数据。 [github](https://github.com) | AWS 用户     |
| CosmosDB      | https://azure.microsoft.com/products/cosmos-db | $0  | 25GB 存储         | 兼容 MongoDB/PostgreSQL | Azure 上的多模型数据库，支持多种 API 和全球分布。                             | Azure 用户   |
| Cloudflare D1 | https://developers.cloudflare.com/d1           | $0  | 5GB 存储          | SQLite 实现             | Cloudflare 上的云 SQLite 数据库，适合和 Workers 搭配构建边缘应用。            | Workers 集成 |
| Upstash Redis | https://upstash.com                            | $0  | 1 万请求/天，256MB   | Serverless Redis      | 提供按调用计费的 Redis 服务，和 Edge/Serverless 环境很好配合。                | 边缘部署       |

***

## 五、存储与 CDN

| 服务名称          | 链接                                     | 价格  | 免费额度                  | 特点          | 一句话介绍                               | 备注     |
| ------------- | -------------------------------------- | --- | --------------------- | ----------- | ----------------------------------- | ------ |
| Cloudflare R2 | https://www.cloudflare.com/products/r2 | $0  | 10GB 存储，1000 万次 A 类操作 | 无带宽费用，S3 兼容 | 像 S3 一样存对象，但省掉大部分外网流量费用，适合做下载/媒体存储。 | 强烈推荐   |
| Cloudinary    | https://cloudinary.com                 | $0  | 25GB 存储，25000 次转换     | 图片/视频处理     | 专门用来存、处理和分发图片/视频的云服务。               | 媒体处理强大 |
| BunnyCDN      | https://bunny.net                      | $0  | 免费 tier               | CDN 加速      | 价格便宜、地区覆盖广的静态资源加速服务。                | 性价比高   |
| jsDelivr      | https://www.jsdelivr.com               | $0  | 完全免费                  | JS 库 CDN    | 专门加速开源 JS/CSS 资源的公共 CDN。            | 公共库加速  |
| 网易云 NOS       | https://www.163yun.com/product/nos     | ¥0  | 50GB 存储，20GB 下行/月     | 国内访问快       | 网易云的对象存储，适合面向国内用户的静态资源托管。           | 国内用户   |
| 又拍云           | https://www.upyun.com                  | ¥0  | 10GB 存储，15GB 下行/月     | 国内 CDN      | 国内老牌 CDN/存储服务商，适合小站点冷启动。            | 需要申请联盟 |
| GoEnhance.ai  | https://goenhance.ai                   | $0  | 注册送 10G               | 对象存储+CDN    | 新兴的对象存储+CDN 组合方案，适合存放文件、图片和下载资源。    | 流量费用免费 |

***

## 六、监控与分析

| 服务名称             | 链接                                      | 价格     | 免费额度           | 特点     | 一句话介绍                                                                                                                      | 备注      |
| ---------------- | --------------------------------------- | ------ | -------------- | ------ | -------------------------------------------------------------------------------------------------------------------------- | ------- |
| Google Analytics | https://analytics.google.com            | $0     | 完全免费           | 网站统计分析 | 谷歌的站点统计工具，看 PV、来源、转化等数据。 [nerdwallet](https://www.nerdwallet.com/business/software/learn/what-is-stripe)                   | 标准分析工具  |
| Umami            | https://umami.is                        | $0/自托管 | 开源             | 隐私友好分析 | 自托管的简洁统计面板，常被用来替代 GA 以避免隐私问题。                                                                                              | 可自托管    |
| Plausible        | https://plausible.io                    | $0/自托管 | 开源             | 隐私友好分析 | 提供托管版和自托管版，界面非常干净，专注关键指标。                                                                                                  | 可自托管    |
| Grafana          | https://grafana.com                     | $0     | -              | 监控可视化  | 将日志、时间序列等监控数据做成各种大屏、图形和告警。 [devops](https://devops.com/grafana-labs-advances-open-source-visualization-and-observability/) | 技术监控    |
| PostHog          | https://posthog.com                     | $0     | -              | 产品分析   | 用事件、用户画像、漏斗等方式分析产品使用行为。                                                                                                    | 产品优化    |
| UptimeRobot      | https://uptimerobot.com                 | $0     | 50 个监控项，5 分钟检查 | 网站监控   | 定时请求你的站点，宕机时通过邮件/通知提醒你。                                                                                                    | 服务可用性监控 |
| Uptime Kuma      | https://github.com/louislam/uptime-kuma | 自托管    | -              | 开源监控工具 | 类似 UptimeRobot 的自托管版，可以自己在服务器上搭。                                                                                           | 自托管方案   |

***

## 七、邮件服务

| 服务名称        | 链接                                  | 价格  | 免费额度             | 特点           | 一句话介绍                                | 备注        |
| ----------- | ----------------------------------- | --- | ---------------- | ------------ | ------------------------------------ | --------- |
| Resend      | https://resend.com                  | $0  | 3000 封/月，100 封/天 | 开发者友好，现代 API | 现代感很强的邮件发送服务，和 JS/Node 生态结合紧密。       | 推荐        |
| Mailgun     | https://www.mailgun.com             | $0  | 免费 tier          | 邮件发送服务       | 老牌邮件 API 提供商，文档和生态都比较成熟。             | 独立开发者     |
| React Email | https://react.email                 | $0  | 开源               | 邮件模板         | 用 React 编写和预览邮件模板，再配合任意 SMTP/API 发送。 | 配合 Resend |
| Unsend      | https://github.com/unsend-io/unsend | 自托管 | -                | 开源           | 可以自己部署的邮件发送后端，用来替代托管型服务。             | 自托管方案     |
| AWS SES     | https://aws.amazon.com/ses          | 按量  | -                | 付费           | AWS 提供的大规模批量邮件发送服务，单价非常低。            | 大规模邮件     |
| 网易企业邮       | https://qiye.163.com                | ¥0  | 3G 容量            | 国内访问快        | 帮团队申请企业域名邮箱，适合国内团队。                  | 国内用户      |
| Zoho Mail   | https://www.zoho.com/mail           | ¥0  | 免费 tier          | 企业邮箱         | 提供无广告的免费企业邮箱账户。                      | 无广告       |

***

## 八、支付服务

| 服务名称 | 链接 | 价格 | 特点 | 一句话介绍 | 适用场景 | 备注 |
| --- | --- | --- | --- | --- | --- | --- |
| Stripe | https://stripe.com | 按交易收费 | 成熟稳定，生态完善 | 全球主流线上收款平台，支持信用卡、钱包等多种支付方式。 [stripe](https://stripe.com/payments/features) | 正式产品 |
| Paddle | https://www.paddle.com | 按交易收费 | 税务处理完善 | 专门替软件开发者代收款、代处理增值税和发票。 | 国际销售 |
| Creem.io | https://creem.io | 按交易收费 | 无需开公司 | 帮个人开发者代收海外款、结算到国内账户。 | 国内开发者 |
| Lemon Squeezy | https://www.lemonsqueezy.com | - | 被 Stripe 收购 | 以前很火的数字产品收费平台，目前不再开放新用户。 | - |

***

## 九、设计资源

| 资源类型 | 服务名称 | 链接 | 价格 | 特点 | 一句话介绍 | 备注 |
| --- | --- | --- | --- | --- | --- | --- |
| 图标 | Font Awesome | https://fontawesome.com | $0 | 丰富图标库 | 常用图标集，比如菜单、社交图标等，前端项目里经常会用到。 | 基础需求满足 |
| 图标 | Iconfinder | https://www.iconfinder.com | $0 | 可筛选免费图标 | 可以按风格、用途筛选图标，支持只看免费授权。 | 补充资源 |
| 图标 | Tabler Icons | https://tabler.io/icons | $0 | 1900+ 统一风格图标 | 提供一整套线性风格图标，适合后台、管理系统 UI。 | 简洁美观 |
| 图标 | Iconbolt | https://iconbolt.com | $0 | 6 万+ SVG 图标 | 聚合多套图标库，方便一站式搜索 SVG 图标。 | 免费使用 |
| 插画 | Undraw | https://undraw.co | $0 | 免费插画 | 提供风格统一的扁平插画，可以按主题选择并自定义主色。 | Landing page 必备 |
| 插画 | Storyset | https://storyset.com | $0 | 可调整参数 | 提供多种风格插画，支持在线改颜色、姿势、场景等。 | 丰富多样 |
| 落地页 | Ant Design Landing | https://landing.ant.design | $0 | 基于 Ant Design | 用 Ant Design 的组件搭建营销/展示型页面模板。 | 组件丰富 |
| 落地页 | Tailblocks | https://tailblocks.cc | $0 | Tailwind CSS 组件 | 各类常见版块（Hero、Features 等）的 Tailwind 片段，可直接复制使用。 | 常用模块齐全 |
| 图片编辑 | Photopea | https://www.photopea.com | $0 | 在线 PS | 浏览器里的“简化版 Photoshop”，支持 PSD/PSB 等格式。 | 无需下载 |
| 图片放大 | Upscayl | https://github.com/upscayl/upscayl | $0 | 开源图片放大 | 用 AI 将小图放大并增强细节，适合处理模糊图片。 | AI 增强 |
| 图片清理 | Lama Cleaner | https://github.com/Sanster/lama-cleaner | $0 | AI 擦除物体 | 在图片中一键抹掉多余物体、人物，并自动补全背景。 | 开源工具 |

***

## 十、其他工具

| 功能分类      | 服务名称                | 链接                                                                              | 价格    | 特点               | 一句话介绍                                   | 备注            |
| --------- | ------------------- | ------------------------------------------------------------------------------- | ----- | ---------------- | --------------------------------------- | ------------- |
| 学生资源      | 学生资源福利汇总            | https://edu.52it.de/                                                            | 折扣    | 学生资源             | 学生资源福利汇总 - 为学生提供最全面的优惠资源                |               |
| 教育邮箱      | maricopa            | https://www.maricopa.edu/                                                       | $0    | 美国社区大学           | 美国社区大学，可免费注册教育邮箱                        |               |
| 软件订阅      | GamsGo              | https://www.gamsgo.com/details/chatgpt                                          | 折扣    | 折扣价格订阅软件         | 折扣价格订阅软件                                |               |
| 域名        | Freenom             | https://www.freenom.com                                                         | $0    | 免费域名 (.tk/.ml 等) | 提供部分后缀的免费域名，适合测试或冷启动项目。                 | 冷启动使用         |
| 域名        | Cloudflare Domains  | https://www.cloudflare.com/products/registrar                                   | $10/年 | .com 域名          | Cloudflare 自带的域名注册服务，价格接近成本价。           | 价格稳定          |
| 域名        | Spaceship           | https://spaceship.com                                                           | 可变    | 首年便宜             | 提供较便宜首年域名注册，注意第二年起续费价格。                 | 注意续费价格        |
| 域名        | Regery              | https://regery.com                                                              | 可变    | 首年便宜             | 另一家便宜域名注册商，用来“薅首年价格”。                   | 注意续费价格        |
| 域名搜索      | tldx                | https://tldx.cc                                                                 | $0    | 开源域名搜索           | 搜索不同后缀下可用的域名，帮助你取名字。                    | 寻找合适域名        |
| HTTPS 证书  | acme.sh             | https://acme.sh                                                                 | $0    | 免费证书             | 利用 Let’s Encrypt 自动申请和续期 HTTPS 证书的脚本。   | Let's Encrypt |
| HTTPS 证书  | certbot             | https://certbot.eff.org                                                         | $0    | 免费证书             | EFF 提供的官方 Let’s Encrypt 申请工具，支持多种服务器环境。 | EFF 提供        |
| 音乐素材      | Uppbeat             | https://uppbeat.io                                                              | $0    | 免费背景音乐           | 提供适合视频、播客的免版税背景音乐，需按要求署名。               | 需注明版权         |
| 音乐素材      | Pixabay Music       | https://pixabay.com/music                                                       | $0    | 免费音乐             | 可商用的音乐素材下载站，适合做视频配乐。                    | 可商用           |
| 音乐素材      | Openverse           | https://openverse.org                                                           | $0    | 免费音乐             | 搜索各类 CC 授权音频/媒体资源的聚合平台。                 | WordPress     |
| 音乐素材      | NCS                 | https://ncs.io                                                                  | $0    | 无版权音乐            | 在 YouTube 等平台很常见的免费电音/背景音乐。             | 适合视频          |
| 音乐素材      | Dova-s              | https://dova-s.jp                                                               | $0    | 免费音乐             | 日本站点，提供大量免费 BGM，适合游戏/视频。                | 日本站点          |
| 音乐素材      | Incompetech         | https://incompetech.com/music                                                   | $0    | 免费音乐             | 个人音乐人网站，提供可署名使用的背景音乐。                   | 个人创作          |
| 工作流       | n8n                 | https://n8n.io                                                                  | 自托管   | 开源工作流自动化         | 像“自建 Zapier”，通过拖拽节点串联各种 API 和任务。        | Zapier 替代     |
| OAuth 管理  | Nango               | https://www.nango.dev                                                           | 自托管   | 开源 OAuth 管理      | 统一管理集成第三方服务时的 OAuth 授权流程和 token。        | 自托管方案         |
| 通知服务      | ntfy                | https://ntfy.sh                                                                 | $0    | 开源 pub-sub 通知    | 通过 HTTP/订阅方式向手机、浏览器等发送自定义通知。            | 推送消息          |
| 通知服务      | Gotify              | https://gotify.net                                                              | $0    | 开源通知服务           | 自托管的推送服务器，配合客户端 App 接收通知。               | 类似 ntfy       |
| 博客平台      | Sonic               | https://github.com/go-sonic/sonic                                               | $0    | Go 语言博客系统        | 类似 WordPress 的博客系统，但用 Go 编写，性能不错。       | 轻量级           |
| 数据库练习     | Crunchy Data        | https://www.crunchydata.com/developers/tutorials                                | $0    | 在线 PostgreSQL 练习 | 提供在线环境和练习题，帮助你学习 SQL 和 Postgres。        | 学习 SQL        |
| Linux 练习  | Sadservers          | https://sadservers.com                                                          | $0    | Linux 服务器管理题库    | 在“故障服务器”上做题，一边修一边练 Linux 运维技能。          | 浏览器实例         |
| Python 教程 | Full Stack Python   | https://www.fullstackpython.com                                                 | $0    | 免费英文教程           | 从 Web、数据、部署等角度系统讲 Python 生态。            | 实战应用          |
| Flask 教程  | Flask Mega-Tutorial | https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world | $0    | 免费电子书            | 非常经典的 Flask 系列教程，从零搭建一个完整网站。            | Flask 学习      |

如果你打算把这份表分享给刚入门的独立开发者，你最希望他们先重点看哪几块（比如：“先看开发工具 + 托管 + 支付”）？


## 十一、平台组合建议

### 入门级组合（完全免费）
- **部署**: Vercel（非商业项目）、Cloudflare Worker
- **数据库**: Supabase 或 Neon
- **认证**: Clerk 或 Better-Auth
- **存储**: Cloudflare R2
- **邮件**: Resend
- **月成本**: $0（在免费额度内）

### 稳定运营组合（小型商业项目）
- **部署**: Vercel Pro ($20/月)
- **数据库**: Neon ($19/月)
- **认证**: Clerk Pro ($25/月)
- **存储**: Cloudflare R2
- **邮件**: Resend Pro ($20/月)
- **支付**: Stripe 或 Creem
- **预估月成本**: $25-100

### Cloudflare All-in方案（$5/月）
- **计算**: Cloudflare Workers
- **数据库**: Cloudflare D1
- **存储**: Cloudflare R2
- **KV存储**: Cloudflare KV
- **优势**: 无带宽费用，全球CDN
- **适合**: 高流量无收入项目

### 自托管方案（VPS）
- **服务器**: Hostinger 或腾讯云
- **PaaS平台**: Dokploy 或 Coolify
- **数据库**: PostgreSQL + Redis
- **监控**: Uptime Kuma + Grafana
- **分析**: Plausible + Umami
- **邮件**: Unsend + AWS SES
- **备份**: Cloudflare R2
- **优势**: 完全控制，一站式搞定

## 十二、成本优化建议

### 1. 初创阶段（0-100用户）
- 使用免费额度最大化
- 选择Cloudflare All-in方案
- 使用公益AI API进行测试
- 月成本控制在$0-5

### 2. 成长阶段（100-1000用户）
- 升级到付费计划
- 使用ZenMux等稳定API
- 考虑Vercel Pro或类似服务
- 月成本控制在$20-50

### 3. 稳定阶段（1000+用户）
- 根据需求选择最佳服务
- 考虑自托管降低成本
- 使用Stripe等成熟支付
- 月成本根据收入调整

### 4. 优化技巧
- **图片优化**: 使用Cloudinary或R2 + 压缩
- **数据库优化**: 使用索引，避免N+1查询
- **缓存策略**: 使用Redis或Cloudflare KV
- **代码优化**: 使用Edge Function减少服务器时间
- **监控告警**: 设置用量告警避免超额

## 十三、注意事项

### 免费额度限制
1. **Vercel免费版不支持商业项目**
2. **注意各平台的用量限制**
3. **设置预算告警**
4. **定期备份数据**

### 服务选择建议
1. **优先选择成熟平台**（Stripe、AWS等）
2. **考虑长期成本**（不只是首年价格）
3. **评估迁移成本**（避免后期难以迁移）
4. **关注社区支持**（遇到问题容易解决）

### 国内用户特别提示
1. **备案要求**: 国内服务器需要备案
2. **访问速度**: 国内用户优先选择国内CDN
3. **支付限制**: 国内开发者使用Stripe需要额外配置
4. **AI API**: 国内访问国外API可能不稳定

---

**最后更新**: 2026-03-16
