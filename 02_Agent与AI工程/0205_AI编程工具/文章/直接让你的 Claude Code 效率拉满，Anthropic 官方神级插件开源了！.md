---
title: 直接让你的 Claude Code 效率拉满，Anthropic 官方神级插件开源了！
author: java1234
date: 小锋小锋
url: https://mp.weixin.qq.com/s?__biz=MzIxNTAwNjA4OQ==&mid=2247571379&idx=1&sn=68203641dfafd11a34ccdf89d5966a8b&chksm=96d4d92f8376b98ebeb2aac3814d3f3d2586c2f25235bb9ab413ba5e8209613005d81303f610&mpshare=1&scene=24&srcid=0602aaI15lbD76azzZVSeYND&sharer_shareinfo=f46089d317e57955f96dceba6ffa250b&sharer_shareinfo_first=f46089d317e57955f96dceba6ffa250b#rd
---

大家好，我是锋哥。

今天介绍一个Anthropic官方发布开源项目 - claude-plugins-official，目前已经接近 3万 Star ，非常受欢迎。

---

## 目录

* 写在前面：插件到底解决什么问题
* 这个项目是什么
* 仓库里有什么
* 三类插件，各干各的活儿
* 一分钟上手安装
* 插件长什么样（目录结构）
* 值得先装的几个「神级」插件

---

## 写在前面：插件到底解决什么问题

如果你已经在用 **Claude Code**，大概率遇到过这种情况：每次换项目，都要重新配一遍 MCP、斜杠命令、团队规范；想接 GitHub、Linear、Figma，又得自己翻文档、改配置文件。

**插件（Plugin）** 就是把这一堆能力打包成「即插即用」的小模块。装一个，就多一套命令、技能或外部工具连接，不用从零搭环境。

而 Anthropic 最近把自家维护的插件目录 **完整开源** 到了 GitHub——这就是今天要聊的 claude-plugins-official。

---

## 这个项目是什么

简单说：**Anthropic 官方的 Claude Code 插件应用商店源码**。

* 由 Anthropic 团队维护，质量有底线
* 已内置在 Claude Code 里，**不用额外注册市场**，打开就能逛
* 目前收录 **一百多款** 插件（数量会持续更新）

---

## 仓库里有什么

打开仓库，核心结构很清晰：

| 路径 | 说明 |
| --- | --- |
| `plugins/` | Anthropic 自研插件，如 LSP、代码审查、提交助手等 |
| `external_plugins/` | 第三方合作伙伴插件，如 GitHub、Figma、Linear、Playwright 等 |
| `.claude-plugin/marketplace.json` | 市场清单，Claude Code 靠它识别可安装的插件 |

此外还有 `example-plugin` 示例，想自己写插件的同学可以直接当模板抄。

---

## 三类插件，各干各的活儿

不用记名字，按「能帮你干什么」来理解就行：

**1. 写代码更顺手（LSP 类）**

给 Claude 装上「语言服务器」后，改完代码能立刻看到类型错误、lint 提示，还能跳转定义、查引用——体验接近 VS Code。常见有：

* `typescript-lsp`、`pyright-lsp`（Python）
* `rust-analyzer-lsp`、`gopls-lsp`（Go）
* `jdtls-lsp`（Java）、`clangd-lsp`（C/C++）等

**2. 开发流程更省心（工作流类）**

* `feature-dev`：结构化做功能开发
* `code-review`、`pr-review-toolkit`：代码 / PR 审查
* `commit-commands`：规范化 Git 提交
* `security-guidance`：安全相关提醒

**3. 对接你每天都在用的工具（集成类）**

在 `external_plugins` 里能找到 GitHub、GitLab、Atlassian、Asana、Figma、Vercel、Sentry、Playwright、Terraform 等。装好后，Claude 可以直接调这些服务的 API，而不是让你把 issue 链接复制粘贴进对话框。

---

## 一分钟上手安装

Claude Code 里已经预置了 `claude-plugins-official` 市场，**零配置** 就能用。

**方式一：命令安装**

```
/plugin install {插件名}@claude-plugins-official
```

例如装 TypeScript 语言支持：

```
/plugin install typescript-lsp@claude-plugins-official
```

**方式二：图形界面**

在终端输入 `/plugin`，切到 **Discover** 标签页浏览、一键安装。

完整目录也可以在网页查看：claude.com/plugins

---

## 插件长什么样（目录结构）

每个插件都遵循统一约定，方便维护和二次开发：

```
plugin-name/  
├── .claude-plugin/  
│   └── plugin.json      # 插件元信息（必填）  
├── .mcp.json            # MCP 服务配置（可选）  
├── commands/            # 斜杠命令（可选）  
├── agents/              # Agent 定义（可选）  
├── skills/              # Skill 技能（可选）  
└── README.md            # 说明文档
```

有的插件不自带完整目录，而是从别的 Git 仓库「打包」一批 Skill 进来，市场清单里会用 `strict: false` 和 `skills` 数组声明——对普通用户透明，装完就能用。

---

## 值得先装的几个「神级」插件

没有「唯一正确答案」，按日常技术栈选即可。下面是多数人反馈好用的起步组合：

| 插件 | 适合谁 | 一句话 |
| --- | --- | --- |
| `typescript-lsp` / `pyright-lsp` | 前端 / Python 开发者 | 让 AI 改代码时「看得见」报错 |
| `feature-dev` | 做新功能的同学 | 把需求拆成可执行的开发步骤 |
| `commit-commands` | 经常提交代码的人 | 提交信息更规范、更省事 |
| `github` 或 `gitlab` | 用 Git 托管的团队 | 直接查 PR、Issue，少复制链接 |
| `playwright` | 做 Web 测试的同学 | 浏览器自动化一条龙 |

## 

> 项目地址：https://github.com/anthropics/claude-plugins-official

---

[2026年，锋哥又开始收Java编程学员了！目前活动，送AI编程+Python+AI大模型VIP。。。](https://mp.weixin.qq.com/s?__biz=MzIxNTAwNjA4OQ==&mid=2247571315&idx=2&sn=4bb42e912d7c5986d3f5cd6a7e1c2e5d&scene=21#wechat_redirect)

#### 最近锋哥录制了一些AI编程视频教程 高清视频+源码+领取。 ``` 扫描下方公众号【小锋学AI 】回复：888， 可获取下载链接 👇👇👇 👆长按上方二维码 2 秒 回复「888」即可获取 ```