---
title: Ralph Loop：让AI连续工作37小时不休息的“永动机”架构
author: 思维与成长
date: 
url: https://mp.weixin.qq.com/s?__biz=MzA4MTAwOTgzNA==&mid=2456298541&idx=2&sn=f84192ba20fec96c94214fce42350dea&chksm=89d6f66f549bdef5c0949816a4a93b9ea72ae5fcce5036f10484d4ad5113217a3b22b1e70a5b&mpshare=1&scene=24&srcid=0414gaUdwBVgSz2hIup87HbZ&sharer_shareinfo=86efeed3e929314dec54ff8049701ae2&sharer_shareinfo_first=86efeed3e929314dec54ff8049701ae2#rd
---

# Ralph Loop：让 AI 连续工作 37 小时写完 2000 行需求的“永动机”架构

> **导读**：大多数 AI 编程助手（Cursor, Copilot）都有一个死穴：**上下文窗口爆炸**。写着写着它就变笨了，忘了前面的需求。开发者 @d4m1n 分享了一种名为 **“Ralph Loop”** 的架构，让 AI 连续运行 37 小时，完成了 250 个任务，实现了真正的“无人值守开发”。
>
> 本文将带你拆解这个架构，看它是如何把 AI 从“辅助驾驶”变成“自动驾驶”的。

---

## 一、 为什么传统的 AI Coding 会失败？

你一定遇到过这种情况：刚开始，AI 写代码如有神助。但随着项目变大，文件变多，对话轮次增加，AI 开始**胡言乱语**：

* 它忘了之前定义过的变量。
* 它开始修改不该动的文件。
* 它的逻辑前后矛盾。

这是因为 **Context Window（上下文窗口）** 满了。就像一个人的短期记忆只有那么多，塞满了新的，旧的就被挤出去了。

**Ralph Loop 的核心思想：****状态不应该存在于 Context 里，而应该存在于文件系统和 Git 中。**

---

## 二、 Ralph Loop 架构拆解

Ralph Loop 不是一个黑盒模型，而是一个**工作流（Workflow）**。它的运行逻辑非常像一个人类开发者：

1. **读取任务列表** (`tasks.json`)。
2. **启动一个全新的 Context**（这一步是关键！每次都是新的大脑，没有历史包袱）。
3. **挑选最高优先级的任务**。
4. **编写代码**。
5. **运行测试**（Playwright/Vitest）。
6. **提交代码 (Git Commit)**。
7. **退出**。
8. **重复步骤 1**。

通过这种“**用完即焚**”的策略，AI 永远处于最清醒的状态，永远不会因为上下文过长而“变笨”。

---

## 三、 如何在 10 分钟内搭建 Ralph Loop？

@d4m1n 把这套复杂的流程封装成了一个简单的 CLI。

### 1. 初始化项目

随便你用什么栈（Next.js, Python, etc.），但必须安装 **测试框架**。

> *Mikko 注：没有测试的 Agent 就是脱缰的野马。测试是它的缰绳。*

### 2. 安装 Ralph Loop

```
npx @pageai/ralph-loop
```

这会在你的项目里生成一个 `.agent/` 目录，里面包含了：

* `PROMPT.md`：核心指令。
* `tasks.json`：任务列表。
* `skills/`：AI 的技能包。

### 3. 生成 PRD（产品需求文档）

这是最关键的一步。你不需要手写 JSON，直接告诉 AI：

> “我要做一个类似 Twitter 的应用，支持 Google 登录，用 Next.js…”

Ralph Loop 自带的 `prd-creator` 技能会把你的脑图转换成结构化的任务列表。

### 4. 启动 Docker 沙箱

为了安全，Ralph Loop 运行在 Docker 容器里。

```
docker sandbox run claude .
```

### 5. 启动循环

```
./ralph.sh -n 10
```

这行命令的意思是：“连续执行 10 个任务循环”。你可以去喝咖啡了，Ralph 会一个接一个地把任务做完。

---

## 四、 2026 年的核心竞争力：不再是写代码

Ralph Loop 的出现，标志着 Coding 的范式转移。

以前，你的能力体现在：**手速、API 记忆、Debug 能力**。现在，你的能力体现在：**清晰描述需求（PRD）、设计测试用例、验收成果**。

你不再是**搬砖的工人**，你是**包工头**。你需要学会如何给 AI “派活”，如何定义“什么是做好了”。

**Mikko 辣评：**Ralph Loop 这种架构其实揭示了 Agent 的本质：**Agent = LLM + Loop + State (Filesystem)**。不要迷信模型的大小，**流程（Flow）** 才是决定 AI 能否落地的关键。如果你还在用 Chat 界面一句一句地“哄” AI 写代码，那你已经落后这个版本了。

---

**参考链接：**

* @d4m1n 的原推
* Ralph Loop GitHub