> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: 让 Claude Code 自己修 Bug:一套配置 + 钩子,告别每天重复教 AI
author: 陈大猫AI
date:
url: https://mp.weixin.qq.com/s?__biz=MzUyNjAzMjM4Mw==&mid=2247485069&idx=1&sn=941e836e58cda7093075e261454faad3&chksm=fb8a8e335ea29d8bd1bd0acd1fcfac173ce3b3d2ce574f2e6562344d87cebb31d6965dee5681&mpshare=1&scene=24&srcid=0605MQQrVy8A2J3Q5zIw2FJX&sharer_shareinfo=3669f5a41b08bee80600948358161ee0&sharer_shareinfo_first=3669f5a41b08bee80600948358161ee0#rd
---

用 Claude Code 写代码的人,大概都熟悉这个循环:它写完,测试报错,你解释哪儿错了,它修好——顺手又把别处搞坏。开个新会话,昨天花二十分钟修的 bug,今天照样复发。

Claude Code 的创始人 Boris Cherny 最近在采访里爆了个料:他们团队内部已经不再人工修复 Claude 的错误,而是让 Claude 自己修。靠的就是一套配置——让它自动发现错误、自动修复、并且**记住别再犯**。

我照着搭了一套,跑了一段时间,确实顺手不少。这篇把配置讲清楚——顺便把网上流传那版里几个会让你照着配就翻车的坑,一并改对。

## 为什么它总在重复犯错

根本原因只有一个:Claude 没有跨会话记忆。每次新会话都从零开始,上次踩的坑、纠正过的规则,它通通不记得。

所以解题思路也很清晰:把「不该做的事」沉淀成它每次开工前必读的东西,再用钩子在它干活的当口实时拦错。下面一层层拆。

## 01 · 会自己长大的 CLAUDE.md

`CLAUDE.md` 是 Claude 每次开始工作都会读的项目说明。关键不是拿它写「人格设定」,而是写具体、可执行的规则,而且让它随着每次犯错不断扩充。

我的 `CLAUDE.md` 里有两节。一节是固定规则:

## 规则

- 不要重构 / 重命名 / 清理无关代码

- 所有数据库查询走 services/,别写进组件

- 改完代码必跑 tsc 类型检查,失败先修再继续

- 提交前缀:feat / fix / docs / refactor / test / chore

- 不用 enum,用字面量联合类型

- 不要 force push,会覆盖共享历史

另一节才是精髓——「从错误中学到的」:

## 从错误中学到的

- 不要把两种现有写法折中,选一种跟到底

- 不确定目录结构时,先查现有文件再动手

- stop-loss 函数必须返回数字,不是布尔值(踩过生产事故)

每次纠正完 Claude,在末尾补一句「把这条更新进 CLAUDE.md,别再犯」。坚持下来,这份文件会沉淀成项目里它犯过的所有错的清单,开工前先读一遍,错误率自然往下走。

一个提醒:官方建议单个 CLAUDE.md 尽量控制在 200 行以内。不是硬限制,而是越长越占上下文、规则执行率越低。所以别把它堆成百科,只留真正高频踩的坑。

## 02 · PostToolUse 钩子:写完立刻查

钩子(hooks)会在特定时机自动触发。`PostToolUse` 在 Claude 写完或编辑完文件之后立刻运行,在问题扩散前就把它摁住。

思路是:每写完一个文件,自动跑格式化和类型检查,让 Claude 在**同一轮对话里**就看到报错、立刻修掉。

这里要敲黑板——网上流传那版的写法是错的,照抄不生效。两个坑:

•matcher \*\*不支持 Write(\*.ts) 这种通配\*\*,得用精确工具名、多个用 | 隔开,比如 Edit|Write

•拿文件路径不是 $file 这种环境变量,而要从标准输入的 JSON 里解析

正确写法长这样:

"PostToolUse": [

{

"matcher": "Edit|Write",

"hooks": [

{

"type": "command",

"command": "jq -r '.tool\_input.file\_path' | xargs npx prettier --write"

}

]

}

]

没有钩子时:Claude 写完 5 个文件,你跑一次检查,蹦出十几个报错再逐一解释。有钩子时:每个文件落地即检查,错误实时修。差别很大。

## 03 · Stop 钩子:质量门禁 + 自愈闭环

这是最关键的一个。`Stop` 钩子在 Claude 说「我完成了」的时候触发——在它真正收工前,验收一下活儿到底达不达标。

最实用的玩法是跑测试,不过就让它自己接着修。强制 Claude 继续的正确机制,是钩子输出一段 `decision: block` 的 JSON:

"Stop": [

{

"hooks": [

{

"type": "command",

"command": "npm test || echo '{\"decision\":\"block\",\"reason\":\"测试没过,继续修\"}'"

}

]

}

]

测试失败 → 钩子吐出 `block` + 原因 → Claude 读到反馈,自动继续工作,不用你插手。

但有一条**必须记牢**:只在**测试失败时**才 block,通过了就放行(正常退出)。如果你写成「无条件 block」,Claude 会永远收不了工,陷进无限循环,把 token 烧穿。这是这一步最容易翻车的地方。

想要更聪明的验收,Stop 钩子还支持 `type: "prompt"`——直接让 Claude 自己评估:

{

"type": "prompt",

"prompt": "检查任务是否真的完成:有没有 bug、漏掉的边界情况、缺的测试。有问题就继续修,没问题再确认完成。"

}

## 04 · PreToolUse 钩子:动手之前就拦下

`PreToolUse` 在 Claude **执行工具之前**运行,用来过滤输入、拦截代价高的危险操作。

比如它要读一个一万行的日志,钩子先过滤、只喂给它带 ERROR 的那几十行;比如它要写 `.env`,直接拦下,操作根本不会发生。

又一个要改对的坑:拦截要用 `exit 2`,不是 `exit 1`。`exit 1` 只是个非阻塞的报错,挡不住操作;`exit 2` 才会真正阻止工具执行。

"PreToolUse": [

{

"matcher": "Write",

"hooks": [

{

"type": "command",

"command": "jq -r '.tool\_input.file\_path' | grep -q '\\.env' && { echo '禁止写 .env' >&2; exit 2; } || exit 0"

}

]

}

]

## 05 · 自动重试,但务必设上限

对那种要试好几次的任务,可以直接在指令里给它定规矩:

修复这些失败的测试,最多试 3 次。

每次之后:跑测试 → 过了就完成;没过就读报错、换个思路。

不要用同一个修法试两次。3 次还不行,就说清楚试了什么、还差什么。

它和第 03 步的 Stop 钩子一配,就是个**自愈闭环**:写代码 → 测试 → 没过读错误重来 → 直到通过或用完次数。

但**上限是底线**。不设次数,它可能在一个修不动的 bug 上无限重试,token 哗哗地烧。

## 06 · 跨会话记忆:让经验跨天留存

钩子治的是单次会话里的错,跨天复发的错得靠记忆。Claude Code 里有个 `/memory` 命令,用来管理 `CLAUDE.md` 这类记忆文件、查看自动记下的条目。

另外它也在补**自动记忆**的能力——在后台整理历史会话、沉淀持久知识。不过这块有些功能(比如社区在传的「Dreaming」)目前还没正式进官方稳定文档,先知道方向就行,别当成已落地的配置去抄。

可以这么理解分工:`CLAUDE.md` 管项目级规律,`/memory` 管会话级学习,自动记忆负责把它们整合留存。

## 完整配置,可以直接抄

把上面几块拼起来,一份能用的 `settings.json` 大致是:

{

"permissions": {

"allow": [

"Read", "Edit", "Write(src/\*\*)", "Write(tests/\*\*)",

"Bash(npm test \*)", "Bash(npx tsc \*)",

"Bash(npx prettier \*)", "Bash(npx eslint \*)"

],

"deny": [

"Read(\*\*/.env\*)", "Write(\*\*/.env\*)",

"Bash(rm -rf \*)", "Bash(git push \*)"

],

"defaultMode": "acceptEdits"

},

"hooks": {

"PostToolUse": [

{

"matcher": "Edit|Write",

"hooks": [

{ "type": "command", "command": "jq -r '.tool\_input.file\_path' | xargs npx prettier --write" }

]

}

],

"Stop": [

{

"hooks": [

{ "type": "command", "command": "npm test || echo '{\"decision\":\"block\",\"reason\":\"测试没过,继续修\"}'" }

]

}

]

}

}

## 配置前后,差的是什么

**配置之前**:Claude 写完,你跑测试 4 个失败,逐一解释;它修好 3 个又引入 1 个新 bug,你再解释;一个功能来回四十多分钟,同样的错明天还来。

**配置之后**:写完每个文件钩子自动格式化加检查,说完成测试自动跑,不过它自己读错误接着修,`CLAUDE.md` 挡掉昨天的坑、记忆挡掉上周的坑。一个功能十来分钟,大半时间你不用盯着。

说到底,和 AI 协作真正省时间的地方,不在于每次把话说得多清楚,而在于花一次,把「别再犯」沉淀进规则和钩子里。这才是会复利的那部分。

最后补一句:上面的钩子语法我按官方文档校过一遍,和网上很多转发版不一样的地方,基本都是那些版本抄错了——以官方文档为准,别照着错的配。