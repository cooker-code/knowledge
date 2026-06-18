---
title: Claude Code 2.1.110：全屏无闪烁、手机推送、三十项修复同日到齐
author: 克劳德猎手
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzNTY0NDA4Mg==&mid=2247483925&idx=2&sn=79d9802d3e7ff4e5dd6e8185ad178b69&chksm=c3d0d9c74b29e26fd2615f61513276fb28e80dd59c97c0761d3cd1ae475e53f9368256e13ab9&mpshare=1&scene=24&srcid=0417mGqksKvX2l0YxWf4DloZ&sharer_shareinfo=14084121eb03b2fda190dd538b0bb33b&sharer_shareinfo_first=14084121eb03b2fda190dd538b0bb33b#rd
---

|  |
| --- |
| 更新日志 · Claude Code |
| Claude Code 2.1.110三十项修复，三个新方向 |
| |  | | --- | | 与 2.1.109 同日发布，规模却大得多：/tui 全屏无闪烁、手机推送通知、Write 工具感知 diff 编辑，加上 20+ 项 bug fix。 | |
| 📖 阅读约 6 分钟 | 适合 Claude Code 日常用户及开发者 |

|  |
| --- |
| 4 月 15 日，Anthropic 先发了只有 1 条改动的 2.1.109，几小时后又推了 2.1.110——后者带来了 13 项新功能和改进、近 20 项 bug fix。**这不是偶然，而是一个大版本积压后集中释放的节奏。** |

|  |
| --- |
| 1完整更新日志 |

|  |
| --- |
| ✦ 新功能与改进 / Features & Improvements |
| Added `/tui` command and `tui` setting — run `/tui fullscreen` to switch to flicker-free rendering in the same conversation |
| 新增 `/tui` 命令——运行 `/tui fullscreen` 可在同一会话中切换到无闪烁全屏渲染模式 |
| Added push notification tool — Claude can send mobile push notifications when Remote Control and "Push when Claude decides" config are enabled |
| 新增推送通知工具——开启 Remote Control 并配置"Claude 自行决定时推送"后，Claude 可主动向手机发送通知 |
| Write tool now informs the model when you edit the proposed content in the IDE diff before accepting |
| Write 工具增强：当你在 IDE diff 界面接受前修改了建议内容，模型现在会收到通知，感知到你的实际改动 |
| Changed `Ctrl+O` to toggle between normal and verbose transcript only; focus view is now toggled separately with the new `/focus` command |
| `Ctrl+O` 现在仅切换普通/详细 transcript 模式；Focus 视图改为独立的 `/focus` 命令控制 |
| Session recap is now enabled for users with telemetry disabled (Bedrock, Vertex, Foundry, `DISABLE_TELEMETRY`). Opt out via `/config` or `CLAUDE_CODE_ENABLE_AWAY_SUMMARY=0` |
| Session recap（离开后自动总结）现已对关闭遥测的用户开放（Bedrock/Vertex/Foundry 及设置了 `DISABLE_TELEMETRY` 的用户），可通过 `/config` 或环境变量关闭 |
| Improved `/plugin` Installed tab — items needing attention and favorites appear at top, disabled items hidden behind a fold, `f` key favorites selected item |
| 改进 `/plugin` 已安装列表：需要关注的项目和收藏置顶，已禁用插件折叠隐藏，`f` 键快速收藏 |
| `--resume`/`--continue` now resurrects unexpired scheduled tasks; `/context`, `/exit`, `/reload-plugins` now work from Remote Control clients |
| `--resume`/`--continue` 现可恢复未过期的计划任务；`/context`、`/exit`、`/reload-plugins` 现在可以从手机/网页 Remote Control 端执行 |

|  |
| --- |
| 🔧 问题修复 / Bug Fixes（精选） |
| Fixed MCP tool calls hanging indefinitely when the server connection drops mid-response on SSE/HTTP transports |
| 修复 MCP 服务器连接在响应中途断开时工具调用无限挂起的问题（SSE/HTTP 传输） |
| Fixed non-streaming fallback retries causing multi-minute hangs when the API is unreachable |
| 修复 API 不可达时非流式回退重试导致持续数分钟卡顿的问题 |
| Fixed garbled startup rendering in macOS Terminal.app and other terminals that don't support synchronized output |
| 修复 macOS Terminal.app 等不支持同步输出的终端在启动时界面乱码的问题 |
| Hardened "Open in editor" actions against command injection from untrusted filenames |
| 加固"在编辑器中打开"功能，防止不可信文件名导致的命令注入漏洞 |
| Fixed `PermissionRequest` hooks returning `updatedInput` not being re-checked against `permissions.deny` rules; `setMode:'bypassPermissions'` updates now respect `disableBypassPermissionsMode` |
| 修复 hook 返回 `updatedInput` 后未再次经过 deny 规则检查的权限绕过漏洞；`bypassPermissions` 模式现在正确遵守 `disableBypassPermissionsMode` |
| Fixed high CPU usage in fullscreen when text is selected while a tool is running; fixed dropped keystrokes after CLI relaunches |
| 修复全屏模式下工具运行时选中文字导致 CPU 占用过高的问题；修复 CLI 重启后按键丢失的问题 |

|  |
| --- |
| 2三个值得展开的变化 |

/tui fullscreen：终端 UX 的一次认真投入

长期用 Claude Code 的人都遇到过屏幕闪烁——每次工具调用返回、模型更新输出，终端画面就抖一下。这在快速操作时会很明显，影响专注度。`/tui fullscreen` 引入了"同步输出"机制，让所有渲染变更在同一帧提交，从而消除闪烁。

值得注意的是，它是**同一会话内直接切换**，不需要重开终端。同时还新增了 `autoScrollEnabled` 配置项，可以关闭全屏下的自动滚动。这两项合起来说明：Anthropic 开始把终端交互体验当成正经的产品工程来做，而不是留给用户"自己配置终端去适应"。

Push Notification：Remote Control 生态在成形

推送通知单独看是个小功能，但放在 Remote Control 的框架里看就不一样了。Remote Control 让用户可以从手机或网页端控制 Claude Code 的 CLI 会话，push notification 则让 Claude 可以主动"叫"用户回来——比如任务完成了、需要确认了、遇到错误了。

|  |
| --- |
| 💡 配合使用效果  启动一个长任务（代码重构、大规模文件处理），离开终端去做别的事，Claude 完成时主动推送手机通知——这才是"异步 AI 工作流"真正可用的形态。同时这次还修复了 Remote Control 会话标题不同步、reconnect 错误提示等问题，配套体验在补齐。 |

Write 工具感知 diff 编辑：闭合了一个反馈缺口

之前的工作流有一个隐形问题：Claude 建议修改一段代码 → IDE diff 界面弹出 → 用户觉得建议不完全正确，手动微调了几行 → 点接受。**但 Claude 不知道你改了什么**，它认为自己的建议被 100% 采用，下一步推理可能就基于错误的前提。

这次 Write 工具会把"用户修改了建议内容"这个信息反馈给模型，让模型知道实际写入文件的内容与它建议的有出入。这对长对话、多步骤代码任务的一致性很重要，是 human-in-the-loop 工作流的一个实质性改进。

|  |
| --- |
| 3两项安全修复不能忽略 |

这次有两处安全相关修复，藏在 bug fix 列表里，但重要性不亚于功能更新：

|  |  |
| --- | --- |
| ① | 命令注入加固"Open in editor" 功能修复了一个通过恶意文件名注入命令的漏洞。在 AI 工具频繁处理外部代码和未知文件的场景下，这类漏洞有实际攻击面。 |
| ② | 权限绕过修复`PermissionRequest` hook 返回修改后的输入时，未再经过 deny 规则二次检查——意味着精心构造的 hook 可以绕过拒绝规则。企业部署场景下用了自定义 hooks 的团队应关注此项。 |

· · ·

|  |
| --- |
| 2.1.110 说明什么节奏 密集发布背后的逻辑 |

同一天发两个版本——一个只有 1 条改动，一个有 30+ 条——这不是发布节奏混乱，而是主干持续开发的产物：**小改动快速发布，大版本积攒够了再合并出来**。

2.1.110 三个主要方向加在一起看很清晰：终端体验（/tui）、移动端控制（push notification）、人机协作精度（Write diff 感知）——这是在从三个维度把 Claude Code 从"能用"推向"好用"。

两项安全修复也是信号：随着 Claude Code 在企业中的使用越来越广，Anthropic 开始更认真地对待权限模型和命令执行的安全边界，而不只是功能层面的迭代。

|  |
| --- |
| 💡 如何升级  Native 安装会自动后台更新，无需手动操作。如需立即应用：`claude update`。Homebrew 用户：`brew upgrade claude-code`。 |

|  |
| --- |
| 每一个版本我们都帮你拆开看  关注「克劳德猎手」不漏任何更新  关注公众号「克劳德猎手」获取更多内容 👇 |