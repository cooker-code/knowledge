---
title: 超强AI工作流平台：n8n本地部署+实践应用！0成本+自动化，运营一个AI新闻聚合频道！真正解放你的双手！
author: 云原生架构实践
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI3MTE1MDQyOA==&mid=2247484954&idx=1&sn=4be13820025e16afe52ea34b23044b3e&chksm=ebd5a2c773b982af570ce21cf7469ec4115e85ef10b9dfac76408c999d1fce727c17afa17d41&mpshare=1&scene=24&srcid=09250uMMf4HVCJrRhP8Vo4KB&sharer_shareinfo=6389060dd6b164abca89bd5748473f4d&sharer_shareinfo_first=6389060dd6b164abca89bd5748473f4d#rd
---

TL;DR

* 创建了 n8n 工作流程来汇总和分析来自多个来源的科技新闻
* 工作流获取 RSS 提要、过滤 AI 相关内容并生成汇总文章
* 使用 DeepSeek AI 的 GPT 模型等 AI 服务进行内容分析和文章摘要汇总

# RSSHub 万物皆可 RSS

RSSHub 提供从各种来源聚合的数百万内容，充满活力的开源社区确保提供 RSSHub 的新路线、新功能和错误修复。

* 🌐去中心化  
  5000 多个实例在运行，形成了世界上最大的 RSS 网络。
* 🕊️开放  
  所有代码都在 MIT 许可下开源，并遵循开放标准和协议。
* 🌿活跃的社区  
  超过900名贡献者和维护者正在积极支持RSSHub。
* 🥳开箱即用  
  大量预配置路由和公共实例可供立即使用。
* 🧩可扩展  
  强大的 API 和生态项目正在支持各种场景。

# Docker Compose 部署（推荐）（防屏蔽版）

RSS主要定义新闻信息也是可以选择公共的实例，但是绝对多数会被GWF 公共的实例：https://docs.rsshub.app/routes/popular

## 安装

参考：https://docs.rsshub.app/deploy/#install

### 下载docker-compose.yml

也可以选择Kubernetes（Helm）部署

```
$ wget https://raw.githubusercontent.com/DIYgod/RSSHub/master/docker-compose.yml
```

```
services:  
    rsshub:  
        # two ways to enable puppeteer:  
        # * comment out marked lines, then use this image instead: diygod/rsshub:chromium-bundled  
        # * (consumes more disk space and memory) leave everything unchanged  
        image: diygod/rsshub # or ghcr.io/diygod/rsshub  
        restart: always  
        ports:  
            - "1200:1200"  
        environment:  
            NODE_ENV: production  
            CACHE_TYPE: redis  
            REDIS_URL: "redis://redis:6379/"  
            PUPPETEER_WS_ENDPOINT: "ws://browserless:3000"# marked  
        healthcheck:  
            test: ["CMD", "curl", "-f", "http://localhost:1200/healthz"]  
            interval: 30s  
            timeout: 10s  
            retries: 3  
        depends_on:  
            - redis  
            - browserless # marked  
  
    browserless: # marked  
        image: browserless/chrome # marked  
        restart: always # marked  
        ulimits: # marked  
            core: # marked  
                hard: 0 # marked  
                soft: 0 # marked  
        healthcheck: # marked  
            test: ["CMD", "curl", "-f", "http://localhost:3000/pressure"] # marked  
            interval: 30s # marked  
            timeout: 10s # marked  
            retries: 3 # marked  
  
    redis:  
        image: redis:alpine  
        restart: always  
        volumes:  
            - redis-data:/data  
        healthcheck:  
            test: ["CMD", "redis-cli", "ping"]  
            interval: 30s  
            timeout: 10s  
            retries: 5  
            start_period: 5s  
  
volumes:  
    redis-data:
```

```
检查是否需要更改任何配置  
$ vi docker-compose.yml  # or your favorite editor
```

启动服务：

```
docker-compose up -d
```

http://{Server IP}:1200在浏览器中打开，尽情享受吧！✅

# ▶️ 立即行动方案

n8n 工作流程，其功能如下：  
从多个新闻源获取 RSS 内容解析 RSS 内容过滤与 AI 相关的条目，利用人工智能生成扩展文章将结果发送到机器人频道

## 效果预览

📱 企业微信频道自动推送：微信公众号后台回复'新闻工作流'获取完整工作流

[[N8N]手把手教你用n8n打造智能办公机器人！零代码轻松搞灵活的AI工作流程自动化+AI Agent（部署篇）](https://mp.weixin.qq.com/s?__biz=MzI3MTE1MDQyOA==&mid=2247484892&idx=1&sn=6f680ef142847a813e18739c6a4c827f&scene=21#wechat_redirect)