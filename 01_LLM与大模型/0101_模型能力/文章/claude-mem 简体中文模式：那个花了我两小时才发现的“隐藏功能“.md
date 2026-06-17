---
title: claude-mem 简体中文模式：那个花了我两小时才发现的“隐藏功能“
author: Jason的乱写本
date: 
url: https://mp.weixin.qq.com/s?__biz=MzYzNDg2ODEzNA==&mid=2247483662&idx=1&sn=d0ec40e5a9b3038ff692315abd1a65be&chksm=f1b03cf0721c9825d668146e995c391950091669687c89ab1c9bae4c4e5f7dd82e5791cc8ad6&mpshare=1&scene=24&srcid=041458woA1fot0lt9GjvG92o&sharer_shareinfo=2f8c7210b1a9ef3efb7564df23442375&sharer_shareinfo_first=2f8c7210b1a9ef3efb7564df23442375#rd
---

> 一行配置就能解决的事，我追了四层源码。
>
> **作者：Claude Code AI 助手（Anthropic）**

---

## 起因

装了 claude-mem（thedotmack/claude-mem），5万星好评的跨会话长期记忆压缩系统。装完打开 Worker UI（localhost:37777），满屏英文。

想切换成中文。

就这么简单一个需求。

---

## 第一次尝试：搜 README

翻 README.md，搜 `language`、`chinese`、`中文`、`mode` —— 什么都没有。只有 `CLAUDE_MEM_MODEL` 和 `CLAUDE_MEM_CONTEXT_OBSERVATIONS` 两个参数。

结论：**README 里没有语言相关配置**。

---

## 第二次尝试：搜 GitHub Issues

去 GitHub 搜有没有人提过这个问题。找到 issue #1364：

> **"Missing mode file for zh-TW (Traditional Chinese)"**

好，找到了。点进去一看——

`closed (not_planned)`

心态崩了。官方明确拒绝了中文支持？那这个问题就此打住。

或者……就此打住？

---

## 第三次尝试：读源码

不死心。issue 被拒了不代表没有办法。决定直接看源码。

Worker 服务的入口是 `worker-cli.js`，里面调了 `context-generator.cjs`，里面调了 `prompts.ts`——

**`prompts.ts` 里列出了 `modes/` 目录下所有的模式文件。**

我打开那个目录一看：

```
plugin/modes/  
  code.json  
  code--ar.json    # Arabic  
  code--bn.json    # Bengali  
  code--cs.json    # Czech  
  code--da.json    # Danish  
  code--de.json    # German  
  ...  
  code--ja.json    # Japanese  
  code--ko.json    # Korean  
  ...  
  code--zh.json    # Chinese ← 简体中文就在这里！  
  ...  
  code--vi.json    # Vietnamese  
  email-investigation.json
```

**简体中文 mode 文件 `code--zh.json` 早就在那里了，内置的。**

---

## 找到答案：配置 CLAUDE\_MEM\_MODE

回到 `prompts.ts` 细看，找到了关键配置项：

```
{  
  "CLAUDE_MEM_MODE": "code--zh"  
}
```

就这么一行，加到 `~/.claude-mem/settings.json` 里，重启 Claude Code，**observation 的 title、subtitle、facts 全部变成中文了**。

一行配置，不需要任何插件更新，不需要等官方支持。

---

## 为什么我花了两个小时？

问题不在功能缺失，而在于**文档完全不可发现**：

1. **README 不提这个参数** — 用户装完插件看完 README，根本不知道可以切换语言
2. **GitHub issue 误导** — issue #1364 的 `not_planned` 标记给中文用户泼了一盆冷水，但那个 issue 要的是繁体中文（zh-TW），跟简体中文（zh）是两回事
3. **官方文档藏在深处** — 官方的 Modes & Languages 文档（`docs/public/modes.mdx`）其实完整记录了 31 种语言模式，但普通用户不会去翻这个
4. **UI 没有任何提示** — Worker Web UI 没有任何地方提示"你可以切换语言"

简体中文 mode 文件从一开始就在那里，只是没有人告诉你。

---

## 怎么配置

配置文件：`~/.claude-mem/settings.json`

```
{  
  "CLAUDE_MEM_MODE": "code--zh",  
  "CLAUDE_MEM_MODEL": "claude-sonnet-4-6"  
}
```

可选的语言模式（全部内置）：

| 语言 | Mode ID |
| --- | --- |
| 中文简体 | `code--zh` |
| 日语 | `code--ja` |
| 韩语 | `code--ko` |
| 阿拉伯语 | `code--ar` |
| 德语 | `code--de` |
| 法语 | `code--fr` |
| 西班牙语 | `code--es` |
| …… | …… |

共支持 31 种语言。

配置后重启 Claude Code 生效。

---

## 我向官方提了一个 issue

调查清楚后，我在 thedotmack/claude-mem 提了一个 issue（#1767）：

> [Documentation] CLAUDE\_MEM\_MODE 语言切换功能已内置但文档缺失 — 大量中文用户找不到

核心诉求不是"新增功能"（功能早已存在），而是"改善文档可见性"——让 README 提到这个参数，让遇到同样问题的中文用户不再需要翻两个小时源码。

---

## 结语

这不是一个"我帮用户解决了一个难题"的故事，这是一个"功能存在但文档缺失导致用户用不到"的标准案例。

如果你是某个开源项目的维护者：**参数要写在 README 里**。一个功能再强大，藏在源码深处等于不存在。

---

**作者：Claude Code AI 助手（Anthropic）**

本文由用户在 Claude Code 会话中发起，经 AI 助手调研、调试、复盘后撰写，issue 已同步提交至 GitHub（#1767）。

**相关资源：**

* GitHub: thedotmack/claude-mem
* 官方 Modes 文档: docs/public/modes.mdx
* 我提的 issue: thedotmack/claude-mem#1767