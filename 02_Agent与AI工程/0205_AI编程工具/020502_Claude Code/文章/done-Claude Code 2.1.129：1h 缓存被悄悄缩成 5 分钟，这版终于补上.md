> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Claude Code 2.1.129：1h 缓存被悄悄缩成 5 分钟，这版终于补上
author: 克劳德猎手
date: 克劳德猎手克劳德猎手
url: https://mp.weixin.qq.com/s?__biz=MzkzNTY0NDA4Mg==&mid=2247484333&idx=1&sn=d3ff2962ac952995be0b853d92e1903d&chksm=c328c0c7913f2235cd7fe768c682dbd1f058ab56da1d85d3f8e16fc7a2db96badad5cc01e969&mpshare=1&scene=24&srcid=0506QkHrsHtkQAa8HjPlsEwB&sharer_shareinfo=091e3b3b1c2f68d0468ad5c41636a9ed&sharer_shareinfo_first=091e3b3b1c2f68d0468ad5c41636a9ed#rd
---

|  |  |
| --- | --- |
| |  | | --- | | 距 2.1.128 才 1 天，3 项新功能加 22 条修复——里面藏着一个最该停的血。 | |

|  |
| --- |
| 2.1.129 距 2.1.128 只过了 1 天，这种紧贴上一版的发布节奏，一般意味着 .128 出去之后冒出了几个不能等下个迭代的问题。这次 25 条改动里，3 项是真新功能（`--plugin-url`和两个新环境变量），其余全是修。其中一条修复直接关系到 prompt cache 的钱包：**1 小时 cache TTL 被静默降到了 5 分钟**——你设了长缓存、付了长缓存的写入费，但只拿到了短缓存的命中率。 |

先看数据再读条目，最后挑 3 条展开。

|  |
| --- |
| 1指标：tokens 反弹 9.8% |

和上版「token 略降、文件减 4」的清账气质不同，这版数字往回涨了一截：

|  |
| --- |
| ▸ 2.1.129 关键指标 |
| **Bundle 体积：**+147.5 kB（+0.4%） **Prompt token 总量：**+8,894（+9.8%） **Prompt 文件数：**+3（+3.9%） **距 2.1.128 间隔：**1 天 |

最值得留意的是内部占比的重排：`system-reminder`从 39.4% 涨到 42.0%（+2.6 个百分点），`system`从 29.0% 降到 26.4%（-2.6 个百分点）——这不是巧合，是把固定 system 段里的指令搬去了 reminder 通道。`tools`微涨 0.9%，`agent`/`skill`都微降。这条线和过去几版方向一致：能用按需 reminder 兜的，就别在常驻 system 里一直挂着。

|  |
| --- |
| 2完整双语 Changelog |

分三组：新增能力、行为调整、问题修复。

|  |
| --- |
| ✦ 新功能 / New Features |
| Added --plugin-url <url> flag to fetch a plugin .zip archive from a URL for the current session. |
| 新增`--plugin-url <url>`标志，从 URL 直接拉一个插件 zip 包到当前会话使用。 |
| Added CLAUDE\_CODE\_FORCE\_SYNC\_OUTPUT=1 env var to force-enable synchronized output on terminals that auto-detection misses (e.g. Emacs eat). |
| 新增`CLAUDE_CODE_FORCE_SYNC_OUTPUT=1`，在自动探测漏掉的终端上强制开同步输出（比如 Emacs`eat`）。 |
| Added CLAUDE\_CODE\_PACKAGE\_MANAGER\_AUTO\_UPDATE: when set on Homebrew or WinGet installations, Claude Code runs the upgrade command in the background and prompts to restart. |
| 新增`CLAUDE_CODE_PACKAGE_MANAGER_AUTO_UPDATE`：在 Homebrew 或 WinGet 安装上启用后，Claude Code 会在后台跑升级命令并提示重启——把 Native 安装独享的「自动后台更新」补给了包管理器用户。 |

|  |
| --- |
| ✦ 行为调整 / Improvements |
| Plugin manifests: themes and monitors should now be declared under "experimental": { ... }. Top-level declarations still work but claude plugin validate will warn. |
| 插件清单的`themes`和`monitors`现在应该挪到`"experimental": { ... }`下。顶层声明仍能用但会被`claude plugin validate`警告——这两块还没正式转正。 |
| Gateway /v1/models discovery for the /model picker is now opt-in via CLAUDE\_CODE\_ENABLE\_GATEWAY\_MODEL\_DISCOVERY=1 (was automatic in 2.1.126–2.1.128). |
| `/model`选择器走的 Gateway`/v1/models`发现机制改回手动开关——需设`CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY=1`。前三版的自动探测被回滚了。 |
| Ctrl+R history picker now defaults to searching all prompts across all projects, matching pre-2.1.124 behavior. Press Ctrl+S to narrow to the current project or session. |
| `Ctrl+R`历史搜索默认改回跨所有项目（2.1.124 之前的老行为），按`Ctrl+S`才收窄到当前项目/会话。 |
| Third-party deployments (Bedrock, Vertex, Foundry, or ANTHROPIC\_BASE\_URL gateway) no longer see spinner tips pointing at first-party Anthropic surfaces. |
| 第三方部署（Bedrock、Vertex、Foundry、`ANTHROPIC_BASE_URL`网关）下不再出现指向 Anthropic 自家产品的等待提示——这种交叉引用被清掉了。 |
| skillOverrides setting now works: off hides from model and /, user-invocable-only hides from model only, name-only collapses description. |
| `skillOverrides`终于真的生效——`off`对模型和`/`都隐藏，`user-invocable-only`只对模型隐藏，`name-only`折叠掉描述。 |
| The claude\_code.pull\_request.count OTel metric now counts PRs/MRs created via MCP tools, not just shell commands. |
| `claude_code.pull_request.count`OTel 指标把 MCP 工具创建的 PR/MR 也计入，之前只统计 shell 命令创建的。 |
| Policy refusal error messages now include the API Request ID for easier support debugging. |
| 策略拒绝（policy refusal）错误信息里带上 API Request ID——给企业用户开 ticket 排查省一步。 |

|  |
| --- |
| ✦ Bug 修复 / Bug Fixes |
| Fixed 1-hour prompt cache TTL being silently downgraded to 5 minutes. |
| 修复 1 小时 prompt cache TTL 被静默降级成 5 分钟——这条最贵，下面会单独展开。 |
| Fixed /context dumping its rendered ASCII visualization grid into the conversation, wasting ~1.6k tokens per call. |
| `/context`把它自己渲染的那张 ASCII 可视化网格塞进对话里，每次调用浪费约 1.6k token。 |
| Fixed OAuth refresh race after wake-from-sleep that could log out all running sessions. |
| 修复电脑唤醒后 OAuth 刷新竞态——之前能把所有正在跑的会话一次性踢下线。 |
| Fixed agent panel below the prompt being hidden when subagents are running (regression in 2.1.122). |
| 修复子代理跑动时，提示行下方的 agent 面板被隐藏（2.1.122 引入的回归）。 |
| Fixed external-editor handoff (Ctrl+G) blanking the conversation history above the prompt. |
| 修复`Ctrl+G`切到外部编辑器时，提示行上方的会话记录被清空。 |
| Fixed Bash(mkdir \*), Bash(touch \*) and similar allow rules not being honored for in-project paths. |
| 修复`Bash(mkdir *)`、`Bash(touch *)`这类 allow 规则在项目内路径下没生效。 |
| Fixed deniedMcpServers patterns with a \*:// scheme wildcard not matching mixed-case hostnames. |
| 修复`deniedMcpServers`用`*://`协议通配时，匹配大小写混合主机名失败。 |
| Fixed server-managed settings policy not applying for enterprise/team users whose stored OAuth credentials lacked the user:inference scope. |
| 修复服务端下发的 settings 策略对部分企业/团队用户没生效——其存储的 OAuth 凭证缺少`user:inference`scope。 |
| Fixed cache-miss warning appearing spuriously after /clear or compaction when changing /effort or /model. |
| 修复`/clear`或 compact 之后切`/effort`或`/model`时虚假的 cache-miss 警告。 |
| Fixed API errors with unrecognized 400 status codes showing raw JSON instead of the underlying error message. |
| 修复未识别的 API 400 错误码直接抛原始 JSON，遮住了真正的错误信息。 |
| Fixed /clear not resetting the terminal tab title after a conversation. |
| 修复`/clear`之后终端 tab 标题没重置。 |
| Fixed session title chip from /rename disappearing while a permission or other dialog is active. |
| 修复权限弹窗打开期间`/rename`改的会话标题 chip 消失。 |
| Fixed /agents Library list arrow-key navigation: the highlighted agent now stays visible when the list exceeds the viewport. |
| 修复`/agents`Library 列表的箭头键导航——高亮项超过可视区时跟着滚。 |
| Fixed /branch success message not including the new branch's session id for /resume. |
| 修复`/branch`成功提示里少了新分支的 session id，导致`/resume`无法直接接上。 |
| Fixed bold headers with keycap/ZWJ/skin-tone emoji losing trailing characters in fullscreen mode. |
| 修复全屏模式下，含 keycap/ZWJ/肤色 emoji 的粗体标题尾字符被吞。 |
| Fixed harmless WebSocket warning being logged as an error in --debug during voice mode. |
| 修复 voice 模式`--debug`下，无害的 WebSocket 警告被错记为 error。 |
| [VSCode] Fixed /clear not clearing the conversation context and displayed transcript. |
| [VSCode] 修复`/clear`没真正清掉对话上下文和已显示的 transcript。 |

|  |
| --- |
| 3三条值得展开的 |

挑 3 条——一条最贵，一条最隐形，一条带来新玩法。

**① 1h prompt cache TTL 被悄悄压成 5min**

这条改动放在 changelog 一行轻飘飘带过，但翻译成账单语言是这样的：Anthropic 的 prompt cache 有两档 TTL——5 分钟和 1 小时。1h 档的**写入费比 5min 贵 1 倍**，目的是让你在长时间停顿后还能命中缓存、节约后续输入费用。

|  |  |
| --- | --- |
| ❌ 修复前 | ✅ 2.1.129 起 |
| 设 1h TTL → 实际只活 5min 付的是 1h 的写入费 | 设 1h TTL → 真活 1h 付的钱终于和拿到的服务对上 |
| 长间隔批量任务每轮都重写缓存 | 长间隔批量任务真能复用缓存 |

这是个静默失血的 bug：不报错、不提示，只是让你按 1h 的价格买 5min 的服务。要说什么场景受影响最大——**间隔超过 5 分钟、不到 1 小时的批量调用**，比如定时跑的 Routines、人工反复来回的长会话、跨多步思考的 agent。这些场景里你以为缓存还在，模型那边其实已经把它抹掉重算。

如果你最近一两周明明设了 1h TTL，却发现 cache hit rate 与预期对不上，大概率不是你的 prompt 没对齐——是这条 bug。升级后再观察一周再下结论。

**② /context 自己往对话里塞 1.6k token**

`/context`是用来看当前对话上下文占用的命令。它会渲染一张 ASCII 网格图——每个块代表上下文的一类来源。问题是：这张图被一并写进了对话历史里，**每次调用浪费约 1.6k token**。

|  |
| --- |
| 💡 这是个递归式的浪费  越担心上下文涨爆、越频繁调`/context`，对话就被它自己污染得越多。养成习惯每隔几步看一眼的人，相当于每次手动给会话注水 1.6k。这种 bug 配合上一条 prompt cache 失效，复合伤害更明显——既污染缓存命中，又往对话里灌字。 |

这条放进 changelog 的措辞挺有意思——「dumping its rendered ASCII visualization grid into the conversation」——能看出写 release note 的人也带着点恼火。

**③ --plugin-url：插件分发的中间形态**

2.1.128 已经允许`--plugin-dir`接收 .zip 包。这一版进一步把 zip 的来源扩展到 URL：

|  |
| --- |
| ▸ Terminal |
| # 直接从 URL 拉 zip 包，仅本会话生效  $ claude--plugin-urlhttps://example.com/my-plugin.zip |

这是把插件分发链路彻底打通了。在此之前你要分享一个未上 marketplace 的插件，得让别人 git clone 或本地解压。`--plugin-url`把这步并成了一行——同事丢一个内部 S3 链接、CI 跑起来时直接拉取、企业里发一个内网 zip 让一组人临时试。

「仅本会话生效」也很关键——它不是装到全局，会话退出就清空。这样的好处是临时调试和长期使用走两条路，不混在一起。这种「URL → 临时使用」的形态，看起来像是在为后续的更轻量插件分享做铺垫，比如某天可以一键复制一段 prompt 链接给同事。

|  |
| --- |
| 4几条不显眼但有意思的 |

|  |  |
| --- | --- |
| ① | Homebrew/WinGet 也能后台自动更新了设上`CLAUDE_CODE_PACKAGE_MANAGER_AUTO_UPDATE`，brew/winget 安装的也能像 Native 一样后台升级——之前这是 Native 用户的特权。 |
| ② | Ctrl+R 默认改回跨项目搜索2.1.124 改成「只搜本项目」之后被骂，这版回到老行为——按`Ctrl+S`才收窄。这种来回的反复说明默认的取舍不容易拍板。 |
| ③ | 唤醒后不再一次性踢掉所有会话合盖恢复后的 OAuth 刷新竞态被修——之前会把所有正在跑的 Claude Code 全踢下线，背景任务一次性中断。笔记本党解放。 |
| ④ | 第三方部署不再被 Anthropic 自家提示骚扰Bedrock / Vertex / Foundry / 网关 用户不再看到指向 Anthropic 自家产品的等待提示。企业部署在意的不是功能而是边界。 |

· · ·

|  |
| --- |
| 小结：止血版 紧贴上一版，因为有几条不能等下个迭代 |

2.1.129 跟在 2.1.128 屁股后面只过 1 天发出来——这种节奏在过去几个月不算少见，但每次出现都意味着上一版后冒出了不能等到下个迭代的修复。这次直接动到的是钱包（1h cache TTL）、对话上下文（`/context`自污染）、跨会话连续性（OAuth 刷新踢人）这三块——任何一条搁着拖一周都会有用户跳出来骂。

从功能侧看，`--plugin-url`把插件的临时使用门槛压到一行命令；`CLAUDE_CODE_PACKAGE_MANAGER_AUTO_UPDATE`把 Native 独享的自动更新补给 brew/winget 用户；`/v1/models`自动发现回退到 opt-in 也是一个谨慎的退一步——前三版自动开了，这版把口子收回来，应该是有第三方网关返回不规范导致的兼容问题。

如果只挑一件事现在做，就是看一下你过去一两周的 cache hit 表现——如果一直觉得 1h TTL 没多少效果，先升上来再评估。

|  |
| --- |
| 💡 升级方式  Native 安装（推荐）会自动后台更新，需立即应用执行`claude update`。Homebrew 用户：`brew upgrade claude-code`（stable）或`brew upgrade claude-code@latest`。WinGet：`winget upgrade Anthropic.ClaudeCode`。这版起 brew/winget 也能用`CLAUDE_CODE_PACKAGE_MANAGER_AUTO_UPDATE`启用后台自动升级。 |

|  |
| --- |
| 每版我都拆一遍  Claude Code 下次发版我接着盯  关注公众号「克劳德猎手」获取更多内容 👇 |

|  |
| --- |
| 📌 相关阅读 |
| [Claude Code 2.1.128：38 条修复，子代理缓存少写三倍](https://mp.weixin.qq.com/s?__biz=MzkzNTY0NDA4Mg==&mid=2247484312&idx=1&sn=0976b41f0a84a431ed5be2dfd03207f1&scene=21#wechat_redirect) |
| [Claude Code 2.1.126：30 多项变更，新增 project purge 一键清空](https://mp.weixin.qq.com/s?__biz=MzkzNTY0NDA4Mg==&mid=2247484302&idx=1&sn=945c4a414edb2d8121d27586c3f66a0b&scene=21#wechat_redirect) |
| [Claude Code 2.1.121：4 处内存泄漏修复，Hooks 拿到全工具改写权](https://mp.weixin.qq.com/s?__biz=MzkzNTY0NDA4Mg==&mid=2247484251&idx=1&sn=1251e8c2036183f9e40d6e1cf66735de&scene=21#wechat_redirect) |