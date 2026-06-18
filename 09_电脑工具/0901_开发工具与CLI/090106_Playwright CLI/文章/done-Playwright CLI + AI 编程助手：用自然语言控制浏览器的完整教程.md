---
title: Playwright CLI + AI 编程助手：用自然语言控制浏览器的完整教程
author: AI李工科
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3NTA2Mw==&mid=2247484041&idx=1&sn=7a2cfb4fe87aab50c343e8d44947531c&chksm=c11b4d9471ea4bbb8731ebcd59ad54118c6db5cf13649d51782f342af476eb9ac9f7dff9e638&mpshare=1&scene=24&srcid=0418ApLEIOvrtv15pTdLuNM0&sharer_shareinfo=4643c110eb661968188c6a6fda12d260&sharer_shareinfo_first=4643c110eb661968188c6a6fda12d260#rd
---
> 已吸收至：[[09_电脑工具/0901_开发工具与CLI/090106_Playwright CLI/090106_核心知识点/PlaywrightCLI浏览器自动化边界|Playwright CLI 浏览器自动化边界]]


微软 2026 年初开源了一个叫 Playwright CLI 的命令行工具，专门给 AI 编程助手（如 Claude Code、GitHub Copilot）用的。你用自然语言告诉 AI “帮我抓这个页面的前 100 条评论”，AI 就能通过 Playwright CLI 操控浏览器帮你完成。

本文是一份从零开始的上手教程。

---

## 这东西是什么？跟之前的方案有什么区别？

Playwright 可能大家很早就听过，微软出的浏览器自动化框架。之前 AI 要控制浏览器，主要走的是 Playwright MCP（Model Context Protocol）方案——每次操作，MCP 会把整个页面的无障碍树、截图等数据一股脑塞进 AI 的上下文窗口。

问题是，一个典型的浏览器自动化任务，MCP 方案大约消耗 114,000 个 token，而 CLI 方案只需要约 27,000 个——差了大约 4 倍。

核心区别：Playwright CLI 把页面快照保存到磁盘上的 YAML 文件，而不是塞回 AI 的上下文窗口。AI 收到的只是一个文件路径，需要的时候再去读。

这就意味着 AI 在执行多步骤任务时不会因为上下文爆满而“失忆”。

---

## 环境搭建（三步搞定）

### 1. 安装 Node.js

去 nodejs.org 下载安装，要求 18 以上版本。装完验证：

```
node --version
```

### 2. 安装 Playwright CLI

通过 npm 全局安装：

```
npm install -g @playwright/cli@latest
```

验证安装：

```
playwright-cli --version
```

然后初始化工作空间并安装浏览器：

```
playwright-cli install
playwright-cli install-browser
```

### 3. 安装 AI 编程Agent

目前支持最好的两个框架：

* **Claude Code（Anthropic 出品）**
* **Codex（OpenAI 出品）**

装好框架后，还需要安装 Playwright CLI 的 Skills 文件：

```
playwright-cli install --skills
```

这会在项目目录下生成 `.claude/skills/playwright-cli/` 文档，AI 编程助手会自动发现并使用这些技能说明。

---

## 基础命令速查

Playwright CLI 的操作逻辑很简单：**打开浏览器 → 拿到页面快照 → 用元素编号交互。**

```
# 打开浏览器并导航到页面
playwright-cli open https://example.com

# 获取页面快照（返回元素引用编号，如 e15、e21）
playwright-cli snapshot

# 用编号点击元素
playwright-cli click e15

# 输入文字
playwright-cli type"Hello World"

# 按键
playwright-cli press Enter

# 截图
playwright-cli screenshot

# 关闭浏览器
playwright-cli close
```

两个常用参数：

* `--headed：打开可见的浏览器窗口（默认是无头模式，后台静默运行）`
* `--persistent：保存 cookie 和登录状态到磁盘，下次不用重新登录`

```
playwright-cli open https://example.com --headed --persistent
```

还有一个很实用的可视化面板：

```
playwright-cli show
```

这个命令会打开一个仪表盘，里面展示所有运行中的浏览器会话的实时画面，点击任意会话可以放大查看甚至手动接管控制。

---

## 核心概念：Skills 机制

Skills 是这套方案里最关键的效率杠杆。

简单说，Skill 就是一份“操作说明书”。微软官方把playwright-cli的使用方式直接封装成了一个 skill，这样 AI Agent 就可以照着这份操作说明书知道怎么来使用playwright-cli操作浏览器。

同样的，当你让 AI 在目标网站完成你的任务时（注册公众号、发小红书笔记），就像人类一样，AI第一次也需要自己摸索了解网站的操作流程（消耗大量 token）。而当任务完成后，你也可以把这次摸索的结果提炼成一个 Skill 文件，让 AI 照着做就行了。

**效率对比：**

| 执行方式 | Token 消耗 | 是否依赖 AI | 适用场景 |
| --- | --- | --- | --- |
| 首次探索 | 高（约 41% 上下文窗口） | 是 | 新任务第一次跑通 |
| 用 Skill 执行 | 低（约 5% 上下文窗口） | 是 | 重复执行的任务 |
| 导出为脚本 | 零 | 否 | 完全固定的标准化流程 |

**创建 Skill 的方法：**

第一次任务跑完后，让 AI 自己总结流程：

> “请把刚才的操作流程提炼成一个 Skill，保存到项目目录下。”

AI 会生成一个包含步骤说明、避坑提示的 Skill 文件。下次执行同类任务，直接引用这个 Skill。

---

## 实战案例

### 案例 1：抓取电商评论导出 CSV

让 AI 执行：

> “用 Playwright CLI 打开 [商品页面]，抓取前 100 条用户评论，包括用户名、评分和评论内容，保存为 reviews.csv。”

首次执行 AI 会自己摸索翻页、定位元素，跑完后让它固化为 Skill。之后同类商品的评论抓取，效率提升约 8-10 倍。

最终还可以让 AI 把整个流程导出为一段 Shell 脚本，以后完全脱离 AI 运行，token 成本归零。

### 案例 2：Markdown 文章自动发布

痛点：Markdown 文章粘贴到发布平台，格式错乱、图片丢失。

解决思路分两步：

1. **预处理：**

   AI 写一个 Python 脚本，把 Markdown 中的网络图片下载到本地，并转换为 HTML 格式
2. **自动发布：**

   Playwright CLI 打开发布平台 → 创建文章 → 粘贴 HTML → 上传本地图片 → 发布

而将整个流程固化为 Skill 后，以后只需要给 AI 一个文件路径，全自动完成。

### 案例 3：Web 应用自动化测试

让 AI 读你的代码，自动生成测试用例，然后通过 Playwright CLI 逐条执行：注册 → 登录 → 核心功能操作 → 验证结果。

配合定时任务，可以做到每天自动跑一遍回归测试。

---

## CLI vs MCP：应该怎么选？

CLI 更适合需要在有限上下文窗口内平衡浏览器操作与代码编写的高吞吐场景，MCP 更适合需要持久状态和对页面结构做深度推理的探索性自动化场景。

简单判断标准：

* 任务步骤多、流程相对固定 → **用 CLI**
* 需要 AI 反复观察页面、动态决策 → **用 MCP**
* 两者可以在同一个项目中混合使用

---

## 一些常见问题

**Q：支持哪些浏览器？** Chrome（默认）、Firefox、WebKit、Edge。指定方式：

```
playwright-cli open --browser=firefox
```

**Q：怎么处理需要登录的网站？** 用 `--persistent` 参数保持登录态。登录一次后，后续操作不需要重新登录。

**Q：Skills 的源码在哪？** Playwright CLI 的官方 npm 包是 `@playwright/cli，`Skills 文档在 GitHub 仓库 的 `skills/` 目录下。
