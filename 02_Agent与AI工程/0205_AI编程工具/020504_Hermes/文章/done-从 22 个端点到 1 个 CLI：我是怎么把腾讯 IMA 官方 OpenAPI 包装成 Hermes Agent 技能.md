> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020504_Hermes/020504_核心知识点/Hermes协作与记忆治理边界|Hermes协作与记忆治理边界]]
---
title: 从 22 个端点到 1 个 CLI：我是怎么把腾讯 IMA 官方 OpenAPI 包装成 Hermes Agent 技能的
author: 常晓辉
date: 常晓辉常晓辉
url: https://mp.weixin.qq.com/s?__biz=MzkzNjc1NzEyOQ==&mid=2247485788&idx=1&sn=d6bcd841594497b74ec8a1b569c70dec&chksm=c3bf60231b0b2770213dad93800dc489001a80d803f6e86b43b7fd4097873ae03c334934e75d&mpshare=1&scene=24&srcid=0605nLqqR2fYVFD4kLqNqPiK&sharer_shareinfo=d5f8b33a02df3263418c64fd29e6e817&sharer_shareinfo_first=d5f8b33a02df3263418c64fd29e6e817#rd
---

> 第一次给开源社区做贡献，我没改一行腾讯的源码，只是把官方给的工具"翻译"成 Agent 能听懂的话。但这件事让我重新理解了"基于原版改造"这 7 个字的重量。

"改造" 不等于 "fork"。标注清楚"我做了什么"比"我做得多好"更难 —— 商业产品的 OpenAPI 往往比文档多出 30% 灰色能力，你得自己找。

---

## >\_ 1. 起点：商店没我想要的

我不是第一个想做这个的人。Hermes Agent 的技能商店里早就有 `tencent-ima-skill` v1.0.1，2.7k 次下载，clawhub 标记 `clean`。我装上跑了 5 分钟，关掉，理由只有一条 —— 它是"控制桌面应用"的路线，要的是 cookie + bkn。我用的是 OpenAPI，要的是 clientId + apiKey，**路线不兼容**。

商店里"有"不等于"我能用" —— 工具的路线错了，比"没有"还让人烦。选错实现路线，比选错技术栈代价更高。

更现实地讲：我不想在 cookie 体系里赌账号安全。腾讯对 cookie 滥用是有风控的，我一个 2 年的老账号不想因为跑个脚本被封。OpenAPI 的 clientId + apiKey 是腾讯为机器访问设计的正经凭证，用它我心里踏实。

---

## >\_ 2. 动手：基于腾讯官方 v1.1.7 改造

真正的活儿从这里开始。我从腾讯 ima 官方下载页拿到 `ima-skills-1.1.7.zip`（v1.1.7，腾讯 IMA 团队自己发的 OpenAPI 客户端），跟商店版做了**对比清单**，确认它的覆盖面更大且用官方凭证。基于这个版本，我做了 4 项改造：

1. **OpenClaw 风格 frontmatter → Hermes 风格**（让 hermes 知道这个 skill 存在、怎么触发、需要什么凭证）
2. **写 `bin/ima` POSIX 桥接**（22 个子命令覆盖全部 16 个文档化端点 + 1 个便利封装 + 5 个后文会说到的探针发现）
3. **凭证路径对齐 Hermes `.env` 约定**（`IMA_OPENAPI_CLIENTID` / `IMA_OPENAPI_APIKEY`，让 hermes 自动加载，不要求用户改 `~/.bashrc`）
4. **探针发现 5 个未文档化端点**（见下一节）

改造的底线：**不**发明新 API 端点、**不**删改原版字段名 —— 这是我能给原作者最大的尊重，也是给下游用户最大的保护。如果腾讯以后改了字段名，我能跟着改；如果我发明了字段，腾讯改了之后我就废了。

写 `bin/ima` 的 6 小时比写探针的 3 小时还多 —— CLI 桥接是**给未来人**写的，不是给机器。人类读、人类调、人类出问题时能直接 `cat` 出来看 JSON。这个直觉在我后来发现"探针发现的端点字段名要靠实际响应反推"时救了我 —— 因为所有 `bin/ima` 失败时打印的 JSON 都带文件名 + 行号。

---

## >\_ 3. 最有价值的发现：5 个未文档化端点

做完 16 个文档化端点我没停手，做了一件作者没明说但 OpenAPI 没禁止的事：**穷举探针**。我对 `ima.qq.com/openapi/...` 的 50+ 候选路径发 POST，看哪个返回非 `RAW=0`。结果是 5 个端点真存在但 SKILL.md 没写：

* `openapi/wiki/v1/create_folder` —— 创建文件夹
* `openapi/wiki/v1/create_knowledge_base` —— 创建新知识库
* `openapi/wiki/v1/move_knowledge` —— 在 KB 间/内移动条目
* `openapi/note/v1/add_notebook` —— 创建笔记本
* `openapi/note/v1/rename_notebook` —— 重命名笔记本

**但更重要的发现是反面的**：所有 delete 变种、所有 move folder、所有 delete kb entry —— 腾讯路由层**硬拒绝**（`RAW=0`）。OpenAPI 整体**不开放**"删/改/移"能力。这是产品决定，不是技术疏漏。

探针告诉你两件事：能力边界在哪、不能做什么 —— 后者比前者更有价值。
探针是**对原作者**的致敬：你没写的功能，不是我没找到，是**你产品决定不给我**。

---

## >\_ 4. 三个意外发布关

我以为代码写完就结束了。错了。

clawhub 审核第一天给我的反馈是 3 条 "unsupported files"：

1. `bin/ima` —— 它是可执行文件，没后缀 → **改名 `bin/ima.sh`**，文档 23 处引用全改
2. `LICENSE` —— 文档说认可 text-based 文件，**审核不认** → 内容挪进 README 的 License 段
3. `.gitignore` —— 同上，**审核不认** → 移出发布包（Git 仓里仍可有，但不进发布版）

整个过程让我学到的最大教训：**先看商店审核规则再写代码**。不是反过来。文档第一条 `https://docs.openclaw.ai/clawhub/skill-format`[1] 我应该**第一天**看完，但我跳过了，结果第二天回来返工。

商店审核反馈是礼物，不是打击 —— 它把你变成"在 X 商店能用的人"。一个「不兼容 → 改名 → 重新提交」的小循环，比「自由发挥 → 提交 → 被拒 → 大改」节省 80% 时间。

---

## >\_ 5. 三类声明的取舍

我花了比写代码更多的时间想这 3 类声明放哪、写什么：

1. **"基于腾讯官方 v1.1.7 改造"** —— 写进 SKILL.md frontmatter 的 `upstream` 字段、`README.md` 顶部、`_meta.json` 的 `upstream` 字段。三处冗余，但每处目标读者不同（Agent / 人 / 商店爬虫）
2. **"删除/移动/重命名相关指令正在开发中"** —— 探针已证无端点。但用户不知道我做没做 —— **把"没找到"包装成"正在做"是欺骗**。我写"在开发中"是因为 hermes 路线可以走 cookie 自动化曲线，**未来可做**，不是许诺
3. **完整改造清单** —— 不是夸，是透明。**不**藏短板

"在开发中" 三个字不是骗人的避难所 —— 你得知道它**未来能做**，才敢写。
写声明比写代码难：代码错了可以改，声明错了**信任**坏了。

我做这件事之前想了一晚上：要不要写"完全替代官方"？算了，**官方是源头**，我是翻译。翻译说自己"超越原作者"，读者信了会去问原作者"你为什么不做"，这是给原作者添堵。

---

## >\_ 6. 邀请使用 + 反馈

这个 skill 不只属于我 —— 它属于所有"在 ima 里攒了一堆知识库、又想让 AI agent 顺手调一下"的人。

**如果你是 Ima 重度用户**：装上之后试着让小飞（或者你用的 agent）"在 ima 里搜下 XX"，感受一下 22 个子命令把整个 ima 装进 agent 嘴里的体验。如果你发现新的未文档化端点，欢迎 PR。

**如果你是 Agent 开发者**：这是我做 Hermes 适配时留的最深的一个 hook —— `bin/ima` 22 个子命令**全部纯 JSON 输出**，可以直接喂给任何 LLM 当 tool use。我之前考虑过加 `ima ask` 这样的高层封装（搜索 + 拼答案），最后决定不写 —— 调度是 agent 的事，不是 skill 的事。

**如果你是腾讯 IMA 团队**：如果你看到这篇 —— 我对你们的 OpenAPI 没有任何批评，文档化 + 探针发现的 21 个端点已经够我做非常多的事情。**删除/移动/重命名**如果能在 OpenAPI 暴露，是给整个生态的礼物。**欢迎收我入伙**。

反馈方式（任选）：

* **GitHub Issues**：[2]
* **clawhub 评论**：[3]
* **微信**：通过你已有的 iLink bot 私聊（这是另一篇的故事）

一个 skill 的价值，不在于代码写得多漂亮，在于**有人用 + 有人反馈**。给腾讯 IMA 团队留话 —— 不卑不亢，**是建议不是抱怨**。

---

## >\_ 7. 最终交付清单

这个 skill 现在叫 `tencent-ima-hermes`，发布在 clawhub + GitHub 双仓：

* 22 个 CLI 子命令（笔记 6 + 知识库 10 + 便利封装 1 + 未文档化 5）
* 一行安装：`hermes skills install tencent-ima-hermes`
* 凭证从 `~/.hermes/.env` 读（`IMA_OPENAPI_CLIENTID` / `IMA_OPENAPI_APIKEY`）
* 完整文档 16 个文件 220KB
* 探针发现的 5 个端点**经用户确认**后我会推到 v1.1.7-hermes.2

22 是个具体数字 —— 比"很多"、"齐全"更让人想用。

---

## >\_ 8. 我的反思

第一次给开源社区做贡献，**学到的不是技术，是三件事**：

1. **审核规则优先于实现**：下次开 skill，**第一件事**是看目标商店的 `skill-format` 文档，不是写代码
2. **声明是产品的一部分**：基于原版改造的，"我做了什么"和"我**没**做什么"都得写
3. **不要给下游留坑**：字段名不能改、API 路径不能改 —— 让上游 PR 有路径可以走

期待社区反馈。如果你有 Tencent IMA 官方之外的 API 经验，欢迎指正。

第一次贡献的价值，不在于做得多好，而在于**做完了 + 敢说"我做了什么"**。
改造者的诚实，是开源的信用货币。

---

## >\_ 参考链接

* [1] https://docs.openclaw.ai/clawhub/skill-format[1]
* [2] https://github.com/wat2012/tencent-ima-skill/issues[2]
* [3] https://clawhub.ai/wat2012/tencent-ima-hermes[3]
* [4] https://github.com/wat2012/tencent-ima-skill[4]

### 引用链接

[1]*https://docs.openclaw.ai/clawhub/skill-format*

[2]*https://github.com/wat2012/tencent-ima-skill/issues*

[3]*https://clawhub.ai/wat2012/tencent-ima-hermes*

[4]*https://github.com/wat2012/tencent-ima-skill*