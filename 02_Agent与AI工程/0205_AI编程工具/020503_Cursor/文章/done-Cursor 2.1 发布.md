> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020503_Cursor/020503_核心知识点/Cursor工程使用与上下文边界|Cursor工程使用与上下文边界]]
---
title: Cursor 2.1 发布
author: 字节笔记本
date:
url: https://mp.weixin.qq.com/s?__biz=MzIzMzQyMzUzNw==&mid=2247508691&idx=1&sn=f495d810ca46e76163973afc0b35f892&chksm=e9a3accc60b640c58c2b93ec90661354f7922bc39eff5ad468aab73e78ac5af4393d81c55c02&mpshare=1&scene=24&srcid=1123BKqo9ovDgbWt0CUFivha&sharer_shareinfo=e863782994e935e75d4ed1e863d72fe2&sharer_shareinfo_first=e863782994e935e75d4ed1e863d72fe2#rd
---

Cursor 2.1 正式发布了，这次更新聚焦代码审查、计划制定和搜索环节的效率。实际体验下来代码响应速度提升明显，延迟减少 50% 以上。

以下是它主要的几点更新内容：

**1.编辑器内自动审查代码。**

Agent模式下直接在 Cursor 编辑器中对当前文件或选定代码变更运行 AI 审查，无需切换工具。

AI 会自动扫描潜在 bug、性能瓶颈、可读性优化、安全漏洞等，并以侧边面板形式展示建议和修复代码。点击Fix就可以完成代码的code review了。

不过不太建议频繁使用这个功能，没有必要做过早的优化。

2.Plan Mode大幅改进

Plan Mode下生成多文件修改或重构计划时，Cursor 会主动抛出问题如“这个函数的预期输入输出是什么？”或“优先级如何排序？”，通过人机交互的形式帮助 AI 更精准理解需求。

聊天界面直接内置简单表单，直接点击回答问题，不需要再大段大段地输入规范化的提示词了。

生成的结果是一个完整的文档，支持 ⌘+F 全文搜索计划内容，便于查找和更迭。

3.即时 Grep 搜索

全面转向grep，实现实时无延迟，智能匹配。目前所有代码库搜索基于 grep 命令转为即时响应，无需等待时间。

Grep支持所有模型如 GPT-5、Claude Sonnet 4.5、Composer-1，那结果查找更智能更快，同时支持高级搜索条件，比如内置正则表达式、单词边界匹配、路径过滤等。

4.支持组合模型

支持组合多个模型打配合使用，Agent会自动根据任务 难度进行模型匹配。

虽然目前Cursor的Agent模式功能是越来越强大了，但是在整体交互上做的越来越复杂，多栏布局让整个的编码交互显得特别繁琐，其实完全可以抄Codex的插件交互逻辑，简单又高效。

[收录100+ Gemini Banana Pro生图玩法集合](https://mp.weixin.qq.com/s?__biz=MzIzMzQyMzUzNw==&mid=2247508674&idx=1&sn=a74992f2702d51316338bc58df934163&scene=21#wechat_redirect)

[豆包输入法来了！！](https://mp.weixin.qq.com/s?__biz=MzIzMzQyMzUzNw==&mid=2247508663&idx=1&sn=0abcc45bd520bd098fec3bb730623195&scene=21#wechat_redirect)

[使用Google全线产品制作Nano Banana Pro 玩法集合网站](https://mp.weixin.qq.com/s?__biz=MzIzMzQyMzUzNw==&mid=2247508662&idx=1&sn=ec52cb71881bf9b74a15ec5d1351aeb8&scene=21#wechat_redirect)

[如何二开和部署Gemini 3 生成的内容？](https://mp.weixin.qq.com/s?__biz=MzIzMzQyMzUzNw==&mid=2247508605&idx=1&sn=3a5557eb73c4eadabbb89f52bf4af8fc&payreadticket=HLcMGTrYAEjAcMsCEBGwBGqo2wpmWJ4x8EwdzibJq7-mdWZzI009hpX9BwukPKl4nr5zsHo&scene=21#wechat_redirect)

[使用Gemini 3 Generative UI 生成红楼梦人物关系图谱](https://mp.weixin.qq.com/s?__biz=MzIzMzQyMzUzNw==&mid=2247508531&idx=1&sn=bbd6b16b4cf00f84478ffddc65ffb86a&scene=21#wechat_redirect)