> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: 给Claude Code装一个"视觉外挂"
author: 词元视界AI
date: 词元视界词元视界
url: https://mp.weixin.qq.com/s?__biz=MzY4MjMyODU3MQ==&mid=2247483990&idx=1&sn=85bdd3077f479d6563571df82eb8d15f&chksm=f299182f7c87983526f9492b4af7bd94b8aeae57d4f967bbd31f42107f8622362aba1faba10a&mpshare=1&scene=24&srcid=0603YZzwJ3k9HNVFgTTID1N3&sharer_shareinfo=55d1af0804f038f39aca78b4b28f7dfd&sharer_shareinfo_first=55d1af0804f038f39aca78b4b28f7dfd#rd
---

词元视界 · 实战教程 · 第 03 期

2026-06-02

上一期我们聊了 Claude Code 的四种玩法，但留下一个遗憾：**如果你按第一期教程接了 DeepSeek/Qwen/GLM/Kimi，Ctrl+V 粘贴的截图只显示文件名，模型不认。（上图为Claude Code 成功接入DeepSeek V4）**

原因很简单：这些模型是纯文本模型，天生不会看图。但今天有一个方案，能让它们"看懂"图片——**不加钱换模型，加一个"视觉翻译官"就行**。

一、思路：一个"翻译官"解决一切

方案叫 **claude-vision-skill**（GitHub 开源），核心思路极其简洁：

你贴一张图
⬇
vision.js 读取图片 → 转 base64
⬇
发给阿里云 Qwen-VL（视觉模型）
⬇
Qwen-VL 返回：「画面包含一个登录框，左上角有红色错误提示……"
⬇
DeepSeek/V4 拿到这段文字，当"输入"来回答你

**不是让 DeepSeek 直接看图，而是用一个会看图的模型帮它把图片"翻译"成文字。**DeepSeek 看到的是文字描述，所以它能处理——就像你不会读盲文，但有人帮你把盲文翻译成中文读给你听。

二、申请视觉 API Key（2 分钟搞定）

你需要一个能看图的视觉模型 API。推荐阿里云百炼：**新用户 100 万 Token 免费，约 0.02 元/次**。

**步骤：**

① 打开 bailian.console.aliyun.com，用支付宝登录
② 左侧菜单 → **模型广场** → 搜索「qwen-vl-max」→ **开通**
③ 右上角头像 → **API Key** → 创建新的 Key → 复制保存

**⚠️ 注意：**Key 只显示一次，复制后存好。国内访问阿里云完全不需要科学上网。

三、部署视觉外挂

**第二步：配置 API Key**

**第三步：把 CLAUDE.md 规则写入项目**

下载的文件里有一个 **CLAUDE.md**，把它**合并到你的项目根目录的 CLAUDE.md 里**（没有就新建一个）。内容很简单，就是告诉 Claude Code：以后遇到图片，用 vision.js 去识别：

就这么简单。接下来 Claude Code 遇到图片就会自动调用 vision.js。

四、验证——拍一张桌面试试

配好之后，最简单的测试方式：

# 截一张报错图，Mac 上用 Cmd+Shift+4，存到桌面

# 打开 Claude Code，把图片文件拖进终端

# 说一句：
"这张图里的报错是怎么回事"

如果一切正常，你会看到流程：

① Claude Code 检测到图片 → 不会直接发给 DeepSeek
② 自动调用 node vision.js
③ vision.js 把图片发给阿里云 Qwen-VL
④ Qwen-VL 返回文字描述
⑤ DeepSeek 拿到描述，给你回答

这里博主截图了一张财经软件上的截图：

然后启动Claude后让他帮我分析一下这张图片：

然后Claude code就会帮你自动识别内容提取出信息：

💡 Mac 快速截图：Cmd+Shift+4 → 框选区域截图 → 自动存到桌面 → 把文件拖进 Claude Code 终端。或者直接微信截图，然后control +v粘贴到Claude Code

五、用其他视觉模型（如果你不想用阿里云）

vision.js 不绑定阿里云，它走的是 OpenAI 兼容格式，换上任何视觉模型的 API 地址都行：

|  |  |  |  |
| --- | --- | --- | --- |
| 服务商 | 模型 | 需翻墙 | 价格 |
| 阿里云百炼 | qwen-vl-max | 不需要 | ~0.02 元/次 |
| OpenAI | gpt-4o-mini | 需要 | ~0.03 元/次 |
| 其他 | 任何兼容 OpenAI 格式的视觉模型 | 取决于服务商 | 取决于服务商 |

想换模型？改 vision.js 里的 **BASE\_URL** 和 **MODEL** 就行，不需要改任何其他代码。

六、完整配置清单（对照检查）

|  |  |  |
| --- | --- | --- |
| ✓ | 检查项 | 不通过怎么办 |
| ☐ | vision.js 在项目目录下 | 从 GitHub 下载后拷贝过去 |
| ☐ | vision.js 里填了 API Key（不是 sk-xxx） | 去阿里云百炼创建 Key |
| ☐ | vision.js 里 MODEL 填了 qwen-vl-max | 修改第25行的 "xxx" → "qwen-vl-max" |
| ☐ | 项目根目录有 CLAUDE.md（含识图规则） | 把下载的 CLAUDE.md 内容合并进去 |
| ☐ | 拖一张截图进 Claude Code 能正常响应 | 检查 Node.js 版本 ≥ 16；检查 .env 或环境变量 |

七、花了多少钱？一次花多少？

以阿里云 Qwen-VL-Max 为例：

✅ 新用户：**100 万 Token 免费**，按每张图 2000 Token 算大约 500 张图
✅ 超出后：约 **0.02 元/次**，一天用 20 次不到 5 毛钱
✅ 只在你贴图片的时候计费，回答问题不额外计费（视觉只负责"翻译"那一瞬间）

相当于花 5 毛钱给 DeepSeek 装了一个"眼睛"，这笔账划得来。

八、总结：不会看图的模型，可以配一个会看图的助手

这个方案的精妙之处在于——**它不改你的主模型，只在旁边站一个"翻译官"**。你的 DeepSeek/通义千问/GLM 继续安静地写代码，遇到图片时，视觉外挂悄悄介入，把画面翻译成文字，然后退场。

三篇文章走下来，从安装到配置到视觉外挂，你的 Claude Code 已经从"一个聊天框"变成了"一个能看图、能重构、能审查的 AI 开发搭档"。

「模型不会看图不是问题
不会给模型装眼睛才是」

—— 词元视界AI

🔄 转发给那个还在手动截图切窗口的朋友

🔜 下期预告：Claude Code Skills —— 安装专属技能包

📌 词元视界 · 一天一篇，从安装到精通