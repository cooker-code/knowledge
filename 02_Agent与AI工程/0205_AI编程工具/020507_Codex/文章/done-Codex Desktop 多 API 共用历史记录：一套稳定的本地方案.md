> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020507_Codex/020507_核心知识点/Codex工程使用与沙箱边界|Codex工程使用与沙箱边界]]
---
title: Codex Desktop 多 API 共用历史记录：一套稳定的本地方案
author: 起来学AI吧
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg4MDUzMDYwNQ==&mid=2247483675&idx=1&sn=0635fad83b8c0480d88d199b8e3eac01&chksm=ce7e6661a37166ec87c148134945b3907d3d6b48b0e025fdfb29fd2a1c68717c85339c5a864a&mpshare=1&scene=24&srcid=0526MGbn5FTXs1Xbty9wflnY&sharer_shareinfo=59fc823c47fa29b27ad06ab435b4188e&sharer_shareinfo_first=59fc823c47fa29b27ad06ab435b4188e#rd
---

换 API 之后，Codex Desktop 找不到以前的对话？

很多时候，问题不在账号，而在本机历史状态没有统一。

**核心做法：让不同 API 登录的 Codex，都读取同一套本地历史记录。**

## 先说结论

稳定方案有 4 个关键点：

共享 sessions

共享 archived\_sessions

修复数据库和索引

项目列表只保留真正项目

其中最容易漏掉的是 `archived_sessions`。

只共享 `sessions`，历史可能能显示，但归档、项目归属、侧边栏仍然可能异常。

## 需要统一哪些状态

Codex 的历史不只是一堆聊天文件。它通常包含这些部分：

.codex\sessions

.codex\archived\_sessions

.codex\session\_index.jsonl

.codex\state\_5.sqlite

.codex\.codex-global-state.json

可以简单理解为：`sessions` 是普通对话，`archived_sessions` 是已归档对话，数据库和全局状态负责索引、项目和侧边栏。

## 推荐目录结构

我最终使用的是一个独立共享目录：

CodexSharedHistory

├─ sessions

├─ archived\_sessions

├─ session\_index.jsonl

└─ state\_5.sqlite.snapshot

然后把 Codex 本地目录里的两个会话文件夹接过去：`sessions` 对应共享的 `sessions`，`archived_sessions` 对应共享的 `archived_sessions`。

这样无论换哪个 API，只要还在同一台电脑上，Codex 都能读到同一批历史文件。

## 项目列表不要乱修

**不要把所有历史工作目录都恢复成项目。**

Codex 历史里会出现很多临时目录。它们可能只是某次对话自动生成的工作区，不一定是真正长期维护的项目。

正确做法是：长期项目显示在“项目”里，临时目录回到普通“对话”列表。

## 工具箱应该保持克制

本地工具箱最后只保留几个常用动作：

Codex 修复

- 修复项目新建

- 修复历史 / 对话名



CPA 服务

- 启动 CPA

- 打开 Web 管理器

- 停止 CPA

其它低频操作，比如扫描、回滚、打开共享目录，放到维护工具里。

涉及历史数据库和状态文件的工具，按钮越多，误操作概率越高。工具箱应该降低心智负担，而不是把所有底层开关都摊在首页。

## 日常怎么用

**项目下显示“暂无对话”，或者不能新建会话：**

点击：修复项目新建

完全退出 Codex

等待几秒

重新打开

**对话标题乱了，或者历史显示不完整：**

点击：修复历史 / 对话名

重新打开 Codex

**需要 CPA/Sub2ap：**

启动 CPA

打开 Web 管理器

用完停止 CPA

Web 管理器可以通过本地入口打开，不需要每次手动填管理信息。

## 最终状态

多 API 登录：共用同一批历史

普通对话：正常显示

归档对话：正常归档和读取

项目列表：只保留真正项目

临时目录：不污染项目栏

**一句话总结：**Codex Desktop 多 API 共用历史，核心不是同步账号，而是统一本机历史状态。