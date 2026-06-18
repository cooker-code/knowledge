> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020202_MCP/020202_核心知识点/MCP与CLI代码执行边界|MCP与CLI代码执行边界]]、[[02_Agent与AI工程/0202_工具调用/020202_MCP/020202_核心知识点/MCP生产接入与治理边界|MCP生产接入与治理边界]]
---
title: 从 MCP 到 CLI：Agent 工具调用的范式转移
author: 反时钟效率笔记
date:
url: https://mp.weixin.qq.com/s?__biz=MzkzNTk3NTYyNA==&mid=2247484343&idx=1&sn=3dcc0eb49e60af1d7b8621f84c9047ee&chksm=c3444e930808b2c7be61efe5519bd29d4beb72de3de11281aa6db258b665c24f6b6d9ce8d04b&xtrack=1&req_id=1774958009696547&scene=90&subscene=93&sessionid=1774958028&flutter_pos=0&clicktime=1774958031779&enterid=1774958031779&finder_biz_enter_id=4&ranksessionid=1774958009&key=daf9bdc5abc4e8d0ee96b139102015b5c2d2e7c075c07ba74bd8d5d7fcdeca65f77050d0ee5669c6843c0ac83b7d4fb0b91c0ae1df25a541d112450f2d0d374649fde512a137dd42675ea82360020eaf5805de6d45e8a339583787a3c319e98dc5069e4081b065361f9659264779db4cd8c8e36e9bbee9d3a3cd23c6bb8ea2b9&ascene=0&uin=MjAwNTI2NjEyMw%3D%3D&devicetype=OHOS-22&version=f3800f40&abtest_cookie=AAACAA%3D%3D&lang=zh_CN&countrycode=CN&exportkey=n_ChQIAhIQLwwtg9db1AmIWSjBK0tAdhLfAQIE97dBBAEAAAAAAHFWIeWqLuYAAAAOpnltbLcz9gKNyK89dVj0lw1SM72ee5uiqAj6ctsTiFWeNPjdtVaiDqxXW%2FCgsn2GlVhoPPawrp8dKdk1AWNZNDPkf7vugtlUs8z1JfbHRGeeJor2Nurm4g0Dr3P3NY1ZYM2zJixn3NkazBXQ%2FTu2oXUO%2BV76S%2BPPRj9GcQH77i9vo6PJ1N7LG%2FMqB7JOXc10VBtjft0SZmR4wKdQrKzEf0J51oHmoTvlYpxXBCXJEsnbaem0vPrGZKZDjXD3gdIu4mrFlFjujWU%3D&pass_ticket=MPu4bGWIfEvFdqRlPGoMMeiiHc%2B9KdYvAys6W%2FCDl9nnaEOM65XwiObxLqsGrH%2Fe&wx_header=3
---

最近，CLI（命令行）项目密集爆发。

飞书、钉钉、企业微信、Google Workspace 接连开源了自己的 CLI 工具。OpenCLI和港大实验室的 CLI-Anything 项目在 GitHub 上热度飙升。社区里甚至开始讨论"CLI 才是 Agent 的终局"。

这背后有一个正在成型的共识：**CLI 是 Agent 与世界交互的"母语"**，而去年被寄予厚望的 MCP，正在被边缘化。

---

## 01 CLI 在 Agent 时代到底是什么

传统意义上的 CLI，就是在终端里敲命令——`git status`、`docker ps`、`kubectl get pods`。

在 AI Agent 的语境里，CLI 指的是：**Agent 通过 shell 命令直接调用外部能力**，而不是走函数调用的 schema 或 JSON-RPC。

具体来说：

* Agent 想干活 → 输出一条命令（如 `lark-cli calendar +agenda`）
* 系统在沙箱里执行 → 把 stdout/stderr 返回给 Agent
* Agent 继续推理 → 输出下一条命令

这相当于工具调用（Tool Calling），但更原始，也更强大。

传统的 Tool Calling 必须预先定义好函数名、参数 schema，上下文里塞一大堆描述。而 CLI 不需要预定义任何 schema——Agent 自己"写代码"生成命令，只要系统有对应命令，就可以直接用。

LLM 的训练数据里有几十年 Unix 哲学的积累，天然就会读/写命令行。CLI 对 Agent 来说几乎是零学习成本。

---

## 02 为什么 CLI 突然这么火

2024 年底，Anthropic 推出 MCP（Model Context Protocol）时，所有人都觉得这是"AI 工具的 USB 接口"。OpenAI、Google、Microsoft 全部跟进。

但 MCP 的痛点暴露得很彻底：

* 必须启动 MCP Server（JSON-RPC）
* 所有工具定义一次性预加载到上下文 → Token 成本线性爆炸
* 初始化不稳定、调试困难、权限控制复杂

问题出在 MCP 的 schema 注入机制。它要求把所有工具定义一次性塞进上下文。你只是想用一个功能，但模型得先读完所有工具的说明书。

而 CLI 完全不同：Agent 只需要跑一下 `--help`，按需读取某个子命令的参数说明。用多少读多少，不用把所有命令的文档一股脑塞进上下文。

这就是为什么一个 Skill.md 只要 800~2000 token，而 MCP 的工具清单动辄占掉几万 token。

**CLI + Skill 的组合几乎完美解决了 MCP 的问题：**

* **零协议**：不需要额外 Server
* **按需执行**：Agent 想用什么就生成什么命令
* **上下文极省**：只塞一个 800~2000 token 的 Skill.md 就够了

大厂直接跟进：飞书 Lark CLI、钉钉 CLI、企业微信 CLI、Google Workspace CLI 全开源，让 Agent 能"原生"调用它们。

现在社区的主流观点是：**MCP 会下沉成企业集成中间件，CLI + Skill 才是 Agent 与世界交互的终局**。

---

## 03 CLI、MCP、Skill 的本质区别

三者不是竞争关系，而是层级不同。

* **MCP = 连接层（Protocol）**

像 USB 接口 + 驱动。优点是标准化、支持远程、OAuth 权限好；缺点是上下文重、要写 Server、初始化慢。

定位：解决"让 Agent 够得着"外部系统的问题。

* **Skill = 知识层（SOP / 操作手册）**

一个 Markdown 文件（SKILL.md），里面写：这个任务的场景、最佳实践、常见坑、具体用哪些 CLI 命令 + 参数、输出格式要求。

优点是按需加载，上下文极省，还注入领域知识。

定位：解决"教 Agent 怎么干得好"的问题。

* **CLI = 执行层（Execution）**

真正的命令运行在 shell 里。优点是最轻量、LLM 最懂、无额外抽象。

定位：解决"真正干活"的问题。

**最强组合 = CLI + Skill**：

* Skill 负责"教"（知识 + 流程）
* CLI 负责"干"（实际执行）
* 完全绕过 MCP 的中间层

| 维度 | MCP | Skill | CLI |
| --- | --- | --- | --- |
| 核心作用 | 标准化连接协议 | 领域知识 + SOP 封装 | 实际命令执行 |
| 上下文成本 | 高（预加载所有工具） | 极低（按需加载） | 几乎为 0 |
| 开发难度 | 高（要写 Server） | 低（写 Markdown + 脚本） | 极低（写命令/脚本） |
| 灵活性 | 中等 | 高 | 最高（操作系统级别） |
| 适用场景 | 复杂远程 API、权限严格场景 | 内部流程、领域专家知识 | 几乎所有本地/云工具 |
| 当前趋势 | 正在被边缘化 | 越来越火 | 最火（大厂疯狂开源） |

一句话：**MCP 解决了"能不能连"的问题，但没解决"怎么用得好"。Skill 解决了"怎么用得好"。CLI 提供了最简单、最可靠的"连"和"用"底层。**

## 04 哪些东西能做出 CLI

CLI 的核心优势是**几乎任何东西都能变成 CLI**：

| 类型 | 例子 | 如何实现 | 适合场景 |
| --- | --- | --- | --- |
| 已有成熟 CLI | `git` 、`gh`、`docker`、`kubectl` | 直接给 Agent 权限 | 开发、运维、云原生 |
| 自定义脚本 | Python/Shell 脚本包装业务逻辑 | 写个带 `--help` 和 `--json` 的 CLI | 内部工具 |
| 自动生成 CLI（本地 GUI） | CLI-Anything | 喂源码 → 自动生成完整 CLI 接口 | GIMP、Blender、LibreOffice |
| 自动生成 CLI（Web 服务） | OpenCLI | 注册网站/App → 浏览器自动化封装 | Twitter、Reddit、Notion |
| 大厂官方 CLI | 飞书 Lark CLI、钉钉 CLI、企微 CLI | 官方开源，直接支持 Agent | 企业协作、办公自动化 |
| 框架内置 | OpenClaw、Gemini CLI、Claude Code | 原生支持 CLI 执行 | 通用 Agent 框架 |

**两个自动生成工具值得关注：**

* **CLI-Anything** 专注本地桌面 GUI 软件。给它任意软件源码，它跑一个 7 阶段的 pipeline（分析 → 设计命令 → 实现 → 测试 → 生成 SKILL.md），输出一个可 pip install 的 Python CLI 包。GIMP、Blender、LibreOffice 这些原本只能 GUI 操作的软件，一条命令就能被 Agent 直接控制。
* **OpenCLI** 专注 Web 服务和 Electron App。它更像一个"通用 Hub + Runtime"，支持快速转换 80+ 网站，通过浏览器自动化或 API 包装变成命令行。Agent 可以跑 `opencli list` 自动发现所有可用工具。

两者定位互补：CLI-Anything 解决"本地专业软件 CLI 化"，OpenCLI 解决"Web 服务 CLI 化"。一起把 GUI 操作和 Web 操作都变成了 Agent 原生工具。

---

## CLI 不是新工具

CLI 不是"又一个工具"，它是 Agent 时代把 50 年前的 Unix 哲学重新激活。

Agent 终于能像人类运维/开发者一样"敲命令干活"。MCP 曾被寄予厚望，但 CLI + Skill 用更少的成本、更好的可靠性，把它干下去了。

这波"CLI 复兴"才是 Agent 真正的底层基础设施变革。

—The End—

如果你看到这里，**或许这篇内容对你来说是有价值的。**

点亮 **「赞」 和 「推荐」**，让更多人也看到它❤️
你的认可是我继续创作下去的动力~

**▍往期文章推荐**

* [放弃Vibe Coding：我用Superpowers+gstack沉淀出一套大幅减少返工的skill组合工作流（附案例）](https://mp.weixin.qq.com/s?__biz=MzkzNTk3NTYyNA==&mid=2247484332&idx=1&sn=f292a13108026ebe4961539707e0462d&scene=21#wechat_redirect)
* [Claude Code 为什么能同时干多件事？子代理与上下文压缩的底层逻辑](https://mp.weixin.qq.com/s?__biz=MzkzNTk3NTYyNA==&mid=2247484305&idx=1&sn=78388978d911467eda51c1dd0669fe75&scene=21#wechat_redirect)
* [聊聊最近很火的 Harness Engineering：结合 OpenClaw 和 OpenCode 的详细拆解](https://mp.weixin.qq.com/s?__biz=MzkzNTk3NTYyNA==&mid=2247484278&idx=1&sn=5e3ea21409e756846ee53c964307e751&scene=21#wechat_redirect)

* [让多个 Claude 并行工作：多智能体团队协作机制详解及启动方法](https://mp.weixin.qq.com/s?__biz=MzkzNTk3NTYyNA==&mid=2247484306&idx=1&sn=06969ca22660f16c863269365cfd23db&scene=21#wechat_redirect)
* [告别重复劳动：如何用Skill Creator把工作流变成可复用技能 （账号分析skill分享）](https://mp.weixin.qq.com/s?__biz=MzkzNTk3NTYyNA==&mid=2247484241&idx=1&sn=ca6ebf13981cefc0b5bb29b7d5b21254&scene=21#wechat_redirect)

**关于作者 · Aren**

06intj | 211计算机本科在读

AI实战 | 产品拆解 | 干货分享 | 效率提升

欢迎添加我的个人微信，一起交流学习👇