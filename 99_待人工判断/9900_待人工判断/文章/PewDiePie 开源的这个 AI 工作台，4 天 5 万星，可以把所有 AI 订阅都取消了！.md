---
title: PewDiePie 开源的这个 AI 工作台，4 天 5 万星，可以把所有 AI 订阅都取消了！
author: 开源星探
date: 开源星探开源星探
url: https://mp.weixin.qq.com/s?__biz=MzkwMjQ0NzI0OQ==&mid=2247506263&idx=1&sn=f9a664c43b4216646fe965c3ecad84e2&chksm=c17ce9c18d51cdb9cafa12e58b59aeb28b7c6d500dc3d30d266fd15d2546a04b0580a5b9d6ee&mpshare=1&scene=24&srcid=0605h0WpllY1QoiwzI9QMkMn&sharer_shareinfo=c8a09e0c3f65f56f19643ab3589d1c4a&sharer_shareinfo_first=c8a09e0c3f65f56f19643ab3589d1c4a#rd
---

这几天 GitHub 上有个项目刷爆了，一个自托管的开源 AI 工作台。

开源到现在也就四天，已经快 5 万+ Star 了。

装上这个叫 **Odysseus** 的项目，会使你的整个工作流都有所改变。一个浏览器标签页全搞定，接上 Ollama 跑本地模型，API 的钱全省了。

### 这项目到底是什么

**Odysseus** 就是一个自己托管的全能 AI 工作台。

跟 ChatGPT 的界面逻辑差不多——左边聊天列表，中间对话区，但不一样的是：它在你的硬件上跑，数据留在你本地，模型你可以随便换。

项目作者 PewDiePie 称没有追踪，没有订阅，没有猫腻。它永远是你的。确实戳中了很多人的痛点。

从技术栈来看，**Odysseus** 用的是 FastAPI 后端，Python 写的，ChromaDB 做向量存储，SearXNG 做元搜索，Docker Compose 一键部署。

整个项目把十个常用工具整合到一个界面里：聊天、AI 代理、模型管理、深度研究、模型对比、文档编辑器、记忆系统、邮件、待办事项、日历。

### 为什么这个项目能爆火

#### 1、Cookbook — 硬件扫描，自动推荐你能跑的模型

这是最惊喜的功能。点开 Cookbook，它先扫描你的硬件（GPU 型号、显存大小、内存），然后给你推荐你这台机器能跑什么模型。

每个模型旁边有个适配度评分（fit score），显存够不够、推理速度预估都标得明明白白。

#### 2、模型对比 — 盲测才知道差距

选两个模型，问同一个问题，答案不告诉你哪个是哪个。等你选完 A 更好或 B 更好，才揭晓。

避免了你心里已经偏向某个模型导致的不客观。

#### 3、深度研究 - 自动查资料写出报告

相当于 ChatGPT 的 Deep Research 功能，但在本地跑。

你给一个话题，它会自己去网上搜资料、读网页、提炼要点，最后生成一份结构化的报告。

质量跟 ChatGPT 的 Deep Research 差不多，但一分钱不要。

#### 4、邮件 AI - 自动筛垃圾+摘要+回复草稿

接入 IMAP 邮箱后，它会自动：给每封新邮件打分（紧急/普通/垃圾），对长邮件生成一句话摘要，给常见问题生成回复草稿。

#### 5、记忆系统 - 你的 AI 越用越懂你

Odysseus 会记住你之前的对话内容和偏好。

比如你跟它说过我在做 React 项目、不喜欢用 Redux、偏好 Tailwind CSS，下次让它写组件，它自动给你 Tailwind 版本的代码，不会再问你要用 CSS 还是 Tailwind。

底层用的是 ChromaDB 向量存储，支持关键词+语义双重检索。记忆可以导出/导入。

### 完整功能一览

Odysseus 的功能远远不止上面说的这几个，它是一个完整的 AI 工作空间：

#### 🗣️ Chat & Agents

* • 支持多种模型接入：vLLM、llama.cpp、Ollama、OpenRouter、OpenAI
* • Agent 模块带工具调用、MCP、Shell、文件读写、技能和记忆
* • 可以让它接手整个任务自己跑

#### 📚 Cookbook

* • 扫描你的硬件配置
* • 自动推荐能跑的模型
* • 点击一下就能下载并部署起来
* • 支持 GGUF、FP8、AWQ 等量化格式

#### 🔍 Deep Research

* • 多步骤搜集和综合资料
* • 最后输出一份带图表的可视化报告
* • 基于 Tongyi DeepResearch 改编

#### 📝 Documents

* • 多标签页文档编辑器
* • 支持 Markdown、HTML、CSV
* • 语法高亮
* • AI 辅助编辑和建议

#### 📧 Email

* • IMAP/SMTP 邮箱接入
* • AI 自动分类：紧急提醒、自动标签、自动摘要、自动回复草稿、自动垃圾邮件过滤
* • 支持 CalDAV 感知

#### 📅 Calendar

* • 本地优先日历
* • 支持 CalDAV 同步到 Radicale、Nextcloud、Apple、Fastmail
* • 支持 .ics 导入/导出
* • 每个日历可以设置不同颜色

#### ✅ Notes & Tasks

* • 快速笔记带提醒
* • 待办事项列表
* • 定时任务（Agent 可以执行）
* • 支持 ntfy、浏览器、邮件等通知渠道

#### 📱 移动端支持

* • 响应式设计
* • 可安装为 PWA
* • 支持触摸手势

#### 🔧 其他功能

* • 图像编辑器
* • 主题编辑器
* • 文件上传（支持视觉模型和 PDF）
* • Web 搜索
* • 预设配置
* • 会话管理
* • 双因素认证（2FA）

### 快速上手

Odysseus 支持多种安装方式，推荐用 Docker，最简单。

#### 🐳 Docker 部署（推荐）

前提：你已经装了 Docker 和 Docker Compose。

```
git clone https://github.com/pewdiepie-archdaemon/odysseus.git  
cd odysseus  
cp .env.example .env  # 可选，但推荐显式设置默认值  
docker compose up -d --build
```

等容器启动完，打开 `http://localhost:7000`。

**第一次启动会自动创建管理员账号**：

* • 用户名：`admin`（除非你设置了 `ODYSSEUS_ADMIN_USER`）
* • 密码：终端里会打印一个临时密码（Docker 安装的话在 `docker compose logs odysseus` 里看）

登陆后记得去设置里改密码。

**默认端口占用了怎么办？**  
在 `.env` 里设置 `APP_PORT=7001`，然后重新创建容器。

**想从局域网访问？**  
设置 `APP_BIND=0.0.0.0`，但**千万不要直接暴露到公网**！如果需要远程访问，用 Tailscale 组局域网，前面加 Caddy 或 nginx 做 HTTPS 反向代理。

#### 🖥️ 原生 Linux / macOS

```
git clone https://github.com/pewdiepie-archdaemon/odysseus.git  
cd odysseus  
python3 -m venv venv  
source venv/bin/activate  
pip install -r requirements.txt  
python setup.py  
python -m uvicorn app:app --host 127.0.0.1 --port 7000
```

**要求**：Python 3.11+。Cookbook 还需要 `tmux` 来后台下载和运行模型。

#### 🍎 Apple Silicon（M 系列 Mac）

Docker 在 macOS 上用不了 Metal GPU。想要 GPU 加速的话，跑原生命令：

```
git clone https://github.com/pewdiepie-archdaemon/odysseus.git  
cd odysseus  
./start-macos.sh
```

启动后访问 `http://127.0.0.1:7860`。

想在手机上访问（通过 Tailscale 等信任的局域网/VPN）：

```
ODYSSEUS_HOST=0.0.0.0 ./start-macos.sh  
# 然后打开 http://<tailscale-ip>:7860
```

想做成一个可点击的应用包：

```
./build-macos-app.sh
```

#### 🪟 Windows

```
.\launch-windows.ps1
```

### 写在最后

Odysseus 的火爆，其实反映了一个趋势：大家想要 AI，但不想被大公司锁死，不想每个月交订阅费，不想把自己的数据送到别人的服务器上。

这个项目不是完美的 —— README 里自己都说 "more jank and fun"，界面可能没有 ChatGPT 那么精致，功能可能偶尔会有小问题。

但它代表了一种可能性：你可以拥有一个完全属于自己的 AI 工作环境，数据不出门，模型随便换，功能随便加。

GitHub：https://github.com/pewdiepie-archdaemon/odysseus

 

如果本文对您有帮助，也请帮忙点个 赞👍 + 在看 哈！❤️

**在看你就赞赞我！**