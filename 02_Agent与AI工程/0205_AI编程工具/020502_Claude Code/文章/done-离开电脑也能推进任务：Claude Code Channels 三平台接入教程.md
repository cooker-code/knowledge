> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: 离开电脑也能推进任务：Claude Code Channels 三平台接入教程
author: 克劳德猎手
date:
url: https://mp.weixin.qq.com/s?__biz=MzkzNTY0NDA4Mg==&mid=2247483862&idx=2&sn=af3690bc943276f6739a055ff05b3c9c&chksm=c30a31dd060e8ceb51c4b2e1e01cca90e77060beef7a2f3d04be296a54253c22b5d1213c52b9&mpshare=1&scene=24&srcid=04165gnFvkWtq7cnEgU4lhhW&sharer_shareinfo=8fe3516e35d5c2f26bc4c30f0d53d6e5&sharer_shareinfo_first=8fe3516e35d5c2f26bc4c30f0d53d6e5#rd
---

|  |
| --- |
| 教程 · Claude Code |
|  |
| |  | | --- | | 通过 Telegram、Discord 或 iMessage 向本地 Claude Code 会话发指令——离开电脑，任务继续跑。 | |
| 📖 阅读约 10 分钟 | 适合 Claude Code 日常用户 |

|  |
| --- |
| 你在外面开会，突然想到 PR 里有个变量命名不对。以前只能记在备忘录等回去改，现在可以掏出手机，发一条消息：**「把 auth.ts 里的 userID 全部改成 userId 然后提 PR」**——Claude Code 在你的电脑上执行，完事回复你。 |

这就是 **Claude Code Channels** 的核心能力：把你本地正在运行的 Claude Code 会话接入消息平台，让你在 Telegram、Discord 或 iMessage 里直接发指令，结果通过同一个 App 回复给你。

它不是云端服务——Claude 仍然在你的本地机器上运行，完整访问你的文件系统、MCP 服务器和 git。消息只是触发和回传的通道。

|  |
| --- |
| 1它是怎么工作的 |

Channels 基于 Claude Code 已有的 MCP（Model Context Protocol）架构扩展而来。整个链路如下：

|  |  |
| --- | --- |
| 1 | 你在手机 App 里发消息Telegram / Discord / iMessage，发送自然语言指令 |
| 2 | MCP Server 接收并封装Channel 插件将消息包装为 channel event，推入你的 Claude Code 会话 |
| 3 | Claude 在本地处理完整访问文件系统、MCP 服务器、git，执行你的指令 |
| 4 | 结果回传到你的 AppClaude 通过 reply / react / edit\_message 工具，把结果发回消息平台 |

|  |
| --- |
| 2前置条件 |

接入任何平台之前，先确认以下几点：

|  |  |
| --- | --- |
| ① | Claude Code 已安装并运行会话必须保持活跃，关闭终端会断开 Channel 连接 |
| ② | 选定目标平台Telegram（最快上手）、Discord（团队协作）、iMessage（macOS 专属） |
| ③ | Channels 功能处于 Preview 阶段目前只支持官方白名单内的插件，自定义 Channel 需启用开发者标志 |

|  |
| --- |
| 3Telegram 接入（推荐，5 分钟） |

三个平台里 **Telegram 上手最快**，整个流程约 5 分钟，不需要创建服务器或申请权限审核。

**第一步：在 Telegram 创建 Bot**

打开 Telegram，搜索并进入 `@BotFather`，发送以下命令：

|  |
| --- |
| TELEGRAM · BotFather |
| /newbot |

BotFather 会引导你输入 Bot 名称和用户名（用户名必须以 `bot` 结尾），完成后会给你一串 Token，形如：

|  |
| --- |
| ▸ BotFather 返回示例 |
| 7294810562:AAGxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx |

**第二步：在 Claude Code 中安装 Telegram 插件**

回到你的 Claude Code 会话，运行：

|  |
| --- |
| SHELL · 安装插件 |
| /channels install telegram |

**第三步：配置 Token 并完成配对**

插件会提示你输入 Bot Token，将第一步获得的 Token 粘贴进去。随后系统生成一个 6 位配对码，在 Telegram 里向你的 Bot 发送这个验证码完成绑定：

|  |
| --- |
| ▸ 预期输出 |
| ✅ Telegram channel connected Send this code to your bot to pair: **A7K2M9** Waiting for pairing... |

向 Bot 发送 `A7K2M9`（你的实际配对码）后，终端显示 `Paired successfully` 即完成。

|  |
| --- |
| 💡 验证连接  配对完成后，直接在 Telegram 里给 Bot 发「你好」，Bot 应当在几秒内回复。如果超过 30 秒没有响应，检查终端中的 Claude Code 会话是否仍处于活跃状态。 |

|  |
| --- |
| 4Discord 接入（团队协作场景） |

Discord 的配置步骤比 Telegram 多，但它支持多人协作——整个团队可以通过 Guild 频道向同一个 Claude Code 会话发指令，适合共享的开发环境或调试场景。

**第一步：创建 Discord 应用**

进入 `discord.com/developers/applications`，点击「New Application」创建一个新应用，切换到「Bot」标签页，点击「Add Bot」。

**第二步：开启 Message Content Intent**

在 Bot 设置页面，找到「Privileged Gateway Intents」，**务必开启「Message Content Intent」**——没有这个权限，Bot 收不到消息内容。

|  |
| --- |
| ⚠️ 注意  Message Content Intent 属于特权权限。在开发者门户里如果看到「Requires verification」的提示，说明你的 Bot 已在 100 个以上的服务器里——个人使用不会触发这个门槛。 |

**第三步：复制 Token 并邀请 Bot 进服务器**

在 Bot 页面重置并复制 Token。然后在「OAuth2 → URL Generator」里勾选 `bot` 和 `applications.commands` 权限，生成邀请链接，将 Bot 邀请进你的服务器。

**第四步：在 Claude Code 中安装插件**

|  |
| --- |
| SHELL · 安装 Discord 插件 |
| /channels install discord |

按提示输入 Bot Token 和目标频道 ID（在 Discord 里右键频道可复制 ID，需先在设置里开启「开发者模式」）。

|  |
| --- |
| 5iMessage 接入（macOS 专属） |

iMessage Channel 是三个方案中最「原生」的一个——**完全不依赖外部服务**，插件直接读取 Mac 的 Messages 数据库，通过 AppleScript 发送回复。缺点是仅限 macOS，且需要给终端应用授权。

**第一步：授予终端完全磁盘访问权限**

进入「系统偏好设置 → 隐私与安全 → 完全磁盘访问权限」，将你使用的终端（Terminal.app 或 iTerm2 等）添加进去并开启。这是必须的，因为插件需要读取 `~/Library/Messages/chat.db`。

|  |
| --- |
| 🚫 安全提醒  完全磁盘访问权限是高级权限，授予后终端可访问你所有私人文件。确认你的终端应用来源可信，不要轻易为不明来源的终端开启此权限。 |

**第二步：安装插件并配置发送方号码**

|  |
| --- |
| SHELL · 安装 iMessage 插件 |
| /channels install imessage |

按提示输入你的手机号（iMessage 账号），插件会开始监听来自该号码的消息。用你的 iPhone 给这个号码发一条消息进行验证。

|  |  |  |  |
| --- | --- | --- | --- |
| 平台 | 上手难度 | 适合场景 | 限制 |
| Telegram | ⭐ 最简单 | 个人异步开发 | 需注册 Bot |
| Discord | ⭐⭐ 稍复杂 | 团队共享调试 | 需 Developer Portal |
| iMessage | ⭐⭐ 需授权 | Apple 生态个人用户 | 仅 macOS |

· · ·

|  |
| --- |
| 注意事项 + 总结 使用 Channels 前需要了解的限制 |

有几点需要了解：

**会话必须保持活跃。** 关闭终端即断开连接，Channel 消息将无法送达。可以用 `tmux` 或 `screen` 保持后台运行。

**权限提示会阻断远程操作。** 如果 Claude Code 弹出需要用户确认的权限请求，远程指令会被挂起，直到你在终端手动确认。启用 `--dangerously-skip-permissions` 可绕过，但要谨慎评估风险。

**Preview 阶段只支持官方白名单插件。** 要开发自定义 Channel，需在启动时加 `--channels-dev` 标志。

Channels 和 Remote Control 的差异在于：**Remote Control 给你完整的 claude.ai 界面，Channels 给你轻量异步通道**。前者适合需要上下文的深度交互，后者适合「发个指令让它跑着，过一会儿看结果」的场景。两个功能定位互补，不互相替代。

|  |
| --- |
| 💡 推荐搭配方式  用 **tmux** 保持 Claude Code 会话后台运行 + 接入 Telegram Channel，是目前最轻量的「随时随地发指令」方案。如果你的工作场景偏向团队协作，Discord 是更合适的选择。 |

|  |
| --- |
| Claude Code 的每个细节，这里都有  关注「克劳德猎手」  关注公众号「克劳德猎手」获取更多内容 👇 |