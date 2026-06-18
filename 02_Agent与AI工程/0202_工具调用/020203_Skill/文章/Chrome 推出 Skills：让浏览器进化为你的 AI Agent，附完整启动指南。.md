---
title: Chrome 推出 Skills：让浏览器进化为你的 AI Agent，附完整启动指南。
author: 王胖纸
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI1MzU4ODkzMA==&mid=2247485474&idx=1&sn=a0ce9dd47a04baf661154c891c00f55f&chksm=e84c99c79bae9ed05437fee0fd40bbd9be057dd52a259006a2ede34dc32ffc8113c112f4ba0f&mpshare=1&scene=24&srcid=0416afy5M61vlaTkW09NL79w&sharer_shareinfo=abb0212c0486d0d2bd5ddab531fc5380&sharer_shareinfo_first=abb0212c0486d0d2bd5ddab531fc5380#rd
---

大家好，我是王胖纸，Atlasnote的独立开发者。

最近，Chrome 在实验性功能中上线了 **“Skills（技能）”**。这不仅仅是一个搜索增强包，它是 Chrome 迈向 **Browser Use（浏览器自动化代理）** 的关键一步。通过这些内置技能，Chrome 终于能听懂人话，并直接帮你“干活”了。

---

# 什么是 Chrome Skills？

简单来说，**Skills 是 Chrome 内置的一系列“行动插件”**。

在过去，AI 只能在侧边栏陪你聊天（如 Gemini）；而有了 Skills 以后，AI 拥有了操作浏览器的“手”。它将浏览器的各项底层功能，比如书签管理、历史记录搜索、标签组整理，甚至自动填写表单，拆解为一个个独立的 **Skill（技能）**。

当你在地址栏输入需求时，Chrome 会自动调用相应的技能去执行任务。比如：

• **搜索增强技能**：不再只是关键词匹配，而是理解语意后从上万条历史记录中精准定位。

• **管理型技能**：一键“把所有关于旅行的标签页归类并命名”。

• **内容创作技能**：在任何文本框内通过内置模型进行重写或润色。

# 如何开启？（只需三步）

目前该功能仍处于实验阶段，跟随以下步骤即可抢先体验：

# 第一步：升级 Chrome

确保你的 Chrome 浏览器已更新至最新版本。

地址栏输入：`chrome://settings/help`，立即检查并更新版本。

# 第二步：开启实验性 Flag

1. 在地址栏输入 `chrome://flags`。

2. 在搜索框输入 **skills**。

3. 找到相关的 Skills 选项，比如 `Enables optimization guide skills`，将其状态改为 **Enabled**。

4. 点击右下角的 **Relaunch**，重启浏览器。

# 第三步：访问技能管理页

在地址栏输入 `chrome://skills/browse`（或在设置中的“AI 创新”选项卡中查看），你将看到目前浏览器已加载的所有 AI 技能清单及其实时状态。

# 核心功能详解：它能帮你做什么？

开启 Skills 后，你会发现 Chrome 变得“聪明”得有些陌生：

# 1. 语义化历史搜索（AI History Search）

不再需要记得具体的网址。你可以直接问：“我上周看的那篇关于低碳饮食的文章在哪？”AI 技能会检索页面内容，而不只是标题，瞬间帮你找回记忆。

# 2. 标签页自动架构师（Tab Organizer）

当你打开了 50 个网页感到头大时，调用 “Organize Tabs” 技能。它会自动分析页面主题，把“工作”“购物”“新闻”自动归类进带颜色的标签组，强迫症患者的救星。

# 3. 全局文本重写（Help me write）

这是最实用的技能之一。在任何网页的输入框，比如邮件、论坛、评论区，点右键后，AI 会调用内置的 **Gemini Nano** 模型帮你扩写、缩写或变换语气。

# 4. 预测性操作提示

基于 Skills 架构，Chrome 会在地址栏下方根据你的习惯弹出“技能建议”。比如你刚预定了酒店，它可能会主动提示：“是否查看周边的天气技能？”

# 深度指南：如何玩转你的“Agent”

想要让 Chrome 真正像个 Agent 一样工作，建议养成以下习惯：

• **善用地址栏（Omnibox）**：试着在地址栏直接输入一个@+触发词+指令，比如“@history 找回昨天关于 AI 的 PPT”，观察它是否能直接触发特定的 Skill 响应。

• **关注本地性能**：大部分 Skills 基于本地模型（如 Gemini Nano）运行。如果发现响应慢，可以在 `chrome://components` 中检查 **Optimization Guide On Device Model** 是否已下载完成。

• **隐私掌控**：在 `chrome://settings/ai` 中，你可以随时开关特定的技能。所有的历史记录语义化处理均在本地完成，无需担心浏览轨迹被上传云端。

# 结语

Chrome Skills 的出现，标志着浏览器正从“被动显示工具”向“主动执行代理（Agent）”转变。

虽然目前很多技能还在迭代中，但这种“开箱即用”的 AI 能力，无疑大大降低了普通用户使用 AI Agent 的门槛。如果你也是效率工具控，这波“尝鲜”绝对不亏。

**快去重启你的 Chrome，释放它的 AI 潜能吧！**

如果你对Atlasnote 感兴趣，可以到官网：atlasnote.ai 下载免费试用。也可以参考这篇Atlasnote使用指南。

[AtlasNote 完整使用指南 v1.0：从零构建你的专属知识地图](https://mp.weixin.qq.com/s?__biz=MzI1MzU4ODkzMA==&mid=2247484685&idx=1&sn=7e38b82208c368d69d622228e7aea1f4&scene=21#wechat_redirect)

如果你是一个AI新手，想要进阶了解更多AI知识，可以了解一下我的AI新手进阶65课：

[全网最贴心AI教程：注册、付费、Prompt，使用指南、私藏工具，Agent....你想要的都在这里了](https://mp.weixin.qq.com/s?__biz=MzI1MzU4ODkzMA==&mid=2247484879&idx=1&sn=8221c6f9a748ada9ed9632edddf112bb&scene=21#wechat_redirect)