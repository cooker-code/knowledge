> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020203_Skill/020203_核心知识点/Skill能力封装与治理边界|Skill能力封装与治理边界]]
---
title: Claude Skills 新玩法：用 skill-creator 10 分钟搞定 Excel 报表自动化，职场人必学
author: 海老豹666
date:
url: https://mp.weixin.qq.com/s/NHMuneNRFmFvWfEDPkvV1g
---

#

# 1.前言

skill‑creator 是 Anthropic 在 Claude Skills 体系中提供的“元技能”。它本身是一个可直接在 Claude 对话中调用的 Skill，专门用于 **帮助用户快速创建、编辑、打包其他自定义 Skill**，从而让 Claude 能够在特定业务场景下拥有专业化的能力。在产品发布时，Anthropic 将其定位为“一键生成 Skill 的交互式向导”，用户只需用自然语言描述需求，skill‑creator 会引导完成文件结构、`SKILL.md` 编写、资源打包等全部步骤。

我们可以借助skill‑creator 帮我们生成新的skils技能。那么为什么选择 Skill Creator? 它能拥有强大的元技能系统，帮助你快速构建专业的 Claude 技能扩展

技术架构特点

我们可以通过六个步骤很容易创建一个新的Skill

上期给大家介绍过关于Skills制作，当时没有使用skill‑creator 来实现，而且实现的也较为简单，对上期不了解的小伙伴可以看我之前的文章《[Claude Skills 从零到一：手把手打造专属公众号文风生成器，10 分钟搞定 AI 技能定制](https://mp.weixin.qq.com/s?__biz=Mzg3OTYzMjc1NQ==&mid=2247490164&idx=1&sn=fffc46e60ec9eb411678818648a8854c&scene=21#wechat_redirect)》

本期给大家介绍如何使用skill‑creator创建一个Excel 报表自动生成的 Skill，这个skill 很多职场人士应该用的会比较多。那么话不多说下面教大家如何实现吧 。

# 2.Claude  Skills制作

## 创建skills

提示词如下

```
帮我用 skill‑creator 创建一个用于 Excel 报表自动生成的 Skill
```

核心是让AI 调用skill‑creator

中间AI 会要求我们填写一些个人技能要求：

还是项目技能（当前项目共享）？

Excel 报表主要用于什么场景？

你希望使用哪个 Python Excel 库？

我们填写上面的信息：个人技能（~/.claude/skills/）、数据分析报表, 业务报告, 数据导出, 模板填充、pandas + openpyxl

接下来AI 帮我们生成

接下来AI 帮我们在生成skill里面用到python代码

后面再等待几分钟后 AI 帮我生成了基于excle表格的skill

包含这个excle表格的技能

```
 包含 11 个详细章节：
  1. pandas Excel 操作
  2. openpyxl 核心 API
  3. 样式和格式化（字体、填充、边框、对齐）
  4. 图表 API（柱状图、折线图、饼图、散点图）
  5. 条件格式（色阶、数据条、图标集）
  6. 高级功能（数据验证、表格、保护）
  7. 性能优化技巧

  示例脚本特点

  basic_report.py:
  - 基础报表生成流程
  - 标准样式应用
  - 列宽自动调整
  - 冻结窗格和筛选

  advanced_report.py:
  - 多工作表管理
  - 数据透视表
  - 各类图表创建
  - 条件格式应用
  - 公式自动计算
  - 仪表板生成

  template_fill.py:
  - 模板加载和填充
  - 命名范围使用
  - 批量生成报告
  - 发票生成示例

  quick_reference.py:
  - 500+ 行代码片段集合
  - 覆盖所有常用操作
  - 即复制即用
  - 包含完整示例模板
```

包括如何使用也给我们举例写好了

```
1. 测试 Skill: 在 Claude Code 中问："帮我创建一个简单的 Excel 销售报表"
```

## skills上传打包

AI 在创建这个skills 直接帮我生成到我的claude skills中，我都不需要复制了。

我们进入/root/.claude/skills 目录下

```
cd ~/.claude/skills
```

查看生成好的skills  有哪些内容。

代码

非常完美 一行命令就帮我制作好了一个 Excel 报表自动生成的 Skill

# 3.验证及测试

接下来我们验证一下这个skills 作出的表格是什么样子的。

我们这里为了演示，我让 AI 帮我造成一个表格数据。

```
    请帮我造一个"简单的初中生成期中考试学习成绩表格" 其中包括语文、数学、英语、物理、化学、历史、生物、地理、历史科目的。学生给我生成大概50个人。学生的姓名和成绩随机生成
    生成"2025年101中学八年级期中考试成绩表.xlsx"
```

claude code 会基于我刚才的需求生成python代码，使用这个python代码来生成这个初中成绩表格。

生成完成，我们用电脑打开表格看看

数据表格已经生成了，接下来我们使用刚才创建的excel表格skills  来生成我们要的统计图表。

我的提示词如下：

```
请基于“2025年101中学八年级期中考试成绩表.xlsx”表格使用excel-report-generator 这个skills 技能帮我生成基于“语文、数学、英语、物理、化学、历史、生物、地理、历史科目评价成绩在90分以上、80分-80分、70分以下人员统计占比图” 输出“2025年101中学八年级期中考试成绩表各学科占比统计图.xlsx”
```

生成的结果如下：

结果AI 非常贴心的帮我生成了下面的数据

我们通过上面的自定义创建excel-report-generator 基于excel表格快速生成一个我们要的各种图表，呵呵是不是非常的方便？

视频效果

skills已经开源

https://github.com/wwwzhouhui/skills\_collection

感兴趣小伙伴可以去下载。

# 4.总结

今天主要带大家了解并实践了使用 skill-creator 工具创建 Excel 报表自动生成专属 Claude Skill 的完整流程，该流程以 Anthropic 推出的 Skills 模块化体系为基础，通过借助 AI 辅助生成工具，形成了一套从技能定制到实际报表生成的高效解决方案。

通过这套实践方案，用户能够快速构建专业的 Excel 自动化能力 —— 借助 skill-creator 引导的交互式配置（场景定义、库选择、功能生成），无需复杂的编程经验，就能完成包含 11 个专业章节、4 类实用脚本的 Excel 报表技能封装。无论是数据分析报表生成、业务报告格式化，还是模板批量填充、图表自动绘制，都能通过标准化的 Skill 调用实现，极大降低了职场人士利用 AI 提升 Excel 办公效率的技术门槛。在实际验证中，我们创建的「excel-report-generator」能够稳定处理成绩数据统计、多维度图表生成等任务，有效解决了手动制作报表时的繁琐重复问题。同时，该方案具备很强的扩展性 —— 小伙伴们可以基于此技能扩展更多办公场景应用，如财务报表自动汇总、销售数据可视化看板、库存清单批量生成等，进一步释放 Claude 模型在办公自动化领域的应用价值。

感兴趣的小伙伴可以参照文中提供的步骤，结合自身办公需求创建专属 Excel 技能进行实践。今天的分享就到这里结束了，我们下一篇文章见。

[Claude Skills 从零到一：手把手打造专属公众号文风生成器，10 分钟搞定 AI 技能定制](https://mp.weixin.qq.com/s?__biz=Mzg3OTYzMjc1NQ==&mid=2247490164&idx=1&sn=fffc46e60ec9eb411678818648a8854c&scene=21#wechat_redirect)

[Claude Skills 实战指南：3 分钟搞定 PPT、海报与 Logo，AI 办公效率翻倍！](https://mp.weixin.qq.com/s?__biz=Mzg3OTYzMjc1NQ==&mid=2247490131&idx=1&sn=cac1a6528ff47402333b78779886e557&scene=21#wechat_redirect)

[Claude+Codex协同开发，让AI编程效率翻倍成本直降近半](https://mp.weixin.qq.com/s?__biz=Mzg3OTYzMjc1NQ==&mid=2247490097&idx=1&sn=8b861e57ecaae7dc16dc2b0794b12846&scene=21#wechat_redirect)

[AI 驱动教学革命：3 分钟生成专业级动画课件，还能导出视频 GIF！](https://mp.weixin.qq.com/s?__biz=Mzg3OTYzMjc1NQ==&mid=2247490058&idx=1&sn=6cb8201d1450a3cdbd62911ca23c844c&scene=21#wechat_redirect)

[dify案例分享-用 Dify 一键生成教学动画 HTML！AI 助力，3 分钟搞定专业级课件](https://mp.weixin.qq.com/s?__biz=Mzg3OTYzMjc1NQ==&mid=2247490031&idx=1&sn=b5feaf42dbbfa5bf4ca5c3181d0d4225&scene=21#wechat_redirect)