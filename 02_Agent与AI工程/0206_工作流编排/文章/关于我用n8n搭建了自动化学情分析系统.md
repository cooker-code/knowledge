---
title: 关于我用n8n搭建了自动化学情分析系统
author: PiGou Workshop
date: 
url: https://mp.weixin.qq.com/s?__biz=MzA3Mjk3NTcyNw==&mid=2247484576&idx=1&sn=7537a23295d4fb2c8a77733fcb07131f&chksm=9e3108205fd4becc0d6118e3bd203d6c65dbf6ac180d8f323df987f02210af0cf9e36b4d025d&mpshare=1&scene=24&srcid=09167Zx3Aggr1rOfZ5r9vhXi&sharer_shareinfo=99385ab67d26d8b52c58b34862059cf1&sharer_shareinfo_first=99385ab67d26d8b52c58b34862059cf1#rd
---

AI IN EDUCATION ·······

让AI照进教育

WORKFLOW

AUTOMATION

n8n 自动学情分析

一键生成可视化学情分析报告，让教学数据活起来

告别繁琐手工

30秒智能生成

1

· 核心功能：将Excel成绩表自动转换为包含多维度分析、可视化图表、智能建议的综合学情报告，支持年级、班级、学生多层次分析。

· 技术亮点：基于n8n低代码平台，实现从数据清洗、统计计算到报告生成的全流程自动化。

· 效率提升：无需再手动制作班级成绩分析，年级管理者只需一键提交，30秒内就会生成成绩分析报告。

五大核心节点

智能化处理流程

2

节点1：文件读取器- 自动识别并读取Excel文件，支持多种格式，确保数据完整导入。

节点2：数据清洗器- 智能识别缺失值、异常数据，自动进行数据标准化处理。

📝 数据清洗核心代码示例：

```
// 定义列名映射字典（核心）
const COLUMN_MAP = {
 'name': ['姓名'],
 'class': ['班级'],
 'total_score': ['总分分数'],
 'total_grade': ['总分等级'],
 'chinese_score': ['语文分数'],
 'math_score': ['数学分数'],
 // ... 更多学科映射
};
// 自动标准化处理
for (const item of items) {
 const studentData = {};
 for (const originalKey in item.json) {
   const standardKey = reverseMap[originalKey];
   if (standardKey) {
     // 数据清洗：统一处理非数字值
     studentData[standardKey] = processValue(item.json[originalKey]);
   }
 }
 standardizedItems.push({ json: studentData });
}
```

· 节点3：等级评定器- 基于科学的评分标准，自动计算A+、A、B+、B、C+、C六个等级。

DATA

ANALYSIS

核心计算引擎

深度数据挖掘

节点4：核心计算引擎- 工作流的大脑，执行多维度统计分析：

• 自动计算平均分、最高分、最低分、中位数、标准差  
• 生成班级排名、年级排名  
• 识别进步之星、学科冠军、需关注学生  
• 生成智能分析建议和改进方案  
• 自动计算A+、A、B+、B、C+、C六个等级的分布

⚠️ 注意：由于每个学校的数据文件不同，部分代码的接口和引入需要做适配性修改。

📝 核心计算代码片段：

```
// 主要功能函数展示
// 1. 统计计算函数
function calculateStats(scores) {
 const validScores = scores.filter(s => s != null);
 const sorted = [...validScores].sort((a, b) => a - b);
 return {
   avgScore: avg,
   maxScore: Math.max(...validScores),
   minScore: Math.min(...validScores),
   medianScore: sorted[Math.floor(sorted.length / 2)],
   stdDev: stdDev
 };
}
// 2. 查找亮点学生
function findPraiseStudents(classStudents) {
 const praise = {};
 // 班级总分状元
 praise.topScorer = findTopStudent(classStudents);
 // 单科冠军
 praise.subjectChampions = findSubjectChampions(classStudents);
 // 进步之星
 praise.progressStars = findProgressStars(classStudents);
 return praise;
}
// 3. 生成智能分析
function generateAnalysisSummary(classData, gradeStats) {
 const summary = {
   strengths: [],
   weaknesses: [],
   suggestions: []
 };
 // 智能分析逻辑...
 return summary;
}
```

可视化报告生成

一键导出分享

HTML

REPORT

节点5：HTML报告生成器- 生成精美的可视化报告，包含：

• 柱状图、雷达图、折线图等多种图表  
• 班级与年级对比分析  
• 学生搜索筛选功能  
• 一键导出和打印功能  
• 响应式设计，手机端完美适配

报告视图展示

全年级共两个视图：总体视图和学科视图

|  |  |
| --- | --- |
|  |  |

班级视图四大维度分析

1. 总体概况- 可看出等级分布，班级单科第一等

2. 学科分析- 柱状图和雷达图直观展示班级与年级差距，计算不同等级占比

3. 学生详情- 简化数据文件，快速生成班级学生排名

4. 对比分析- 计算班级与年级的具体差距数值，观察各等级率对比

报告亮点功能

亮点表扬自动识别进步之星、学科冠军、全科优秀学生

智能分析AI生成教学改进建议和个性化辅导方案

多维图表柱状图、雷达图、箱线图全方位展示数据

预警提醒自动识别需要重点关注的学生群体

使用价值

1. 节省时间：原本需要数小时的数据分析工作，现在只需30秒

2. 提升质量：标准化的分析流程，确保报告的准确性和全面性

3. 智能建议：基于数据的教学改进建议，助力精准教学

4. 个性化关注：自动识别每个学生的优势和薄弱环节

5. 可复用性：一次配置，永久使用，支持各类考试数据

📌 当前版本为学情分析1.0，后续我们会继续更新工作流程和报告内容，需要工作流的同学，请部署好本地n8n平台，然后私信我json文件哈。代码内容尽可能针对自己的excel文件和学情出发

🤝 欢迎有兴趣的小伙伴加入，为我们的内容提供宝贵建议！

让数据说话，让教学更精准

n8n低代码平台 + AI智能分析

开启教育数字化新篇章