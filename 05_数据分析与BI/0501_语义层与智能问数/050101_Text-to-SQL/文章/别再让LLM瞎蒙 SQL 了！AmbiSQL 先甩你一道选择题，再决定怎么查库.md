---
title: 别再让LLM瞎蒙 SQL 了！AmbiSQL 先甩你一道选择题，再决定怎么查库
author: Paper易论
date: 
url: https://mp.weixin.qq.com/s?__biz=MzIzOTAxMjk4Ng==&mid=2454621591&idx=1&sn=41676ce0d97534f403f3cf2f6970ea6b&chksm=ffd5714f29fc3aaacc7edbe61e5cedcbfb516a538e881aa9b7ecf9fc0a156d14d064b77ac7a5&mpshare=1&scene=24&srcid=1002ZIEn4uQwdu2O34y5OW2Q&sharer_shareinfo=95eeaef8ea76042628d1296b60349500&sharer_shareinfo_first=95eeaef8ea76042628d1296b60349500#rd
---

AmbiSQL 就是这么一个“问清楚再动手”的插件，挂在 Text-to-SQL  pipeline 前面，像地铁安检员一样，先把乘客口袋里的 ambiguity 掏出来，让你自己认领，再走闸机。整套流程没花里胡哨的动画，就两块面板：  
① 嫌疑短语雷达 → ② 多选题式审讯。

图1：AmbiSQL系统概述

图注：AmbiSQL 两阶段流水线，灰底那块就是审讯室

作者们先给 ambiguity 搞了个“二维坐标”：  
横轴是 DB-related——你嘴里的“最大城市”到底指面积还是人口？纵轴是 LLM-related——“越战结束”你要的是 1975-04-30 当天还是整年？  
坐标一画，之前那些拍脑袋的“模糊”瞬间有了工牌，方便后面 LLM 点名。

点名用的是 GPT-4o，zero-shot， prompt 里塞了 8 个典型“案底”，比如“ranked 2”到底映射到 driverStandings.position 还是 results.position。模型看完案底就能在新句子里把同款嫌疑短语揪出来，precision 87.2%，recall 89.1%，比不少全职标注员都稳。

揪出来之后不直接改 SQL，而是弹多选题让用户拍板。选项怎么来的？DB-related 的把相关列的 schema snippet 贴出来，LLM-related 的就把维基百科或数据库里能查到的常识写两行，省得用户再去 Google。

图2：用户偏好树

Stage 2 允许用户临时加料，比如“我还只要德国车手”。系统把附加条件写进 WHERE，再跑一遍 ambiguity 检测，防止新加的词又埋雷。全部澄清完，才把洗净的句子递给下游的 XiYan-SQL。整套动作像 Git rebase：先解决冲突，再正式 commit。

实验部分作者挑了 BIRD 和 TAG 里 40 条“钓鱼题”——单看字面怎么解释都对，但真执行只有一条能对上 ground truth。裸跑 XiYan-SQL 只有 42.5% 一次就写对，挂上 AmbiSQL 后直接飙到 92.5%；TAG 那组更夸张，从 20% 干到 85%。表格我原封不动搬过来，数字自己看：

如表所示，Exact Match Accuracy，两栏对比，左边没挂插件，右边挂了，绿得发慌

有人可能会杠：多选题要是也给不出我想要的说法呢？作者留了后门——用户能在辅助输入框里直接写自然语言补充，系统一样会把这句话 embedding 进去重写查询，不会把话堵死。

再补一句工程细节：整个插件只调 LLM 不做微调，也就几行 prompt 工程，迁移到新库零成本；前后端分离，前端用 Vue，后端就是 OpenAI API + 轻量缓存，Docker 一把梭。代码扔在 GitHub，配了 5 个现成的 SQLite 样例库，读者下班就能拉下来玩。

图3：AmbiSQL的用户界面

如图，UI 截图，三栏布局：左输入，中审讯，右 SQL 输出，按钮 Compare 一点，对错高亮，爽点满满

看完只想说：Text-to-SQL 卷到 2025 年，早就不是“谁能写更长的 prompt”而是“谁敢先开口问用户”。AmbiSQL 先把“问”这一步做成标准化组件，以后 benchmark 要是再拿“单答案”当真理，直接甩这工具上去，一秒给你生成 N 条合理 SQL，看评测还怎么打负分。  
哪天老板再吐槽“这报表数字不对”，你也能甩锅甩得优雅：“早就让你点过选择题了，这锅用户背好。”

代码发布在github, 地址为:

https://github.com/JustinzjDing/AmbiSQL