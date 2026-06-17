---
title: 写了个Skill，怎么评测它效果好不好
author: 知识药丸
date: 
url: https://mp.weixin.qq.com/s?__biz=MzIwMTM5MTM1NA==&mid=2649477500&idx=1&sn=c73116f9f1efb394aa1937bab38c6434&chksm=8f1daee4e55dc1cf111c35ace6fda5279260b9b21ad9d2d590db0368826bf2a85737f1c9808e&mpshare=1&scene=24&srcid=0326XOGdR9OHBKZWPgbscgkV&sharer_shareinfo=a4acf8882d87ff9e2776d70bc32c1254&sharer_shareinfo_first=a4acf8882d87ff9e2776d70bc32c1254#rd
---

🌟星标 + 👆关注，第一时间知道最新、最有用的AI编程姿势

[《贾杰的AI编程秘籍》](https://mp.weixin.qq.com/s?__biz=MzIwMTM5MTM1NA==&mid=2649474437&idx=1&sn=4ae1516afc0ec71b7638dd79daaae2cb&scene=21#wechat_redirect)付费合集，共10篇，现已完结。30元交个朋友，学不到真东西找我退钱；）

 

以及[我开发的手机编程App，已经冲到了付费榜第一，欢迎围观](https://mp.weixin.qq.com/s?__biz=MzIwMTM5MTM1NA==&mid=2649477442&idx=1&sn=165edcddec85ffaff9db02890f9d5e55&scene=21#wechat_redirect)

---

### 写在前面

我们写了个看起来“能跑”的 Skill，但它到底稳不稳定、边界会不会塌、是不是比“什么都不加的裸模型”更强，这些都很容易被错觉迷惑。

评测（eval）这事就像给 Skill 做体检和碰撞测试，痛一点，但真有用。

### 我们要测什么，怎么测

“可用性”和“可靠性”是两件事，可用性是能不能出活，可靠性是各种奇怪输入来一遍还能不能稳定出活。

评测的基本单位是测试用例，它有三件套：一个真实点的用户 Prompt，一个用人话写清楚的期望结果，还有可选的输入文件（比如 CSV、图片之类）。

最小可行方案很简单，先写 2～3 个用例就好（别一上来重金投入），把它们放进你的 Skill 目录的 evals/evals.json 里，然后先跑一轮看看会吐出什么鬼。

### 测试用例怎么写，别“自说自话”

定义先来一句：好用例=真实+多样+带边界+可验证。

真实，是用户会说的话，会提到文件路径、字段名、场景背景，不是“帮我处理这个数据”这种含糊其辞。

多样，是同一个目标用不同语气和细节表达，既有“随便帮我清理下”的大白话，也有“对 data/input.csv 做 A→B→C”的精准指令。

边界，是至少放一个“刁钻”的例子，比如字段缺失、文件格式半坏、需求本身容易歧义。

可验证，是期望别写“要有条理、要优雅”这种不可落地的东西，先写成“导出 PNG 柱状图，坐标轴有标签，标题提到 Revenue”这种能落地核验的描述。

### 一个最小用例示范（定义之后立刻上例子）

下面这段就是官方推荐结构，我直接照抄改了些词（引自 agentskills.io）：

```
    {  
  "skill_name": "csv-analyzer",  
  "evals": [  
    {  
      "id": 1,  
      "prompt": "I have a CSV of monthly sales data in data/sales_2025.csv. Can you find the top 3 months by revenue and make a bar chart?",  
      "expected_output": "A bar chart image showing the top 3 months by revenue, with labeled axes and values.",  
      "files": ["evals/files/sales_2025.csv"]  
    }  
  ]  
}
```

它不是在定义“怎么判分”，而是在声明“成功长啥样”，判分我们之后用断言来做。

### 跑评测：有 Skill 一次，没 Skill 再一次

核心玩法很朴素：每个用例跑两遍，一遍带 Skill，一遍不带 Skill（或者带旧版本 Skill），这样就有了“增量”的可比性。

每次运行都要“干净启动”，尽量隔离任何先验状态（换会话，或者用支持子代理的环境，比如 Claude Code 的 subagent，每个子任务都是全新上下文）。

执行时把 Skill 路径、用例 Prompt、输入文件和输出目录都说清楚，像这样（引自 agentskills.io）：

```
    - Skill path: /path/to/csv-analyzer  
- Task: I have a CSV of monthly sales data in data/sales_2025.csv. Can you find the top 3 months by revenue and make a bar chart?  
- Input files: evals/files/sales_2025.csv  
- Save outputs to: csv-analyzer-workspace/iteration-1/eval-top-months-chart/with_skill/outputs/
```

基线那次就把 Skill path 去掉，输出改到 without\_skill/outputs/ 即可，做旧版对比的话用快照当基线存到 old\_skill/outputs/。

### 目录怎么摆，别把东西塞一团

建议把评测结果放在一个独立 workspace 里，每次完整循环一个 iteration-N，然后每个用例一个 eval 目录，再分 with\_skill 和 without\_skill 两边输出，跑完会有 outputs、timing.json、grading.json，最外层有个 benchmark.json 汇总统计（结构见原文示意）。

这套结构带来的爽感是可复现、可对比、可回溯（而不是“咦我上次改了啥来着”）。

### 计时与算账：时间和 Token 都是钱

一次运行结束就把 total\_tokens 和 duration\_ms 记下来，别拖到后面再翻日志（很多环境不会帮你存档）。

示例长这样（引自 agentskills.io）：

```
    { "total_tokens": 84852, "duration_ms": 23332 }
```

结论很现实：更好的输出如果贵了三倍 Token，它是不是值得，要结合业务场景算账。

### 断言：让“好不好”变成“对不对”

断言是可验证的“应该满足的事实”，比如“输出文件是合法 JSON”“图表有坐标轴标签”“报告至少包含 3 条建议”。

弱断言会让你自欺欺人，比如“输出是好的”，或者“必须出现固定短语”，前者不可判，后者太脆弱。

把断言补回 evals.json，像这样（引自 agentskills.io）：

```
    {  
  "assertions": [  
    "The output includes a bar chart image file",  
    "The chart shows exactly 3 months",  
    "Both axes are labeled",  
    "The chart title or caption mentions revenue"  
  ]  
}
```

注意顺序是先跑一轮看看产物，再补断言，因为“好”的样子往往得看一次真输出才敢落笔。

### 打分：要证据，不要“主观感觉良好”

对每条断言做 PASS/FAIL，并给出确凿证据，最好引用输出里的具体片段或文件清单。

能写脚本核验的尽量脚本化（比如文件是否存在、维度是否正确、JSON 是否可 parse），脚本比 LLM 判分在这些机械问题上更靠谱，也更可复用。

评分记录可以是这样（引自 agentskills.io）：

```
    {  
  "assertion_results": [  
    { "text": "The output includes a bar chart image file", "passed": true, "evidence": "Found chart.png (45KB) in outputs directory" },  
    { "text": "The chart shows exactly 3 months", "passed": true, "evidence": "Chart displays bars for March, July, and November" },  
    { "text": "Both axes are labeled", "passed": false, "evidence": "Y-axis is labeled 'Revenue ($)' but X-axis has no label" },  
    { "text": "The chart title or caption mentions revenue", "passed": true, "evidence": "Chart title reads 'Top 3 Months by Revenue'" }  
  ],  
  "summary": { "passed": 3, "failed": 1, "total": 4, "pass_rate": 0.75 }  
}
```

打分的原则很“严格但公平”，有标签但没内容算失败，该挑剔就挑剔。

### 聚合：把分数变成决策

所有用例评分完成后，做一份 benchmark.json，统计 with\_skill 和 without\_skill 的平均通过率、时间和 Token，并给出 delta。

示意数据长这样（引自 agentskills.io）：

```
    {  
  "run_summary": {  
    "with_skill": { "pass_rate": { "mean": 0.83, "stddev": 0.06 }, "time_seconds": { "mean": 45.0, "stddev": 12.0 }, "tokens": { "mean": 3800, "stddev": 400 } },  
    "without_skill": { "pass_rate": { "mean": 0.33, "stddev": 0.10 }, "time_seconds": { "mean": 32.0, "stddev": 8.0 }, "tokens": { "mean": 2100, "stddev": 300 } },  
    "delta": { "pass_rate": 0.50, "time_seconds": 13.0, "tokens": 1700 }  
  }  
}
```

这张“成本—收益”对照表很关键，它告诉我们这项 Skill 的性价比到底咋样。

### 模式分析：删“假功劳”，修“真顽疾”，放大“亮点”

有三件事我觉得特别有用。

第一，删掉在两边都“总是通过”的断言，它们只是帮你虚高了通过率，却没有信息增量。

第二，优先排查两边都“总是失败”的断言，要么断言写坏了，要么题太难，要么检查的点不对路，尽快修正。

第三，找出“带 Skill 才通过”的断言，这里就是 Skill 真正创造价值的地方，回溯一下是哪条指令或哪个脚本起了决定性作用，然后把它做厚做稳。

如果某个用例波动很大（stddev 高），往往是指令有歧义或任务太依赖随机性，给更多示例、约束格式、减少自由度，能明显降低抖动。

还有个好招叫“盲比对”，把两个版本的输出在不标注来源的情况下交给 LLM 做整体评分（组织、可用性、排版、打磨度），它和断言互补，能抓到那些“过了断言但整体质感差距明显”的情况。

### 人类复盘：机器判分之外的“味道”

有些东西机器很难判，比如写作风格、可读性、视觉层次、是否“像人写给人看的”，这些最好来一轮人工走查。

我的实践里会把“断言评分”和“人工短评”并排放在同一条用例下，下一轮改进就有明确抓手。

### 一套最小可行循环，我现在就会用它

第一步，写 2～3 个真实且多样的用例，准备好必要文件，先不写断言。

第二步，干净上下文下跑一遍带 Skill，再跑一遍不带 Skill，存储输出、时间、Token。

第三步，基于产物补上断言，再跑一次评分，输出 grading.json。

第四步，聚合统计到 benchmark.json，读 delta 做一次“值不值”的判断。

第五步，按“删易过、救常败、放大增量”的顺序优化 Skill 指令或脚本，再来下一轮迭代。

第六步，每 2～3 轮做一次盲比对和人工复盘，防止“KPI 式过断言”。

### 总结

评测不是添麻烦，而是省时间的“早诊断”，它帮我们把“感觉还行”变成“可复现的稳”，把“可能更好”变成“数据支持的更好”。

当我们把“写 Skill”变成“写—测—改—再测”的小循环，输出质量会上一个非常踏实的台阶（而且可向老板汇报，嘿嘿）。

### 参考资料

Evaluating skill output quality（摘自 agentskills.io）： https://agentskills.io/skill-creation/evaluating-skills

Documentation Index（摘自 agentskills.io）： https://agentskills.io/llms.txt

---

 

 

坚持创作不易，求个一键三连，谢谢你～❤️

以及「AI Coding技术交流群」，联系 ayqywx 我拉你进群，共同交流学习～

订阅链接 https://note.mowen.cn/detail/OLPEp7HzeB0EXJOLe7mM4