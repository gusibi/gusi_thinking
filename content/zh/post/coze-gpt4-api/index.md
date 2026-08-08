---
date: '2024-02-15T16:26:07+08:00'
lastmod: '2024-02-15T16:26:07+08:00'
draft: false
title: coze_gpt4_api_get
slug: coze-gpt4-api
description: 1. 打开 [discord开发者平台](https://discord.com/developers/applications) 。
tags:
- AI
- 工具
categories:
- AI
comment: false
toc: false
---

> https://www.dqzboy.com/16532.html
>
> https://github.com/deanxv/coze-discord-proxy


## 如何使用

1. 打开 [discord开发者平台](https://discord.com/developers/applications) 。
2. 创建bot-A,并记录bot专属的`token`和`id(COZE_BOT_ID)`,此bot为被coze托管的bot。
3. 创建bot-B,并记录bot专属的`token(BOT_TOKEN)`,此bot为我们与discord交互的bot。
4. 两个bot开通对应权限(`Administrator`)并邀请进服务器,记录服务器ID(`GUILD_ID`) ( 过程不在此赘述)。
5. 打开 [coze官网](https://www.coze.com/) 创建自己bot。
6. 创建好后推送(`Auto-Suggestion`为`default`),配置discord-bot的`token`,即bot-A的`token`,点击完成后在discord的服务器中可看到bot-A在线并可以@使用。
7. 配置环境变量,并启动本项目。
8. 访问接口地址即可开始调试。

Render 可以直接部署 docker 镜像, 不需要 fork 仓库：[Render](https://dashboard.render.com/)

```bash
docker run --name coze-discord-proxy -d --restart always \
-p 7077:7077 \
-v $(pwd)/data:/app/coze-discord-proxy/data \
-e USER_AUTHORIZATION="YOUR_VALUE_HERE" \
-e BOT_TOKEN="YOUR_VALUE_HERE" \
-e GUILD_ID="YOUR_VALUE_HERE" \
-e COZE_BOT_ID="YOUR_VALUE_HERE" \
-e PROXY_SECRET="YOUR_VALUE_HERE" \
-e CHANNEL_ID="YOUR_VALUE_HERE" \
-e TZ=Asia/Shanghai \
deanxv/coze-discord-proxy 


docker run --name coze-discord-proxy -d --restart always \
-p 7077:7077 \
-v $(pwd)/data:/app/coze-discord-proxy/data \
-e USER_AUTHORIZATION="YOUR_VALUE_HERE" \
-e BOT_TOKEN="YOUR_VALUE_HERE" \
-e GUILD_ID="YOUR_VALUE_HERE" \
-e COZE_BOT_ID="YOUR_VALUE_HERE" \
-e PROXY_SECRET="YOUR_VALUE_HERE" \
-e CHANNEL_ID="YOUR_VALUE_HERE" \
-e TZ=Asia/Shanghai \
deanxv/coze-discord-proxy
```

## 相关链接 ：

1. []()
