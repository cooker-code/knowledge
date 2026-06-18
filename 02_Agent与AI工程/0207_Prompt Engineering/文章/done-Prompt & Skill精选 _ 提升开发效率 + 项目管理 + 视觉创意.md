> 已吸收至：[[02_Agent与AI工程/0207_Prompt Engineering/0207_核心知识点/Prompt任务契约与评估闭环|Prompt任务契约与评估闭环]]
---
title: Prompt & Skill精选 | 提升开发效率 + 项目管理 + 视觉创意
author: 深港数码
date:
url: https://mp.weixin.qq.com/s?__biz=MzUyNTc2OTU0Ng==&mid=2247487496&idx=5&sn=bd2c99c3a6d7011228a8b387720f03bc&chksm=fb17cd9801606bb0b6c18146e3cdf51116d02bc56caa24d11d9a615efb6b0dadadf272328d56&mpshare=1&scene=24&srcid=02072PBLVqJj39qqvTD8NCQk&sharer_shareinfo=6967f456791f14ee20edd34e4a8075b5&sharer_shareinfo_first=6967f456791f14ee20edd34e4a8075b5#rd
---

## 🎯 Prompt工程

### 1. API 文档生成

**适用场景：** 自动生成代码 API 文档、接口说明

**直接复制使用：**

```
请为以下代码生成专业的 API 文档：

**代码：**
{paste_code_here}

**输出要求：**
1. API 概述（功能、用途）
2. 方法签名（参数类型、返回值）
3. 参数说明（名称、类型、必填、描述）
4. 返回值说明（类型、结构示例）
5. 使用示例（Python/JavaScript/其他语言）
6. 注意事项和限制
7. 相关方法链接

**格式要求：**
- 使用 Markdown 格式
- 代码块标注语言
- 示例需要可运行
```

**来源：**Google API Design Guide

---

### 2. SQL 查询优化

**适用场景：** 数据库查询性能优化、慢查询分析

**直接复制使用：**

```
你是一位数据库性能专家，请帮我优化以下 SQL 查询：

**原始 SQL：**
{sql_query}

**数据库环境：**
- 数据库类型：{MySQL/PostgreSQL/SQL Server/其他}
- 表结构信息：{describe_tables}
- 当前数据量：{estimated_rows}
- 执行计划：{explain_result}

**问题描述：**
{current_issues}

**优化目标：**
1. 提升查询速度
2. 减少全表扫描
3. 优化索引使用
4. 降低资源消耗

请提供：
1. 问题根因分析
2. 优化后的 SQL
3. 索引建议
4. 替代方案对比
```

**来源：**UseTheIndexLuke

---

### 3. README 编写

**适用场景：** 项目文档生成、开源项目说明

**直接复制使用：**

```
请为以下项目生成专业的 README.md：

**项目信息：**
- 项目名称：{project_name}
- 项目描述：{project_description}
- 主要功能：{key_features}
- 技术栈：{tech_stack}
- 目标用户：{target_audience}

**输出要求：**
1. 项目徽章（Build、License、Version）
2. 一句话简介
3. 功能特性列表
4. 快速开始（安装、使用）
5. API 说明（如适用）
6. 配置说明
7. 贡献指南
8. 许可证
9. 联系信息

**风格要求：**
- 简洁清晰
- 面向用户友好
- 包含实际示例
- 中英双语（如需要）
```

**来源：**Make a README

---

### 4. 测试用例生成

**适用场景：** 单元测试、集成测试、边界测试

**直接复制使用：**

```
请为以下代码生成全面的测试用例：

**代码：**
{paste_code_here}

**代码功能说明：**
{function_description}

**测试要求：**
1. 正常流程测试（Happy Path）
2. 边界条件测试（Edge Cases）
3. 异常处理测试（Error Handling）
4. 空值/默认值测试
5. 类型验证测试
6. 性能测试（如适用）

**输出格式（ Jest/Vitest）：**
describe('FunctionName', () => {
  test('should handle normal input', () => {
    // Arrange
    // Act
    // Assert
  });

  test('should handle empty input', () => {
    // Arrange
    // Act
    // Assert
  });

  // 更多测试用例...
});
```

**来源：**Testing Best Practices

---

### 5. 技术方案设计

**适用场景：** 系统设计，技术选型、架构评审

**直接复制使用：**

```
请设计以下技术方案：

**需求描述：**
{requirements}

**约束条件：**
- 性能要求：{performance_requirements}
- 可用性要求：{availability_requirements}
- 扩展性要求：{scalability_requirements}
- 预算限制：{budget_constraints}
- 技术栈限制：{tech_constraints}

**设计要求：**
1. 系统架构图（文字描述）
2. 核心模块划分
3. 数据模型设计
4. API 设计
5. 技术选型对比
6. 性能评估
7. 风险分析
8. 实施计划

**输出格式：**
- 架构图（Mermaid 语法）
- 模块说明
- 数据流描述
- 技术决策点
- 权衡分析
```

**来源：**System Design Primer

---

## 🤖 Skill案例

### 1. notion - 2.8K installs

**总结：** Notion 集成，支持页面创建、数据库操作、块编辑

**核心功能：** 页面 CRUD、数据库查询、块级编辑、搜索同步、团队协作

**安装命令：**

```
npx skills add anthropics/skills#notion
```

---

### 2. linear - 2.3K installs

**总结：** Linear 项目管理集成，支持 Issues、Cycles、Projects

**核心功能：** 问题追踪、周期管理、项目看板、团队协作、工作流自动化

**安装命令：**

```
npx skills add anthropics/skills#linear
```

---

### 3. github-pr - 3.1K installs

**总结：** GitHub Pull Request 管理，支持创建、审查、合并

**核心功能：** PR 创建与编辑、代码审查、状态追踪、自动合并、冲突解决

**安装命令：**

```
npx skills add anthropics/skills#github-pr
```

---

### 4. jira - 1.9K installs

**总结：** JIRA 任务管理集成，支持 Issues、Boards、Sprints

**核心功能：** 问题管理、敏捷看板、冲刺规划、工作流配置、报表生成

**安装命令：**

```
npx skills add anthropics/skills#jira
```

---

### 5. loom - 1.5K installs

**总结：** Loom 视频录制集成，支持录制、分享、分析

**核心功能：** 屏幕录制、摄像头录制、视频编辑、团队分享、观看分析

**安装命令：**

```
npx skills add anthropics/skills#loom
```

---

## 🎨 图片提示词

### 1. 霓虹夜色性感回眸

**提示词：** 惊艳女子，五官立体锐利，带着自信的微笑，回眸看向镜头。长发丰盈波浪，黑色，瀑布般垂落背部。侧身姿势，斜倚木质墙面，胸部紧贴墙面，一只手搭在臀部。紧身midi连衣裙，米色，贴身剪裁。奢华酒廊/酒吧室内夜景，木质墙面，深红色天鹅绒座椅。霓虹氛围灯，蓝色、紫色、金色。电影感摄影风格。

**生成工具：** Banana

**风格特点：** 霓虹夜景、时尚摄影、侧身回眸、奢华氛围

---

### 2. 皮克斯风少女萌宠自拍

**提示词：** 可爱的皮克斯风格3D动画女孩在户外与她那顽皮的小狗近距离自拍。女孩有一双大大的表情丰富的眼睛，脸颊上有雀斑，皮肤柔软红润，浅棕色头发扎成凌乱的发髻，做出惊讶的"O"形嘴表情。狗是一只小型梗犬，毛色为白色和棕色，舌头伸出，快乐而充满活力。明亮的自然阳光，柔和的黄金时段光线，浅景深，绿色草地背景。3D卡通写实风格，皮克斯-迪士尼动画风格。

**生成工具：** Banana **风格特点：** 皮克斯风格、3D动画、萌宠自拍、黄金光线

---

### 3. 神圣水下时尚大片

**提示词：** 女子在水下悬浮成戏剧性后弯，脊柱优雅拱起，头部向后仰向水面光线，双臂向上伸展仿佛被无形力量托起。长发如黑色火焰般向光线方向向上飘动。香槟色缎面连衣裙完全展开如巨大绽放花朵。双足均穿着Rockstud高跟鞋，裸色细高跟。戏剧性神光穿透水体，来自上方水面的强烈方向光。神圣水下大教堂氛围。

**生成工具：** Banana

**风格特点：** 水下摄影、神光穿透、时尚大片、高级质感

---

### 4. 柔和叠色剪纸童话

**提示词：** 柔和叠色剪纸风格，童话世界主题，温馨可爱的氛围，柔和的色彩渐变，层叠的纸艺效果，梦幻的童话场景，精致的手工艺细节，温暖的色调，儿童插画风格，可爱的角色设计，富有想象力的构图。

**生成工具：** Banana

**风格特点：** 剪纸艺术、童话世界、柔和渐变、手工艺

---

### 5. 超市活力少女瞬间

**提示词：** 超市场景中充满活力的少女，青春洋溢，购物车，零食，饮料，明亮的超市灯光，动态抓拍，活力四射，青春时尚，日常生活场景，购物氛围，愉快的情绪。

**生成工具：** Banana

**风格特点：** 超市场景、活力少女、日常生活、青春时尚