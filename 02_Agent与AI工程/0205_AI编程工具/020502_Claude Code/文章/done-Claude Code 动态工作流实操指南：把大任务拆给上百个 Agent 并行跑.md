> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Claude Code 动态工作流实操指南：把大任务拆给上百个 Agent 并行跑
author: 克劳德猎手
date: 克劳德猎手克劳德猎手
url: https://mp.weixin.qq.com/s?__biz=MzkzNTY0NDA4Mg==&mid=2247484648&idx=2&sn=7c1adc03ceb617a74afd6a68ddc55aca&chksm=c3281591e7e160cf1a826e445c8738e64186cbf0dcff59ae1daba97aea89776ccb24e14c9a01&mpshare=1&scene=24&srcid=0530LsHGSuxSR4lI0wFHyscA&sharer_shareinfo=e0400532ed498b95b77c8b50e196071e&sharer_shareinfo_first=e0400532ed498b95b77c8b50e196071e#rd
---

|  |  |
| --- | --- |
| |  | | --- | | 一行 prompt，Claude 自己规划、分配、验证，你只需要确认和执行。 | |

5 月 28 日，Anthropic 在 Claude Code 里上线了**动态工作流（Dynamic Workflows）**。它跟之前手动指派子 Agent 不是一回事——你不用想着"这个文件给 A、那个函数给 B"，Claude 自己写编排脚本、自己决定怎么分任务、自己派 Agent 去互相验证。你只需要告诉它目标。

我花了一天跑了几种场景，结论是：**这可能是 Claude Code 今年最值得学的功能**。它不是锦上添花——对于代码审计、大规模迁移、多角度验证这类任务，它就是解法本身。

这篇文章从环境配置到实操案例，一步步带你上手。不聊概念，只聊怎么跑通。说白了，你缺的不是对「动态工作流」这个概念的理解，是一个能照抄的模板。

|  |
| --- |
| 环境准备 三样东西就位就能开始 |

|  |  |
| --- | --- |
| 1 | Claude Code ≥ 2.1.154在终端跑 `claude --version` 确认。低于这个版本的话先升级：`claude update` |
| 2 | Max / Team 计划（或 Enterprise 管理员开通）Max 和 Team 用户默认开启。Enterprise 用户需管理员在 managed settings 里手动启用 Dynamic Workflows。 |
| 3 | 建议开启 Auto Mode不开 Auto Mode 的话，每个子 Agent 的每个操作都会弹确认框，并行就形同虚设了。在 `/permissions` 里启用。 |

|  |
| --- |
| 1三种方式启动动态工作流 |

动态工作流不是独立命令——它嵌入在 Claude Code 的对话和工作模式里。三种方式，从简单到自动，按需选用。

**方式一：直接说"建一个工作流"**

最直接的方式——在 Claude Code 对话里用触发短语。Claude 读到关键词，自动判断任务适合动态工作流，然后规划、展示给你确认、开始跑。

|  |
| --- |
| shell |
| ``` $ claude  > Create a workflow that audits every API endpoint in this project for missing authentication checks. For each endpoint, verify that auth middleware is present, token validation is correct, and rate limiting is configured. Report any gaps with the exact file path and line number. ``` |

Claude 会自己决定怎么拆——每个 endpoint 一个子 Agent、还是按模块分组、还是按风险等级分层——你不用操心调度细节。它展示计划后等着你确认，首次运行时特别留意这个确认步骤：看看拆得合不合理、预估 token 量你能不能接受。

**方式二：切到 ultracode effort 模式**

不想每次都说"create a workflow"？在 effort 菜单里把思考强度设为`ultracode`——这会把 effort 拉到 xhigh，Claude 遇到大任务时会自己判断要不要启动工作流。

|  |
| --- |
| shell |
| ``` # 在 Claude Code 对话中 /effort # 在弹出的菜单里选择 ultracode  # 之后正常提需求就行，不用加 "create a workflow" > 把这个项目里所有 TODO 和 FIXME 注释找出来，按优先级分类，每个附上所在文件和行号 ``` |

ultracode 模式的好处是你不用刻意想着"这任务该不该用工作流"——Claude 自己掂量。坏处是它可能在你没准备的情况下启动几十个 Agent 烧 token，所以建议先从小任务跑一次，对消耗有个数。

**方式三：配合 Auto Mode（推荐组合）**

Anthropic 官方建议：**开启动态工作流的同时务必打开 Auto Mode。**原因很简单——工作流可能启动几十上百个子 Agent，每个子 Agent 要读写文件、跑命令、检查结果。不开 Auto Mode，每个操作都弹确认，你的工作流就变成了"点点点马拉松"。

|  |
| --- |
| 💡 Auto Mode 的安全机制  Auto Mode 内置了 AI 分类器，会逐个检查工具调用。子 Agent 想执行破坏性操作（删生产数据、改环境变量等）时，分类器照样拦截——不管这命令来自主 Agent 还是子 Agent。安全底线是按操作级别、不是按会话级别生效的。 |

首次触发动工作流时，Claude Code 一定会弹出确认框，展示即将执行的内容让你审阅。后续同一会话里再触发不会重新确认——除非任务范围变化很大。这个设计给了一层安全兜底。

|  |
| --- |
| 2完整实操：用动态工作流做一次代码库安全审计 |

光讲概念没意思，我们从头到尾跑一个真实场景。假设你手里有一个 Node.js 后端项目，几十个 API 路由，你想知道：有没有哪个接口漏了权限校验？有没有 SQL 注入风险？有没有硬编码的密钥？

以前的做法是：手动翻文件、跑 lint、配 SAST 工具、出报告——一个人一上午就过去了，还不敢保证全覆盖。用动态工作流，**一条 prompt 搞定**。

**第一步：打开 Auto Mode，切到 ultracode**

|  |
| --- |
| shell |
| ``` $ claude  > /permissions # 在弹出的面板里选 "Auto mode"  > /effort # 选 "ultracode" ``` |

**第二步：下达审计任务**

|  |
| --- |
| shell |
| ``` > Create a workflow that performs a comprehensive security audit of this project. Check for: (1) missing authentication middleware on any API route, (2) SQL injection risks in database queries, (3) hardcoded API keys or secrets, (4) unsafe deserialization patterns. For each finding, report the exact file path, line number, severity level, and suggested fix. Have one set of agents do the scanning, then have a second set independently verify every finding flagged as high or critical. ``` |

这条 prompt 有几个关键点值得拆开说：

**① 明确检查项。**别只说"做安全审计"——告诉 Claude 具体检查什么。越具体，子 Agent 拆分越准确，漏检越少。

**② 要求输出格式。**"文件路径 + 行号 + 严重程度 + 修复建议"——这决定了你拿到的是可执行的报告，还是一堆你还要自己去翻的模糊描述。

**③ 加验证层。**"第一组扫、第二组独立验证高危发现"——这是动态工作流区别于普通子 Agent 的核心能力：对抗性验证。不是投票（majority wins），是挑刺（adversarial refutation）。高危发现必须经得起第二组 Agent 的攻击性复核。

**第三步：看 Claude 规划并确认**

Claude 会展示一份编排计划：大概多少个子 Agent、各负责哪个模块、谁先跑谁后跑、验证层怎么配置。你审一遍——主要看：

•拆得合不合理？有的文件太大可能需要单独分一个 Agent
•验证 Agent 够不够？高危项至少要两个独立验证
•预估 token 量你接不接受？首次跑建议从子集开始

确认后，子 Agent 开始并行跑。你不会看到几十个终端窗口——编排在后台静默运行，Claude 只把汇总结果和关键进度汇报给你。这跟以前手动开 tmux 窗口盯着每个 Agent 跑的体验完全不同。

**第四步：拿到报告，人工复核关键发现**

跑完后 Claude 会给出结构化的审计报告，类似这样：

|  |
| --- |
| ▸ 审计报告示例（摘要） |
| **🔴 高危（3 处，均已独立验证）** • src/routes/admin.js:42 — 缺少 auth 中间件，任何人可访问 /admin/users • src/routes/payment.js:108 — 原始 SQL 拼接，存在注入风险 • .env.example:15 — AWS\_SECRET\_ACCESS\_KEY 硬编码示例值  **中危（7 处）** • src/routes/api.js:203 — 反序列化未校验输入类型 • ...  **验证记录：**3 处高危发现经 2 个独立 Agent 复核确认，无假阳性。 |

到这里，你拿到的不只是一份列表——每处高危发现都有独立验证记录，不会把 lint 误报当漏洞交给你。

|  |
| --- |
| 3控制 token 成本 + 常见问题 |

动态工作流最大的代价是 token 消耗。Anthropic 自己在公告里写了："动态工作流消耗的 token 可能远多于普通 Claude Code 会话。"这不是劝退，是让你有心理准备。

几条控制成本的实操建议：

**先小后大。**第一次别直接扫整个 monorepo。挑一个模块、一个子目录，跑一遍看大概烧多少 token。心里有数了再放大范围。

**限制子 Agent 数量。**prompt 里可以加一句"limit to 10 parallel agents"来控制规模。Claude 会尊重这个约束。

**用 `/usage` 监控。**Claude Code 2.1.149 之后`/usage`按 skills、subagents、MCP 服务器拆分了账单——能清楚看到动态工作流贡献了多少。

**不需要验证层的任务就别加。**对抗性验证（让另一组 Agent 去推翻前面的结论）很烧 token。如果任务错了也没太大后果，只做并行扫描、不加验证层。

|  |
| --- |
| 💡 什么时候该加验证层  生产环境配置变更、数据库 schema 迁移、安全漏洞修复——错一次代价很高的事，值得多烧 token 加验证层。扫死代码、清理未使用 import、格式化检查——这些不需要，并行扫就够了。 |

**常见问题：**

|  |
| --- |
| ⚠️ 工作流跑到一半断了怎么办？  动态工作流内置了进度持久化。断开后重新进入同一会话，Claude 会从断点继续，不会重头跑。但如果整个会话被关闭了（不是断开），恢复不了——重要任务建议在稳定环境中跑。 |

|  |
| --- |
| ⚠️ Enterprise 用户找不到这个功能？  Enterprise 计划默认关闭 Dynamic Workflows。找管理员在 managed settings 里启用。启用后所有团队成员都能用，不需要逐个开通。 |

|  |
| --- |
| ⚠️ Auto Mode 开了但子 Agent 还是弹确认？  Auto Mode 的 AI 分类器对某些操作（删文件、改环境变量、网络请求到陌生域名）仍然会触发确认。这不是 bug，是安全设计。如果想对特定操作免确认，在`/permissions`里配置 allowlist。 |

· · ·

|  |
| --- |
| 动态工作流还在 research preview，但方向不会错了 现在不入场，三个月后可能就落后半个身位了 |

动态工作流目前还是 research preview——token 消耗高、场景边界在摸索、行为可能变化。但有一件事已经很清楚了：

AI 辅助编程的下一个阶段不是"模型更强"，是**协调成本被压到接近零**。Bun 从 Zig 搬到 Rust 用了 11 天，75 万行，99.8% 测试通过——这事如果放在一年前，哪怕是 AI 乐观派也不会信。它靠的不是某个 Agent 更强了，是 Cladue 把"怎么分任务、怎么并行、怎么验证"这件事从人的手里接了过去。

我的建议很简单：**这周就试一次。**从代码库安全审计或死代码扫描开始——任务本身有价值，又能让你感受到动态工作流跟以前手动开子 Agent 的区别。跑通一次之后，你会开始用不同的眼光看那些"太大了、不想碰"的任务。

|  |
| --- |
| Agent 编队这件事才刚开始，我会持续跟进每一版变化  关注「克劳德猎手」，第一时间拿到实操经验  关注公众号「克劳德猎手」获取更多内容 👇 |

|  |
| --- |
| 📌 相关阅读 |
| [Claude Code 2.1.154：Opus 4.8 到了，动态工作流把 Agent 编队跑起来了](https://mp.weixin.qq.com/s?__biz=MzkzNTY0NDA4Mg==&mid=2247484625&idx=2&sn=68f324eee25227c9f830e5397454f2fd&scene=21#wechat_redirect) |
| [Claude Code Routines 完全指南：三种触发方式，让 Agent 脱离你的电脑独立运转](https://mp.weixin.qq.com/s?__biz=MzkzNTY0NDA4Mg==&mid=2247483933&idx=2&sn=629ac64272698ad8c0310d212d6393ca&scene=21#wechat_redirect) |
| [Claude Code Ultraplan 完整使用指南：把大规划交给云端](https://mp.weixin.qq.com/s?__biz=MzkzNTY0NDA4Mg==&mid=2247483799&idx=1&sn=0817563058e7b1dbf1007af690584513&scene=21#wechat_redirect) |