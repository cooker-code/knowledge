---
title: 开源LiteLLM Agent平台：K8s沙箱+凭证库，让编程Agent永远拿不到真实API密钥
author: AI工程化
date: winkrunwinkrun
url: https://mp.weixin.qq.com/s?__biz=MzA5MTIxNTY4MQ==&mid=2461159744&idx=1&sn=64f2a6f600615896419d5059eea09232&chksm=863335ca34e3ab5be79d4fa90f711bf6af5ad6ac16a96d4eb974f3e9a147c323dbf983d0c139&mpshare=1&scene=24&srcid=0518Lv2iM3SGrhSj73JoJHvO&sharer_shareinfo=2e86d1d00c28b774b20b51706d1bcb97&sharer_shareinfo_first=2e86d1d00c28b774b20b51706d1bcb97#rd
---

LiteLLM团队开源了LiteLLM Agent Platform，解决的是编码Agent最容易被忽略的安全问题——密钥泄露。有开发者Jason Haugh提到，长运行Agent的密钥隔离是那种没人会提前考量，直到被坑才幡然醒悟的问题。生产环境跑Claude Code不用类似K8s沙箱的隔离机制，根本就是在给密钥外泄开绿灯。

核心安全机制

### 

1. **会话级沙箱隔离**：每个Agent运行在全新的Kubernetes Pod中，会话结束后Pod直接销毁，无持久化存储，无跨会话污染，Agent不会留下任何操作痕迹。
2. **出站凭证置换**：Agent环境中仅存放Stub假令牌（如`GITHUB_TOKEN=stub_github_a8f1`），只有对外发起TLS请求时，凭证库才会将假令牌替换为真实密钥——Agent根本碰不到真实密钥，自然无法外泄。

### 核心功能与适配

* 支持多Agent框架：Claude Code、Codex、Hermes，无需更换现有工具链
* 多部署环境：本地、AWS EKS、GCP GKE、Render
* 终端直连沙箱：通过`lap` CLI，可将本地终端通过WebSocket挂接至远程沙箱的TUI，直接操作Agent

### 

平台架构图：

快速上手

### 

#### 1. CLI快速体验（无需自托管）

```
# 安装lap CLI  
git clone https://github.com/BerriAI/litellm-agent-platform.git  
cd litellm-agent-platform/cli && npm install  
ln -sf "$PWD/bin/lap.mjs" ~/.local/bin/lap  
  
# 登录平台  
lap login  
  
# 启动Claude Code沙箱  
lap claude-code-cli
```

启动后会生成全新K8s Pod，本地终端挂接至其TTY，按`Ctrl-D`可断开，会话默认保留24小时。

#### 2. 自托管部署

* 本地开发：使用kind集群，执行`bin/kind-up.sh` + `docker compose up`即可启动，访问`localhost:3000`创建Agent
* 生产部署：推荐AWS EKS作为沙箱集群，Render托管Web+Worker，部署脚本与配置已在仓库`deploy/`目录提供

### 地址：https://github.com/BerriAI/litellm-agent-platform

关注公众号回复“进群”入群讨论。