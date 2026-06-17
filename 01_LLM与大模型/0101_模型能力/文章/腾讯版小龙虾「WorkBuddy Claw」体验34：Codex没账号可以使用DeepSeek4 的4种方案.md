---
title: 腾讯版小龙虾「WorkBuddy Claw」体验34：Codex没账号可以使用DeepSeek4 的4种方案
author: 一个学习日记
date: 学生一枚学生一枚
url: https://mp.weixin.qq.com/s?__biz=MzkzNjEwMDQxNg==&mid=2247484681&idx=1&sn=4828e698c195a57880226d2ae1276da7&chksm=c3f25bf9ecd068fb8385affb8f79a6c933183199e7ac4dd86e414b5a967021e3f681181c5ccb&mpshare=1&scene=24&srcid=0529oJskHrCWmq9F5nwyYFzh&sharer_shareinfo=06fe4c6d32562ab8df3065390a85a6b6&sharer_shareinfo_first=06fe4c6d32562ab8df3065390a85a6b6#rd
---

[Claude Code都用了一段时间](https://mp.weixin.qq.com/s?__biz=MzkzNjEwMDQxNg==&mid=2247484313&idx=1&sn=d8c569ffd5082f600c15abef9388fd97&scene=21#wechat_redirect)， Codex最近也很火，虽然不是很着急因为我claude code用起来也挺好的，但很多人因为的账号问题被困住了，我自己也想尝试下，今天就让我的小龙虾「WorkBuddy Claw」结合我的知识库来一起尝试下如何使用它：

居然给我整理了4个方案：赞一个啊

## 一句话结论

**Codex CLI 原生支持 DeepSeek，完全不需要 OpenAI 账号登录。** Codex Desktop 桌面版需要登录但可通过本地代理绕过。

---

## 方案概览

| 方案 | 需要OpenAI登录 | 难度 | 推荐度 | 适用场景 |
| --- | --- | --- | --- | --- |
| **方案A：Codex CLI 原生** | ❌ 不需要 | ⭐ 极简 | ⭐⭐⭐⭐⭐ | 命令行用户，npm 一键安装 |
| **方案B-1：codex-relay** | ⚠️ 需先登录一次 | ⭐⭐ 简单 | ⭐⭐⭐⭐ | Desktop GUI，pip 安装（Rust 底层） |
| **方案B-2：mimo2codex** | ⚠️ 需先登录一次 | ⭐⭐⭐ 中等 | ⭐⭐⭐⭐ | Desktop GUI，多模型 + Web 管理面板 |
| **方案C：OpenCode** | ❌ 不需要 | ⭐ 极简 | ⭐⭐⭐⭐⭐ | 完全开源，`/connect` 零配置 |
| **方案D：DeepSeek TUI** | ❌ 不需要 | ⭐⭐ 简单 | ⭐⭐⭐⭐⭐ | 专为 DeepSeek 优化，100万上下文 |

方案A看起来最简单，推荐度又高，我就尝试第一个吧，其他几个放后面让需要的人参考吧。

这个地方提醒下，很多都是windows终端下命令操作的，如果这个不会的要先考虑下了：

下面正式开始：

【1】程序安装

1）先确认 Node.js版本>=22，我已经安装过了，都24了，就跳过，没的可以看后面文档说明

2）安装 Codex，开始使用

npm install -g @openai/codex

一直下不去，然后使用了下面的替代参数解决了

```
npm install -g @openai/codex --registry=https://registry.npmmirror.com
```

```

```

```
看看版本：0.134
```

```

```

3）申请DeepSeek API Key，这个之前已经申请过了，拿来就可以用了

4）安装本地代理，这个必须的，否则不能直接用DeepSeek

git clone https://github.com/wujfeng712-ui/codex-bridge.git

这个提醒下，这个git是软件仓库的管理工具，如果没安装过的，需要安装下的。

至此所有的准备工作都做好了。

【2】修改设置

具体怎么操作后面的操作指南里都有的，详细的配置截图我就不发了，因为有我的key在里面。万一不小心泄漏了，就亏大了，是不是。

【3】启动测试，

1）打开一个终端窗口，先启动代理：

```
PS C:\codex-bridge> node --env-file=.env proxy.mjs
```

代理启动成功。

2）再打开一个终端窗口，启测试Codex，

先测试 deepseek-v4-flash, 执行下面命令：

```
codex -p deepseek "hello"
```

结果显示成功了：

再测试  deepseek-v4-pro, 执行下面命令：

```
codex -p deepseek-pro "hello2"
```

结果显示也是成功的：

然后看看我哪个代理的窗口，里面显示也是对的：

怎么样，简单吧，有需要的可以尝试看了，我选的方案A并测试通过了，想偷懒的可对照我的操作来，想选其他方法的，自己看自己练了。

下面是完整的操作指南，给需要的人参考：

# Codex 不登录使用 DeepSeek V4 Pro 完整指南

## 方案A：Codex CLI 原生接入（推荐！）

> **零门槛、不需要 OpenAI 账号、一行命令即可**

### 第 1 步：安装 Node.js（≥ 22）

Codex CLI 需要 Node.js 22 或更高版本：

```
winget install OpenJS.NodeJS.LTS# 或从 https://nodejs.org 下载安装包
```

**验证：**

```
node --version   # 需要 ≥ v22.0.0npm --version
```

### 

### 

### 第 2 步：安装 Codex CLI

```
npm install -g @openai/codex
```

**国内网络慢使用下面地址：**

```
npm install -g @openai/codex --registry=https://registry.npmmirror.com
```

**验证安装：**

```
codex --version    # 示例输出：0.133.0
```

### 第 3 步：获取 DeepSeek API Key

1. 访问 platform.deepseek.com
2. 注册/登录 DeepSeek 账号
3. 进入「API Keys」页面，创建新 Key
4. 复制 Key（格式：`sk-xxxxxxxx`）

> 💰 DeepSeek V4 Pro 价格极低，百万 token 仅需几元人民币

### 第 4 步：搭建本地代理（必须！协议转换）

> ⚠️ Codex CLI 使用 OpenAI Responses API，DeepSeek 只提供 Chat Completions API，协议不兼容。需要本地代理做协议转换。

**安装 codex-bridge（零依赖、不需 npm install）：**

```
git clone https://github.com/wujfeng712-ui/codex-bridge.gitcd codex-bridgecp env.example .env
```

**编辑 `.env` 填入配置：**

```
# 代理认证密钥（自行生成随机字符串）PROXY_AUTH_KEY=sk-proxy-local-your-random-string# DeepSeek API KeyDEEPSEEK_API_KEY=sk-your-key-here
```

**启动代理：**

```
node --env-file=.env proxy.mjs# 代理运行在 http://127.0.0.1:4000
```

### 第 5 步：配置 Codex（认证 + 模型）

> ⚠️ **2026-05-27 更新**：新版 Codex CLI 将 profile 拆成了独立文件，不再放在 `config.toml` 里。

**文件 1：`~/.codex/auth.json`**

```
{  "OPENAI_API_KEY": "sk-proxy-local-你的随机字符串"}
```

> 💡 这里的值和 `.env` 里 `PROXY_AUTH_KEY` **必须完全一致**，这是 codex-bridge 的入站认证口令。

**文件 2：`~/.codex/config.toml`**（只放 provider，不放 profiles）

```
[model_providers.local_proxy]name = "local_proxy"base_url = "http://127.0.0.1:4000/v1"wire_api = "responses"requires_openai_auth = true
```

**文件 3：`~/.codex/deepseek.config.toml`**（每个 profile 一个独立文件）

```
model = "deepseek-v4-flash"model_provider = "local_proxy"
```

**文件 4：`~/.codex/deepseek-pro.config.toml`**（Pro 模型，可选）

```
model = "deepseek-v4-pro"model_provider = "local_proxy"
```

### 第 7 步：开始使用

```
# 用 profile 切换（不是 --provider！）codex -p deepseek "用 Python 写一个快速排序"# Pro 模型（复杂任务）codex -p deepseek-pro "分析这个项目的架构，给出优化建议"
```

## 方案B：Codex Desktop + 本地代理

> 如果你想要 Codex Desktop 的 GUI 体验，但用 DeepSeek 作为后端

### 方案 B-1：codex-relay（Python，最简单）

**项目地址**：github.com/CAZAMA1/Codex-app-with-deepseek-perfectly

**架构：**

Codex Desktop → 本地代理(:4445) → codex-relay(:4446) → DeepSeek API

**安装步骤：**

1. 安装 Codex Desktop（从 OpenAI 官网下载，**需要先完成一次登录**）
2. 安装 Python >= 3.11（⚠️ 2026-05-27 更新：要求已从 3.8 提升到 3.11）
3. 安装 codex-relay：

```
pip install codex-relay   # 最新版 0.2.1（2026-05-24 发布）
```

💡 codex-relay 已用 Rust 重写底层，但以 Python wheel 预编译包发布，`pip install` 即可，**无需安装 Rust 环境**。也可用 `cargo install codex-relay`。

### 方案 B-2：mimo2codex（Node.js，多模型支持）

**项目地址**：github.com/7as0nch/mimo2codex

**安装步骤：**

1. 安装 Node.js 22+
2. 安装 mimo2codex：

```
npm install -g mimo2codex
```

**踩坑提醒：**

* npm 可能不在 PATH 中，需手动添加
* GitHub 直连可能超时，建议配置代理
* mimo2codex 写入的 config.toml 较精简，原有 plugins/MCP 配置需手动合并

## 方案C：OpenCode（完全开源替代）

> 开源的终端编程助手，不依赖 OpenAI 任何服务，GitHub 160K+ Stars

### 安装

**版本要求：>= v1.14.24**（低于此版本不支持 `/connect` 交互配置）

```
# npm（包名是 opencode-ai，不是 @opencode-ai/opencode！）npm install -g opencode-ai# 或一键安装curl -fsSL https://opencode.ai/install | bash# 或 Chocolatey (Windows)choco install opencode
```

### 配置 DeepSeek（零配置文件！交互式）

```
# 1. 启动 OpenCode（在项目目录内）opencode# 2. 在 OpenCode 界面输入/connect# 3. 选择供应商 → deepseek# 4. 粘贴 DeepSeek API Key# 5. 选择模型（推荐 DeepSeek-V4-Pro）# 6. 确认，完成！
```

**使用：**

```
opencode "用 TypeScript 写一个 Express 服务器"
```

**切换模型/供应商：** 再次执行 `/connect` 即可。

## 方案D：DeepSeek TUI

> GitHub 7K+ Stars（仓库：Hmbown/DeepSeek-TUI），纯 Rust 编写，专为 DeepSeek V4 优化

**安装（npm，推荐新手）：**

```
npm install -g deepseek-tui        # TUI 主程序npm install -g deepseek-tui-cli    # CLI 命令行工具（⚠️ 新增，需单独装）
```

**安装（Cargo，Rust 开发者）：**

```
# 需 Rust >= 1.85.0cargo install deepseek-tui --lockedcargo install deepseek-tui-cli --locked
```

**配置**

```
# 方式一：交互登录（推荐）deepseek login --api-key "sk-your-key"# 方式二：环境变量（仍可用）export DEEPSEEK_API_KEY="sk-your-key"
```

**使用：**

```
deepseek-tui# 进入 TUI 界面后直接输入任务
```

**特点（v0.3.x+）：**

* **100万 Token 超长上下文**

  — 覆盖中大型项目全量代码
* **四档思考模式**

  ：non / normal / high / max
* **16路并行子任务调度**
* **三大交互模式**

  ：Plan（只读）/ Agent（审批）/ YOLO（全自动）
* 文件编辑、命令执行、Git 自动备份
* MCP（Model Context Protocol）支持
* LSP 诊断集成
* 成本追踪 + 自动上下文压缩

## 最终推荐

| 你的情况 | 推荐方案 |
| --- | --- |
| 命令行够用，想最快上手 | **方案A：Codex CLI** 🏆 |
| 想要 GUI 桌面体验 | **方案B-1：codex-relay** |
| 技术探索，多模型切换 | **方案B-2：mimo2codex** |
| 完全不想碰 OpenAI 产品 | **方案C：OpenCode** |
| 追求极致 DeepSeek 体验 | **方案D：DeepSeek TUI** |

后面还有更多体验会继续更新的，感兴趣就关注一下，来了就不会错过。

如果要看之前的体验记录可以直接点下面的内容：

---

**往期回顾**：

* 体验1：WorkBuddy安装体验 [腾讯版小龙虾「WorkBuddy Claw」学习和体验结果](https://mp.weixin.qq.com/s?__biz=MzkzNjEwMDQxNg==&mid=2247484052&idx=1&sn=4b114a6bd95bd5b882166284121b41fd&scene=21#wechat_redirect)
* 体验10：”龙虾“越多越厉害？  [腾讯版小龙虾「WorkBuddy Claw」体验10：“龙虾”越多越厉害？WorkBuddy给我泼冷水](https://mp.weixin.qq.com/s?__biz=MzkzNjEwMDQxNg==&mid=2247484226&idx=1&sn=dc7bf96db08a1d29b1e49719c4fe550d&scene=21#wechat_redirect)
* 体验15：hermes什么东西鬼？  [腾讯版小龙虾「WorkBuddy Claw」体验15：hermes 什么东西鬼？比“龙虾”会更好？](https://mp.weixin.qq.com/s?__biz=MzkzNjEwMDQxNg==&mid=2247484281&idx=1&sn=8c928f7514b82643802b10f710fa2759&scene=21#wechat_redirect)
* 体验17：帮我搞定Claude Code国内账号  [腾讯版小龙虾「WorkBuddy Claw」体验17：帮我搞定了Claude Code 国内账号问题啦](https://mp.weixin.qq.com/s?__biz=MzkzNjEwMDQxNg==&mid=2247484313&idx=1&sn=d8c569ffd5082f600c15abef9388fd97&scene=21#wechat_redirect)
* 22：微信小程序小白操作手册 [腾讯版小龙虾「WorkBuddy Claw」体验22：搞定微信小程序，附小白操作手册](https://mp.weixin.qq.com/s?__biz=MzkzNjEwMDQxNg==&mid=2247484464&idx=1&sn=03c2f697fdb7b4e037bd853c6ecf4ba8&scene=21#wechat_redirect)
* 24：我的知识库+obsidian [腾讯版小龙虾「WorkBuddy Claw」体验24：WorkBuddy 设置的知识库和obsidian 知识库数据怎么共享](https://mp.weixin.qq.com/s?__biz=MzkzNjEwMDQxNg==&mid=2247484489&idx=1&sn=e0de6cd1063d11c391292c9f49e5c13f&scene=21#wechat_redirect)
* 28：智能客服怎么实现？[腾讯版小龙虾「WorkBuddy Claw」体验28：智能客服怎么实现的](https://mp.weixin.qq.com/s?__biz=MzkzNjEwMDQxNg==&mid=2247484549&idx=1&sn=989812263565b2ed588a54c105a97623&scene=21#wechat_redirect)

如果要看全部文章的点这里👉 [AI体验](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzkzNjEwMDQxNg==&action=getalbum&album_id=4439720096751697921#wechat_redirect)