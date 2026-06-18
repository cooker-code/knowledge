> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020203_Skill/020203_核心知识点/Skill能力封装与治理边界|Skill能力封装与治理边界]]
---
title: 法律人学Claude｜第九期：给自己定制一个审合同Skill——Skill详解
author: 法知能LawAI
date:
url: https://mp.weixin.qq.com/s?__biz=MzI0NzUwMDUzMw==&mid=2247484051&idx=1&sn=4d90b86f04b6efa4feb9c1eca7c82d21&chksm=e88b390797ddb76e93b827576e62df631766511b67e5ab7cf0a0900794ccdbfcbe3660381496&mpshare=1&scene=24&srcid=0409Pi6tUUZK5Kj1r7zJw8Wh&sharer_shareinfo=da8f2091d89eddf71cb5bec5ab5c6447&sharer_shareinfo_first=da8f2091d89eddf71cb5bec5ab5c6447#rd
---

## 客户发来一份采购合同，八十多页，要求明天上午给意见。

你打开 Claude，把合同扔进去，输入："帮我审一下这份合同。"

Claude 开始输出。洋洋洒洒一大段，有条款摘录，有风险提示，读起来很像回事。但你仔细一看，发现几个问题：违约责任条款漏掉了，争议解决条款没有提到仲裁地点，格式条款部分根本没检查……

不是 Claude 不行，是它不知道你要它按照什么标准审。

每次你都要重新说一遍："帮我重点看违约条款、格式条款、争议解决……"下次再来，还是要说一遍。

这期我们解决这个问题。用 `/skill-creator`，把你的审核标准固化进 Claude，以后每次用 `/contract-review` 一触发，它就知道要怎么干了。

## 一、先搞清楚：skill-creator 是什么

`skill-creator` 本身也是一个 Skill。

它的作用是帮你创建其他 Skill。你用自然语言描述想要的功能，它生成对应的 Skill 文件，自动保存到正确位置。

要理解它怎么工作，先得搞清楚 Skill 文件是什么。

所有 Skill 都存在 `~/.claude/skills/` 这个文件夹里。在[法律人学Claude｜第五期：让Claude用上次抛App——Skills初解](https://mp.weixin.qq.com/s?__biz=MzI0NzUwMDUzMw==&mid=2247483994&idx=1&sn=91cefe0a8249c76493c7704b868edd49&scene=21#wechat_redirect)中讲过。每个 Skill 对应一个子文件夹，文件夹里必定有一个 `SKILL.md`，有些复杂 Skill 还会带其他辅助文件。比如你创建一个合同审核 Skill，结构大概是这样：

```
~/.claude/skills/
└── contract-review/
    └── SKILL.md       ← 这个 Skill 的全部工作指令
```

`SKILL.md` 是一份文字文档，不是代码。它告诉 Claude 遇到这个命令该做什么、按什么顺序做、输出什么格式。

每次你输入 `/contract-review`，Claude 先读这份文件，再读你的合同，再开始审查。

所以直接问"帮我审这份合同"和用 Skill 触发，区别就在这里：前者 Claude 靠临场发挥，后者它先读了一份固定标准再动手。

## 二、用 `/skill-creator` 创建合同审核助手

### 第一步：触发创建流程

在 Claude 对话框输入 `/skill-creator`，Claude 进入引导模式，会先问你：这个 Skill 要做什么？

### 第二步：用一段话说清楚你要什么

这步最重要。你说得越具体，生成的 Skill 越好用。

你可以这样告诉 Claude：

> 我需要一个合同审核助手。读入一份合同后，按照以下顺序逐项检查：
> 第一，格式条款和不平等条款——重点看是否存在加重对方责任、排除对方主要权利的表述；
> 第二，违约责任条款——检查违约金计算方式是否合理，是否有单边免责内容；
> 第三，争议解决条款——识别仲裁还是诉讼，标注仲裁机构和仲裁地点；
> 第四，合同解除条款——看双方的解除权是否对等；
> 第五，其他重大风险条款。
> 输出格式：每类条款用一级标题分开，每个风险点用【高风险】【中风险】【低风险】三档标注，并给出一句话修改建议。

不需要专业词汇，就是这样用大白话描述。Claude 会把你的描述整理成结构化的 Skill 文件。

然后他就做好了计划，自己做一个合同审核的skill。

### 第三步：确认并保存

Claude 生成 Skill 文件后会展示给你看，问你是否需要调整。确认后，它把文件写入 `~/.claude/skills/` 文件夹，注册为可用命令。

### 第四步：自己测试

接着`skill-creator`会自己创建测试用例，测试自己生成的/contract-review到底符合不符合需要。

等一会儿之后。他就会把测试用例没有使用skill和使用skill的做对比发给你看。

下面请欣赏一下Claude skill creator的杰作：

测试1：软件开发合同

测试2：法律顾问协议

测试3：商业用房租赁合同

下面还可以提交你的意见，反馈给Claude，让它再进行改进。

温馨提示：使用skill-creator前确保自己时段内的token额度还足够，如果你没有像我一样有多充值的金额，就有可能会中断。

SKILL.md是什么结构？

开头的一部分是**YAML Frontmatter**，就是用两个“---”包裹住的那一部分，这是skill的元数据。

Claude Code通过读取这部分元数据来识别并加载新技能。它告诉Claude Code“我是一个技能，我叫什么名字，我能做什么”。

**注册名称 (`name`)**：定义技能的唯一名称，用于直接调用（比如这里的 `/contract-review`）。

**触发机制 (`description`)**：**最重要的部分**。Agent 会根据 description 中的文本判断在何时自动激活该技能。一个好的描述包含“做什么”和“什么情况下用”，这直接决定了技能的可用性。

然后主要内容这一块定义了这个skill的角色定位、审核流程、输出格式等等内容，这只是Claude理解合同审核的一部分内容，你完全可以加入自己的经验，给你自己制造一份通用的合同审核skill。

大家如果对这个skill感兴趣，可以后台回复我“contract-review”领取，不保证这个有用哈，纯作为研究可以看看。

## 三、创建之后怎么优化

第一次用完，大概率会有一两个地方不满意。输出格式不对，某类条款漏了，或者风险等级划分和你的习惯对不上。Skill 就是拿来改的，生成只是第一步。

**直接改文件是最快的**。打开 `~/.claude/skills/contract-review/SKILL.md`，找到对应的步骤，用自然语言补上你的要求。发现没有检查保密条款，就加一行："第六，保密条款——识别保密范围、保密期限和违反保密的违约后果。"保存，下次触发立即生效。

**也可以再跑一遍 `/skill-creator`**。输入命令，告诉 Claude："我想在已有的合同审核 Skill 里增加以下内容……"它会读取现有文件，在原有基础上修改，不是推倒重来。

**最值得做的是把律所SOP装进去**。如果你所在的律所有一套合同审核标准SOP，把这份清单的内容描述给 Claude，让它更新 Skill。做完这一步，这个 Skill 就不再是通用工具了，是你自己的审核标准。

## 四、更复杂的玩法

合同审核 Skill 还可以继续接工具、放脚本，放参考文献。

就像minimax-docx这个skill，包含了很复杂的内容。

在接入北大法宝 MCP 之后（参考[法律人学Claude｜第七期：给Claude装上"外挂"——CLI与MCP工具使用指南](https://mp.weixin.qq.com/s?__biz=MzI0NzUwMDUzMw==&mid=2247484016&idx=1&sn=b9c133c92439c778f48d79bb03a1fbb1&scene=21#wechat_redirect)），Claude 审查风险条款时还可以直接检索对应法条和相关判例，风险提示有据可查，不只是经验判断。

审核合同之后再调用 minimax-docx skill，审查完成后直接生成一份格式规范的 Word 审核意见书，能直接发给客户。输入一条命令，拿到一份 Word 文件。

优化完成的skill，就是只需要你输入一条命令，中间的步骤全部自动跑完，最后拿到文件。

## 常见问题

**Q：输入 `/contract-review` 没有反应？**

A：检查 Skill 文件是否保存在 `~/.claude/skills/` 文件夹下，且文件夹名称和命令名一致。创建完成后如果没反应，重启 Claude Code 试试。

****Q：**每次输出格式不一样，时好时坏？**

`A：SKILL.md` 里对输出格式的描述不够具体。回去把你期望的格式写清楚：几级标题、标签格式、每个风险点包含哪些内容。越具体，越稳定。

****Q：**想删掉这个 Skill 怎么操作？**

A：直接删除 `~/.claude/skills/contract-review/` 整个文件夹，下次重启后命令自动失效。

****Q：**可以同时建多个 Skill 吗？**

A：可以。不同类型合同各建一个：采购合同、租赁合同、合伙协议，各有各的审核逻辑，互不干扰。

****Q：**这个 Skill 可以分享给同事用吗？**

A：可以。把 `contract-review` 文件夹发给同事，对方放到自己的 `~/.claude/skills/` 里，重启后就能用，触发同一个命令，执行同样的审核逻辑。

## 往期回顾

[法律人的AI agent教程 合集](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI0NzUwMDUzMw==&action=getalbum&album_id=4453870239231082497#wechat_redirect)

[法律人学Claude｜第一期：桌面版已经很好用了，为什么我还是力推 VSCode 插件版？](https://mp.weixin.qq.com/s?__biz=MzI0NzUwMDUzMw==&mid=2247483900&idx=1&sn=8ed3394ed1fde9b78e5f5b285dfd34a5&scene=21#wechat_redirect)

[法律人学Claude｜第二期：半小时装好 VSCode + Claude Code](https://mp.weixin.qq.com/s?__biz=MzI0NzUwMDUzMw==&mid=2247483936&idx=1&sn=5a4ce57b245c80cee51253db8e605518&scene=21#wechat_redirect)

[法律人学Claude｜第三期：让Claude更高效读懂你的文件](https://mp.weixin.qq.com/s?__biz=MzI0NzUwMDUzMw==&mid=2247483964&idx=1&sn=cffd5fe7bd3d23e91a312f587ed5c906&scene=21#wechat_redirect)

[法律人学Claude｜第四期：你的项目助理—CLAUDE.md使用指南](https://mp.weixin.qq.com/s?__biz=MzI0NzUwMDUzMw==&mid=2247483973&idx=1&sn=a26555517c52d5f6cc8afa97ba0dd89e&scene=21#wechat_redirect)

[法律人学Claude｜第五期：让Claude用上次抛App——Skills初解](https://mp.weixin.qq.com/s?__biz=MzI0NzUwMDUzMw==&mid=2247483994&idx=1&sn=91cefe0a8249c76493c7704b868edd49&scene=21#wechat_redirect)

[法律人学Claude｜第六期：不做8秒记忆的金鱼——优化记忆Memory](https://mp.weixin.qq.com/s?__biz=MzI0NzUwMDUzMw==&mid=2247484000&idx=1&sn=291d442fc2282c767f8b8cc2f773fcf7&scene=21#wechat_redirect)

[法律人学Claude｜第七期：给Claude装上"外挂"——CLI与MCP工具使用指南](https://mp.weixin.qq.com/s?__biz=MzI0NzUwMDUzMw==&mid=2247484016&idx=1&sn=b9c133c92439c778f48d79bb03a1fbb1&scene=21#wechat_redirect)

[法律人学Claude｜第八期：法律人的文档革命——你必须学会Markdown](https://mp.weixin.qq.com/s?__biz=MzI0NzUwMDUzMw==&mid=2247484027&idx=1&sn=5caa8abd763821be91eebf4257ebf6e2&scene=21#wechat_redirect)

对了，我建了一个交流群，有想进群的伙伴可以加我。