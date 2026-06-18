> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020203_Skill/020203_核心知识点/Skill能力封装与治理边界|Skill能力封装与治理边界]]
---
title: 重磅开源！Anthropic 官方发布 anthropics/skills：Claude 终于学会像人一样操作电脑了！
author: 开源github甄选
date:
url: https://mp.weixin.qq.com/s?__biz=Mzk1NzcyNjUyMg==&mid=2247484762&idx=1&sn=efc6b3201ce3927279c9c64e6cfdbb52&chksm=c222bcaa59a697f87df99ac3b34f3453ff1176113478ffdd60dbea3062cceb1d0d953e30c7b7&mpshare=1&scene=24&srcid=0120SUpNIPXPzOF98KnBYWaP&sharer_shareinfo=c90fcdc698361f2ec9e6b0bbc5862ff3&sharer_shareinfo_first=c90fcdc698361f2ec9e6b0bbc5862ff3#rd
---

AI 圈又地震了！就在大家还在讨论大模型如何写代码更强时，Anthropic 直接祭出了杀手锏——**Computer Use（计算机操作能力）**。

今天我们要深度解析的正是这个能力的官方核心仓库：**anthropics/skills**。这意味着，Claude 不再只是一个躲在对话框里的“聊天机器人”，它已经进化成了能够直接操控你屏幕、移动鼠标、敲击键盘的“虚拟员工”。

---

## 01. anthropics/skills

### 1.1 什么是 anthropics/skills？

简单来说，这个仓库是 Anthropic 为 Claude 3.5 Sonnet 模型量身定制的“技能包”。它包含了一套标准化的工具实现，允许模型通过执行 Python 脚本来模拟人类在计算机上的各种操作。

过去，AI 只能处理文本或生成代码。而现在，通过 `anthropics/skills`，Claude 可以：

* **查看屏幕**

  ：通过截屏分析当前 UI 界面。
* **移动鼠标**

  ：点击按钮、拖动窗口、滚动页面。
* **操作键盘**

  ：输入文本、快捷键组合。
* **执行终端命令**

  ：在环境内直接运行复杂的指令。

### 1.2 核心技术亮点

这个仓库最令人兴奋的地方在于其\*\*“所见即所得”\*\*的交互逻辑。

1. **屏幕感知能力**

   ：Claude 会周期性地获取系统截图，利用其强大的视觉理解能力，定位 UI 元素（如搜索框、确认按钮）的具体坐标。
2. **动作标准化**

   ：仓库中定义的技能（Skills）将复杂的底层操作系统指令封装成简单的 API。例如，`mouse_move` 或 `key_press`。
3. **循环迭代（Agentic Loop）**

   ：Claude 会观察当前状态 -> 决定下一步操作 -> 执行操作 -> 观察结果。这种闭环控制让它能够处理极度复杂的长路径任务。

### 1.3 实际应用场景：你能用它做什么？

有了 `anthropics/skills`，很多繁琐的工作将彻底自动化：

* **自动化软件测试**

  ：只需告诉 Claude “登录我的网站并检查购物车功能是否正常”，它就会像真人一样打开浏览器、输入账号、点击测试。
* **复杂资料整理**

  ：让它跨软件操作，比如从 Excel 读取数据，然后填入 Web 端后台管理系统，最后再发一封邮件汇报。
* **环境配置**

  ：新手最头疼的开发环境搭建，现在可以交给 Claude 直接在你的电脑上敲命令搞定。

### 1.4 如何上手体验？

Anthropic 官方提供了一个预装好所有环境的 Docker 镜像。你只需要获取一个 Claude API Key，运行几行简单的命令，就能在本地浏览器中看到 Claude 正在“接管”一个 Linux 桌面系统。

这个仓库不仅是 AI 开发者的宝库，更是未来 **AI Agent（人工智能智能体）** 时代的标准范式。

---

### 结语

`anthropics/skills` 的开源，标志着大模型从“语言理解”正式迈向了“动作执行”。这不仅仅是技术上的进步，更是生产力方式的颠覆。

如果你是一名开发者，或者对 AI 自动化感兴趣，这个仓库绝对是你今年最不能错过的星标项目。快去 GitHub 给它一个 Star，体验 AI 帮你打工的快乐吧！

**项目地址：** `https://github.com/anthropics/skills`

---

*本文首发于微信公众号开源GitHub甄选，关注我们，获取全球最前沿的 AI 开源项目解析！*

看开心了，点一下，关注一下

---

精选文章：

[Nice！使用开源工具组建一个AI编码小助手](https://mp.weixin.qq.com/s?__biz=Mzk1NzcyNjUyMg==&mid=2247483776&idx=1&sn=8791b4802ab2b8874c4b39cd31415296&scene=21#wechat_redirect)

[YYeTsBot：免费的海量影视资源的宝库](https://mp.weixin.qq.com/s?__biz=Mzk1NzcyNjUyMg==&mid=2247483750&idx=1&sn=192604cc798faca854cecab2e7962704&scene=21#wechat_redirect)

[Open WebUI:53.3 Star的webUi系统](https://mp.weixin.qq.com/s?__biz=Mzk1NzcyNjUyMg==&mid=2247483739&idx=1&sn=43a750b91b994c4b31607511bad59416&scene=21#wechat_redirect)

[探秘51.3K Star的Prompt-Engineering-Guide：解锁 AI 提示工程的宝库](https://mp.weixin.qq.com/s?__biz=Mzk1NzcyNjUyMg==&mid=2247483722&idx=1&sn=ac0d8f46d1bdac62ecedd0f405385ace&scene=21#wechat_redirect)

[终端救星：87.3K Star的TheFuck 工具大揭秘](https://mp.weixin.qq.com/s?__biz=Mzk1NzcyNjUyMg==&mid=2247483694&idx=1&sn=a6b16c94188245ad9d57389069254609&scene=21#wechat_redirect)

[代码魔法棒：66K Star的screenshot-to-code 来袭！](https://mp.weixin.qq.com/s?__biz=Mzk1NzcyNjUyMg==&mid=2247483682&idx=1&sn=40c273ef3b5a02d84a947f87cd6d73de&scene=21#wechat_redirect)