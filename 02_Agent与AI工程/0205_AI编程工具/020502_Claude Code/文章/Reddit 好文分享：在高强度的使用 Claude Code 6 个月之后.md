---
title: Reddit 好文分享：在高强度的使用 Claude Code 6 个月之后
author: AgenticCoding 实验室
date: 
url: https://mp.weixin.qq.com/s?__biz=MzAxMDA0MTE5Mw==&mid=2247484555&idx=1&sn=f28f2cf2a88d4a579fdc931221f2d62e&chksm=9ae76d3925c86749d3dd92e40e4dd1fd7b665e7df9119f12f2b1c2f15e7cc763800697d69aae&mpshare=1&scene=24&srcid=1117XJ3Xipsb3oCM79FUUb4I&sharer_shareinfo=4e606eb3ad5b5d938fd0066e316039fc&sharer_shareinfo_first=4e606eb3ad5b5d938fd0066e316039fc#rd
---

# Reddit 好文分享：在高强度的使用 Claude Code 6 个月之后

又看到一篇关于 Claude Code 的实践分享，作者是一名软件工程师，过去七年左右一直在开发生产级 Web 应用，在项目中积极的使用 AI 来进行开发。并且作者在过去的 6 个月里，借助 Claude Code 完成了一个古老项目（包括前后端）的迁移工作，单人重写 30w 行代码，所以有自信分享和探讨使用 Claude Code 的一些技巧和经验。

## 技能自动激活系统（Skills Auto-Activation System）

通过 Hooks 保证 Skill 的正确调用。

**1. 首先建立技能配置文件 `skill-rules.json`**。

`skill-rules.json` 内包含了技能的关键词（layout、database）、意图正则（"(create|add).\*?(feature|route)"））、特定的触发文件路径、特定的文件内容等。

```
{  
  "backend-dev-guidelines":{  
    "type":"domain",  
    "enforcement":"suggest",  
    "priority":"high",  
    "promptTriggers":{  
      "keywords":["backend","controller","service","API","endpoint"],  
      "intentPatterns":[  
        "(create|add).*?(route|endpoint|controller)",  
        "(how to|best practice).*?(backend|API)"  
      ]  
    },  
    "fileTriggers":{  
      "pathPatterns":["backend/src/**/*.ts"],  
      "contentPatterns":["router\\.","export.*Controller"]  
    }  
}  
}
```

**2. Hooks 处理**

`UserPromptSubmit Hook` （用户提交 Prompt 后运行）根据用户输入内容和 `skill-rules.json` 进行匹配，匹配到合适的技能时，再次向 Prompt 中注入需要使用的技能信息，确保 Claude 可以调用到技能。

## Skill 最佳实践

**`SKILL.md` 文件在 500 行以下**，其他内容在 `SKILL.md` 中引用以节省上下文。比如一个前端研发技能目录：

```
skills/  
    frontend-dev-guidelines/  
        SKILL.md // 500 行以下  
        resources/  
            compoents.md  
            router.md  
            data-service.md  
            ...md
```

此外，编写**可复用的脚本**也很重要，在一些测试、mock 场景中将这些脚本放入技能中，可以减少大量的重复工作。

## 如何使用 CLAUDE.md

一句话：**技能处理所有"如何写代码"的指导原则，CLAUDE.md 处理"这个特定项目如何工作"**。

编写代码的最佳实践放到技能中，而项目的有特定信息放到 CLAUDE.md，不要在 CLAUDE.md 放置太多内容。

## 文档系统

开发文档极其重要，能有效防止 Claude 失忆和幻觉。

当有新任务开始时：

**1. 创建任务目录**

```
mkdir -p ~/git/project/dev/active/[task-name]/
```

**2. 创建文档：**

* • [task-name]-plan.md - 接受的计划
* • [task-name]-context.md - 关键文件、决策
* • [task-name]-tasks.md - 工作检查清单

**3. 定期更新：**  
立即标记任务完成。

继续任务时：

* • 检查 `/dev/active/` 中的现有任务
* • 在继续之前阅读所有三个文件
* • 更新"最后更新"时间戳

最后，**人工审查非常重要**，这一步需要多一点时间，后面会轻松很多。

## 其他技巧

1. 1. **合适的人工介入**，如果开发者可以一眼发现问题并且轻松解决，不要犹豫，直接解决。
2. 2. **选择合适的工具**，语音输入，[Memory MCP - 使用本地知识图谱打造“有记忆”的智能助手](https://mp.weixin.qq.com/s?__biz=MzAxMDA0MTE5Mw==&mid=2247484542&idx=1&sn=15f9f0a87e179a071060bedfac291c36&scene=21#wechat_redirect) 等等。
3. 3. **使用脚本**，编写脚本提高开发、测试效率
4. 4. **学习提示词技巧**，[你的提示词技巧可能过时了！最新版Claude 提示词工程最佳实践发布！](https://mp.weixin.qq.com/s?__biz=MzAxMDA0MTE5Mw==&mid=2247484547&idx=1&sn=376f7a2b3ca8c839a059cbe8e3e9d6f3&scene=21#wechat_redirect)