---
title: Anthropic 不会告诉你的 18 个 Claude 设置，老用户全在偷偷用
author: AI 博物院
date: longyunfeigulongyunfeigu
url: https://mp.weixin.qq.com/s?__biz=MzIzNTcxOTU3OA==&mid=2247488442&idx=1&sn=91af5f8d045cde65b60bc631e7d1f806&chksm=e97c488c04d156797fa464920bf9ef3ba28966493a8debc5b1575a89fbaf942656a62ca4bc4e&mpshare=1&scene=24&srcid=0526oOA8WheXq56qCmg0rRoR&sharer_shareinfo=7e18873d9668436e0a3893789c604710&sharer_shareinfo_first=7e18873d9668436e0a3893789c604710#rd
---

X 上有个工程师叫 Mnilax，前两天列了 18 个 Claude 设置项。

我本来想随手划过去，扫到第三条停下来了。他说他把一个叫 `cache_control` 的断点位置改了一下，API 月度账单从 340 美元降到 87 美元。没换模型，没改 prompt，没缩用量。

一个设置放错了位置，吃了三个月冤枉钱。

这两年大家最爱说的一句话是 Claude 变笨了。同一个 prompt，半年前能跑出干净方案，现在输出又长又啰嗦，账单还涨。朋友里有人月度从 200 美元爬到 800，模型没换、量没翻倍。

按 Mnilax 这份清单看下来，多半不是模型的事，是默认值在吃钱。

Anthropic 产品迭代快，文档跟不上。

Mnilax 给的数字是这样的：Claude Code 二进制（v2.1.105）里能找到 125 个以上的设置键，官方文档讲了大约 40 个。剩下 85 个怎么知道？翻 GitHub issue、看工程师在 Discord 偶然贴出来的截图、凌晨一点拿 binary 硬 grep 字符串。

Claude.ai 这边情况类似。他统计下来 14 个真正影响输出质量的设置，藏在设置 → 能力 → 记忆 → 项目作用域这种三层菜单后面。还有 4 个连官方文档都没提，能用但 Anthropic 不告诉你。

默认值是给平均用户准备的，你不是平均用户。跑几个月之后，记忆开始漂移，插件越装越多，缓存断点放错位置，每一项都不致命，累加起来就是 Claude 变笨了的真相。

下面按 Claude.ai、Claude Code、API 三层铺一遍，每条都给一行话的修法。

## Claude.ai 这边，5 个先动起来

**1. 记忆开项目作用域**

路径：设置 → 能力 → 记忆 → 项目作用域。

记忆这个功能 2026 年 3 月才向 Free 和 Pro 全量开放。默认是全局存，你在某个项目里说的一句话，可能会在另一个完全不相关的聊天里被翻出来用。

Mnilax 举的例子：你在某个文件里跟 Claude 说过一句 Python 你喜欢用 tab 缩进，过两周写完全无关的销售文案，Claude 突然在中间塞段 Python 示例，缩进还特意用 tab。

把作用域改成按项目，项目内的记忆只留在项目里，能解决 80% 的漂移问题。

另外两件顺手的事：把不想被记的话题加到排除列表（医疗、薪资、客户名这种）；在任何聊天里直接说 `forget what you remembered about [话题]`，Claude 会现场比对自己的记忆库然后告诉你删了什么，不用进菜单。

我在 Obsidian 里那个项目跟 Claude 聊过的个人偏好，过几天会冒到完全不相干的代码任务里。原来以为是巧合，现在看是默认作用域的问题。

**2. 扩展思考默认设成 Light**

路径：聊天框 → 模型选择下拉 → Extended Thinking。

扩展思考会在回答前加一段 `<thinking>` 推理。对数学、调试、多步规划很有用，对总结、翻译、格式化、改写就是浪费。

Mnilax 算过账：在不需要思考的任务上，扩展思考多花 3 到 12 秒延迟，token 多用 20% 到 40%，答案质量没差别。

默认开 Full 等于在所有任务上都付了难题的钱。改成 Light，Claude 自己判断要不要思考。他推荐给朋友试一周，Opus 的 token 消耗下降 18% 到 25%。

**3. 把自定义样式当输出合同用**

很多人以为自定义样式是用来让 Claude 说话更正式或更简洁。

Mnilax 的角度更接近真实用法——它是一份注入到每次回答前的指令文件，相当于一份输出合同。

你可以贴一份 200 到 1500 字的指令文件，每次回答前都会先应用这份指令。他自己常用三份，分别给 X 草稿、代码审查、PDF 总结用。

给 X 草稿用的那份长这样：

```
开头必须给一个具体数字或一个具体名字。不要"我最近在想……"。  
句子尽量控制在 18 个字以内。  
不要破折号，除非节奏需要。  
禁用词：delve、leverage、robust、unlock、game-changing。  
列举三项以上用减号列表，不用编号。  
结尾必须是陈述句，不要反问。
```

三个样式轮换，替掉了他 80% 的常用 prompt 收藏。

我自己也是受害者。所有写作偏好以前都塞进每次对话开头，每次重粘一遍。看完这条决定整理三个 Style：公众号草稿、源码阅读笔记、知识库整理。

**4. 项目说明填上，别留空**

路径：任意项目 → 右上角三个点 → 编辑项目说明。

Mnilax 观察身边十几个重度 Claude 用户，70% 的项目说明是空的。

这个字段相当于注入到项目里每次对话的 system prompt。空着的话，你每次都要从零开始解释这个项目是干嘛的。填上之后，Claude 自动带着角色和默认立场进场。

他给的例子是一个 Polymarket 研究项目，说明里就一句话：这是一个 Polymarket 研究工作流。默认怀疑论。永远展示概率算式。永远不推荐下注，只描述期望值。

400 字以内，每月重读一次裁一裁。

**5. 网页搜索引用换成脚注模式**

路径：设置 → 能力 → 网页搜索引用 → 脚注。

默认是内联。内联引用的问题：你复制 Claude 的答案到别处（飞书文档、邮件、消息），那些引用标记就变成无效占位符。

切换到脚注，答案干净，参考资料统一列在最后。

剩下的 3 个 Claude.ai 设置 Mnilax 简略提了，我跟着列一下：

* • **过去聊天搜索**（Pro+）：默认关。开了之后注意它是关键词匹配不是语义匹配，要搜以前聊过的中国机器人那一段，输入框就得原样写出这几个字；写成上周聊的那个事这种模糊描述，搜不出来。
* • **Cowork 信任文件夹**：默认每次问，但你只要勾过一次以后不再提示的那个选项，那个文件夹就进了白名单。三月份测试时加的文件夹，Claude 一直在后台读你都忘了。每两个月去清一次。
* • **无痕模式**：侧栏点新建无痕聊天，或者快捷键 `Cmd+Shift+N`。它跳过的不是一件事是四件：记忆写入、聊天记录、过去聊天搜索索引、训练数据采集。涉及薪资、医疗、客户名的对话直接进无痕。

## Claude Code 那边，settings.json 里的 7 个

Claude Code 的设置都在 `~/.claude/settings.json`（全局）或 `.claude/settings.json`（项目）里，项目级覆盖全局。

**6. enabledPlugins：禁用就好，别卸载**

插件市场让安装变得太简单，卸载反而麻烦，结果就是大家装了一堆，没几个真在用。

Mnilax 的提醒是：每个激活插件会把它的钩子、SKILL.md 内容、工具 schema 全部加载进上下文预算。三个你早就忘了存在的插件 = 还没敲一个字，已经预扣三到八千个 token。

他自己开始审计时启用了 14 个，现在常驻 4 个。

修法：

```
{  
  "enabledPlugins": {  
    "formatter@acme-tools": true,  
    "deployer@acme-tools": false,  
    "old-experiment@personal": false  
  }  
}
```

设成 `false` 不是卸载，是装着但不加载。需要的时候 `/plugin enable name@marketplace` 临时启用。

这条我看完立刻去 audit 了自己配置，一堆 plugin 装了之后再没碰过，全在背后吃 token。

**7. permissions.deny 配上，但别全靠它**

`permissions.deny` 用来禁止 Claude 读 `.env`、跑 `rm -rf`、调 `sudo`。听起来可靠。

但 GitHub issue #11544 记录了一个 bug：配置写了，调试日志显示 `0 matchers found`，Claude 该读还是读。偶发，复现条件不明。

Mnilax 的建议是两层防御。第一层正常配 deny 规则：

```
{  
  "permissions": {  
    "deny": [  
      "Read(.env)",  
      "Read(.env.*)",  
      "Read(**/*secret*)",  
      "Bash(rm -rf:*)",  
      "Bash(sudo:*)"  
    ]  
  }  
}
```

第二层在操作系统层兜底：`chmod 600 .env`。即使 Claude 想读，操作系统也会拒绝。别全靠 deny 列表。

配完进 Claude Code 用 `/permissions` 验证，规则没出现就重启会话。

**8. SessionStart 钩子按分支加载上下文**

`hooks.SessionStart` 是 Claude Code 在某个目录打开时自动执行的钩子。

Mnilax 指出大部分人没用它，或者用错了：把所有项目规则都塞进 CLAUDE.md，结果 CLAUDE.md 长到 5000 token，每个会话都要先吞掉这一坨再开始干活。

他的做法是按分支加载上下文：

```
{  
  "hooks": {  
    "SessionStart": [{  
      "matcher": "startup",  
      "hooks": [{  
        "type": "command",  
        "command": "cat .claude/context-$(git branch --show-current).md 2>/dev/null || true"  
      }]  
    }]  
  }  
}
```

main 分支加载 `context-main.md`，`feat/auth` 分支加载 `context-feat-auth.md`。每个文件保持小，上下文占用降了 30% 左右（他实测的数字）。

我的 CLAUDE.md 现在也偏长，按分支拆是个聪明的解，回头试试。

**9. disableAllHooks：紧急开关**

这个开关 2026 年 3 月才出，知道的人不多。

Claude Code 偶尔会抽风（莫名其妙跑命令、会话启动卡死、文件被神秘修改），Mnilax 说 80% 的情况是某个钩子配错了。

逐个禁用排查很慢。`disableAllHooks: true` 一次性全关，重启，看问题是不是没了。没了再一个个开回来定位，还在那就是别处的 bug。

平时保持 `false`，记得这个开关在哪。

**10. 按项目覆盖 model**

`.claude/settings.json` 在项目根目录里设 `model`，会覆盖全局。

Mnilax 说了一个很多人不愿承认的事实：大部分人把全局模型设成 Opus，因为想留着干硬活，然后他们打开一个主要在编辑 Markdown 或跑 shell 脚本的项目，这种任务 Haiku 二十分之一的成本就能干完，他们却按 Opus 的价钱付钱。

修法：

```
// /docs 项目  
{ "model": "claude-haiku-4-5-20251001" }  
  
// /infra 项目  
{ "model": "claude-sonnet-4-6" }  
  
// /core-engine 项目  
{ "model": "claude-opus-4-7" }
```

打开项目，自动用对的模型。

**11. mcpServers 用 enabled 标志，别删**

跟 enabledPlugins 一个道理。MCP 服务器连接到外部工具，每个会把完整的工具 schema 加载进上下文，一个服务器 800 到 6000 token 不等。

Mnilax 给的画像很真实：装了测试，从来不断开。三个月下来连了 12 个，实际常用 3 个，剩下 9 个未用的服务器在每次会话启动时白扣 2.5 万到 4 万 token 的 schema。

```
{  
  "mcpServers": {  
    "github":   { "command": "...", "enabled": true },  
    "postgres": { "command": "...", "enabled": true },  
    "slack":    { "command": "...", "enabled": false },  
    "linear":   { "command": "...", "enabled": false }  
  }  
}
```

按会话需要打开。他自己大部分日子开 2 到 3 个，做规划的日子开到 6 个。

**12. cleanupPeriodDays 改成 180**

这个设置控制 Claude Code 保留会话记录、调试日志、中间会话数据的天数。默认 30 天。

Mnilax 提醒：Dreaming（记忆巩固）和过去聊天搜索都依赖这些记录。默认 30 天意味着 Dreaming 只能从一个月的工作里学东西。改成 180 天，信号量翻 6 倍，磁盘代价大概 200MB。

```
{ "cleanupPeriodDays": 180 }
```

Claude Code 这边 7 个设置讲完了。Mnilax 在 thread 末尾把它们拼成一个完整模板：

```
{  
  "model": "claude-sonnet-4-6",  
  "enabledPlugins": {  
    "formatter@acme-tools": true  
  },  
  "permissions": {  
    "deny": [  
      "Read(.env)",  
      "Read(.env.*)",  
      "Read(**/*secret*)",  
      "Bash(rm -rf:*)",  
      "Bash(sudo:*)"  
    ]  
  },  
  "hooks": {  
    "SessionStart": [{  
      "matcher": "startup",  
      "hooks": [{  
        "type": "command",  
        "command": "cat .claude/context-$(git branch --show-current).md 2>/dev/null || true"  
      }]  
    }]  
  },  
  "disableAllHooks": false,  
  "mcpServers": {  
    "github": { "command": "npx", "args": ["@modelcontextprotocol/server-github"], "enabled": true }  
  },  
  "cleanupPeriodDays": 180  
}
```

复制进 `~/.claude/settings.json`，路径和插件名换成你的，重启 Claude Code，跑一次 `/permissions` 和 `/hooks` 确认配置都加载了。

## API 这边，3 个键决定账单

API 这边的设置直接动账单，Mnilax 说每个调整都可能让账单浮动 30% 到 90%。

**13. cache\_control 断点放对位置**

这就是开头那个把账单从 340 砍到 87 的设置。

`cache_control` 把 prompt 的某段前缀标记为可缓存。后续相同前缀的请求按输入价格的 10% 收费，而不是全价。

大家都知道它存在，但大部分人放错了断点位置，结果只省了一部分没省全部。

断点要放在静态内容和动态内容的交界处。断点之前的所有内容会被缓存，断点之后的会重算。

错的写法：

```
messages = [  
    {"role": "system", "content": SYSTEM_PROMPT},  
    {"role": "user", "content": user_question,  
     "cache_control": {"type": "ephemeral"}}  
]
```

断点放在了用户消息后面。用户消息每次都变，等于什么都没缓存。

对的写法：

```
messages = [  
    {"role": "system", "content": SYSTEM_PROMPT,  
     "cache_control": {"type": "ephemeral"}},  
    {"role": "user", "content": user_question}  
]
```

断点放在稳定的 system prompt 之后，下次相同的 system prompt 直接命中缓存。

TTL 有两档，5 分钟（默认）和 1 小时。会话间不变的 system prompt 用 1 小时：

```
{"cache_control": {"type": "ephemeral", "ttl": "1h"}}
```

经济账：写缓存比基础输入贵 25%，读缓存便宜 90%。盈亏平衡点是 TTL 窗口内读 2 次以上。

自建 Agent 服务的人受这条冲击最大。之前看朋友 Agent 服务的账单结构，主要成本就是重复发送同一份长 system prompt。断点放对，立刻是数量级的差距。

**14. inference\_geo：合规没要求就别开**

`inference_geo` 把推理路由到特定地理区域，仅美国驻留、仅欧盟驻留这种。

但仅美国驻留会让 Opus 4.7 以上的模型贵 10%。这个加价不在标准价目表上，只会出现在账单里。

很多团队默认开这个参数是因为法务说过一句数据要留在美国。但很多时候这只是建议性的要求，并不是合同写死。

问清楚是合同要求还是建议性的。建议性的话直接关掉，每次 Opus 调用立刻省 10%。

**15. 工作区级速率限制**

路径：控制台 → 设置 → 工作区 → [你的工作区] → 按功能速率限制。

Mnilax 给的场景：账户级限制保护你不会破产，工作区级限制保护你的线上产品不被自己的批处理任务拖垮。

你发了一个新功能，bug 了，循环了，把账户全部 ITPM 配额吃光，结果面向客户的聊天开始返 429。

修法：每个使用面（交互聊天、批处理、内部工具、实验）拆一个工作区。每个工作区限额设成账户层级的 60% 到 70%，预留 30% 给需要突发的工作区。

他还专门点出一个隐藏设置：每个工作区内部还有一个按功能维度的上限，只在你点进具体功能卡片时才能看到，工作区总览页不显示，默认无限。如果一个工作区里有三个功能，其中一个会饿死另外两个，工作区级限制抓不到。给任何跑批处理的功能单独设上限。

## 18 项清单和审计脚本

Mnilax 给了一份 20 分钟可以全部跑完的清单：

**Claude.ai**

* • 记忆按项目作用域开，排除列表填上
* • 扩展思考默认 Light
* • 至少建一个工作流自定义样式
* • 每个活跃项目的项目说明都填上
* • 过去聊天搜索打开
* • 网页搜索引用换成脚注
* • Cowork 信任文件夹清一遍
* • 无痕模式快捷键记住

**Claude Code**

* • enabledPlugins 只留真用的
* • permissions.deny 配上 + chmod 600 兜底
* • SessionStart 钩子按分支加载
* • disableAllHooks 知道在哪
* • 项目级 model 覆盖配置好
* • mcpServers 用 enabled 标志
* • cleanupPeriodDays 改成 180

**API / Console**

* • cache\_control 断点放在稳定 system prompt 之后，1 小时 TTL
* • inference\_geo 没合规要求就别开
* • 工作区限额 + 按功能上限都设上

他还顺手给了一个每周审计脚本，扔进 `~/bin/claude-audit.sh`：

```
#!/usr/bin/env bash  
SETTINGS="$HOME/.claude/settings.json"  
  
echo "=== 启用的插件数 ==="  
jq '.enabledPlugins // {} | to_entries | map(select(.value==true)) | length' "$SETTINGS" 2>/dev/null  
echo "目标:3-5 个常驻。"  
  
echo "=== 启用的 MCP 服务器数 ==="  
jq '.mcpServers // {} | to_entries | map(select(.value.enabled==true)) | length' "$SETTINGS" 2>/dev/null  
echo "目标:3 个常驻。"  
  
echo "=== deny 规则数 ==="  
jq '.permissions.deny // [] | length' "$SETTINGS" 2>/dev/null  
echo "目标:至少 5 条。.env / sudo / rm -rf 必备。"  
  
echo "=== SessionStart 钩子数 ==="  
jq '.hooks.SessionStart // [] | length' "$SETTINGS" 2>/dev/null  
  
echo "=== cleanupPeriodDays ==="  
jq '.cleanupPeriodDays // 30' "$SETTINGS" 2>/dev/null  
echo "目标:180。"
```

`chmod +x ~/bin/claude-audit.sh`，每周跑一次直到每项达标。

## 写在最后

这份清单不是一份 Claude 使用技巧，而是一份默认值审计报告。

Anthropic 不是有意藏这些设置，他们更新太快、文档跟不上。默认值是给平均用户准备的，你有自己的工作流、项目模式、成本敏感度，默认值跑久了就开始拖你。

我看完这份 thread 立刻做了三件事：把全局 enabledPlugins 砍了一半、把 cleanupPeriodDays 改成 180、给两个常用项目加了项目级 model 覆盖。总共花了不到 10 分钟，下个月的账单和会话上下文我应该能看出差别。

建议你今晚抽 20 分钟，对着这份 18 项清单一项项过一遍。

Mnilax 原文在 X 上，作者是 @Mnilax。这篇是读后感 + 转译，加了一些自己的延伸点评，原文链接我放在评论区，想看原版的可以去看。

模型没变。变的是你的配置有没有跟上。