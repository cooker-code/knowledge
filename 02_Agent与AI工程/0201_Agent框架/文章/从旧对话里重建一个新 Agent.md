---
title: 从旧对话里重建一个新 Agent
author: HebLabStudio
date: heblabheblab
url: https://mp.weixin.qq.com/s?__biz=Mzk2OTA0ODE5OA==&mid=2247483784&idx=1&sn=8eab3b1ebebc6d4bb1203cf1ff0772f1&chksm=c5573d3db003ab0f2fb0150a98bd8e9bddd7b9ff7e0f27adc04bfb062acb029086c48acae09a&mpshare=1&scene=24&srcid=0515mbmxO2Z5HCMe9Uz1ApKb&sharer_shareinfo=c6030a453017cce20edd697d6eb31c21&sharer_shareinfo_first=c6030a453017cce20edd697d6eb31c21#rd
---

备份旧 Agent 的时候，最常见的错误是：

> 我已经把聊天记录导出了，所以资产保住了。

不对。

聊天记录只是矿石。

真正的资产，是从矿石里炼出来的金属。

如果你只是把几百个 Markdown、JSON、日志文件打包放进硬盘，它们短期看起来很安全，长期大概率再也不会打开。

所以迁移 Agent 的重点，不是“我有没有备份”。

而是：

> 我有没有把旧对话提炼成新 Agent 能直接使用的能力？

---

## 一、先导出，不要边删边想

在任何卸载、重装、迁移之前，先完整导出。

最基础的导出对象包括：

```
sessions/          # 对话历史  
memory/            # 日期记忆  
MEMORY.md          # 长期记忆  
USER.md            # 用户偏好  
.learnings/        # 纠错与学习记录  
skills/            # 技能目录  
workspace/         # 工作区产物  
local_memos_db/    # 本地记忆数据库
```

导出时不要急着分类。

第一阶段只做一件事：

**保证原始材料完整。**

因为一旦你开始手动筛选，就可能误删还没意识到价值的东西。

所以建议先有一个只读原始备份：

```
raw_backup_YYYYMMDD/
```

后续所有整理，都从它复制一份出来做。

---

## 二、把对话按任务类型分类

导出之后，不要从第一条对话开始读。

那样很快会崩。

更好的方法是按任务类型分类。

比如：

| 任务类型 | 可能对应资产 |
| --- | --- |
| GitHub 仓库研究 | research workflow / repo analysis skill |
| 公众号文章创作 | writing workflow / style memory |
| 系统架构设计 | PRD skill / architecture flow |
| 故障排查 | troubleshooting skill / command playbook |
| 图片 Prompt 生成 | visual prompt template / style library |
| 多 Agent 设计 | agent role matrix / collaboration protocol |

分类时可以先用关键词：

```
grep -r "GitHub\|仓库\|README\|clone" sessions/  
grep -r "公众号\|标题\|封面\|排版" sessions/  
grep -r "PRD\|架构\|数据表\|API" sessions/  
grep -r "报错\|failed\|Error\|配置" sessions/
```

这一步的目标不是分析，而是聚类。

先把相似任务放到一起。

---

## 三、从重复任务里提炼 Workflow

只出现一次的任务，先不要急着做 Skill。

真正值得沉淀的是反复出现的任务。

比如你多次让 Agent 做“项目需求分析”，那么旧对话里很可能已经自然形成了一条流程：

```
项目背景 → 用户目标 → 角色与场景 → 功能边界 → 数据模型 → API 分层 → 前端模块 → 验收标准 → 开发任务拆解
```

这就是 Workflow。

Workflow 比 Skill 更底层。

Skill 是“能力模块”。

Workflow 是“这类任务应该怎么走”。

一个好的 Workflow 应该包含：

```
任务名称：公众号文章整理  
适用场景：用户提供草稿，希望整合成可发布文章  
输入材料：草稿、参考段落、目标平台、风格要求  
执行步骤：  
  1. 识别主题主线  
  2. 判断是否需要拆成系列  
  3. 合并重复观点  
  4. 删除不可靠表达  
  5. 调整为公众号排版  
  6. 生成标题、摘要、封面 Prompt  
  7. 给出发布建议  
输出结果：完整文章 + 标题方案 + 配图建议  
注意事项：保留文学性，但不夸大，不编造理论
```

这份 Workflow 可以直接成为新 Agent 的操作手册。

---

## 四、把 Workflow 升级成 Skill

不是所有 Workflow 都需要变成 Skill。

只有满足三个条件，才值得正式 Skill 化：

1. 高频出现
2. 输入输出稳定
3. 下次还会继续用

比如“公众号文章整理”就很适合变成 Skill。

它的 Skill 文档可以这样写：

```
# Skill: WeChat Article Refiner  
  
## Purpose  
把用户提供的草稿、片段、参考资料整理成可发布的微信公众号文章。  
  
## Inputs  
- 草稿正文  
- 目标读者  
- 发布平台  
- 风格要求  
- 是否允许拆成系列  
  
## Process  
1. 提取核心命题  
2. 判断一篇还是多篇  
3. 建立文章结构  
4. 合并重复内容  
5. 删除不准确和过度幻想表达  
6. 输出公众号排版版  
7. 附标题、摘要、封面图提示词  
  
## Output  
- 主标题  
- 摘要  
- 正文  
- 结尾引导  
- 配图建议  
- 分发建议  
  
## Guardrails  
- 不编造事实  
- 不把观察写成诊断  
- 不为了爆款牺牲准确性  
- 不使用空泛鸡汤式结尾
```

这就完成了从“旧对话”到“正式 Skill”的转化。

---

## 五、把事实写入 Memory，把偏好写入 USER

很多人迁移 Agent 时，会把所有内容都塞进 MEMORY。

这会导致记忆越来越臃肿。

更好的做法是分层：

### MEMORY.md：长期事实

适合写：

* 用户长期项目
* 正在进行的产品
* 常用技术栈
* 业务方向
* 已确定的方法论

例如：

```
用户长期关注 AI Agent、本地知识管理、OpenClaw/Hermes、多 Agent 协作中台、公众号技术内容创作。常用输出形式包括 PRD、系统架构、任务拆解、公众号排版、视觉提示词。
```

### USER.md：协作偏好

适合写：

* 输出风格
* 沟通习惯
* 内容边界
* 禁忌表达
* 质量标准

例如：

```
用户偏好结论先行、结构清晰、工程化表达。公众号文章可以有文学性，但必须克制，不要幻想化，不要使用没有依据的理论比喻。涉及技术和历史事实时要谨慎核对。
```

### WORKFLOWS.md：任务流程

适合写：

* 公众号文章整理流程
* GitHub 仓库研究流程
* PRD 生成流程
* 故障排查流程

### SKILLS.md：能力索引

适合写：

* 已启用 Skill
* 触发词
* 适用场景
* 输入输出

记忆不是越多越好。

记忆越清晰，新 Agent 越容易继承。

---

## 六、保留“纠错记录”，它比成功案例更有价值

很多人只想保留成功产物。

比如最终文章、最终代码、最终方案。

但对 Agent 迁移来说，纠错记录同样重要，甚至更重要。

因为成功案例告诉新 Agent：

> 这样做是对的。

纠错记录告诉新 Agent：

> 这样做会惹用户不满意。

比如：

```
错误：把历史人物、技术概念、项目背景混淆。  
纠正：涉及历史和技术背景时，要先确认，不要凭印象补全。  
  
错误：公众号文章过度使用宏大叙事。  
纠正：保持文学性，但要克制、真实、有工程感。  
  
错误：输出太像模板。  
纠正：减少万能开头和万能结尾，保留具体场景。
```

这些内容应该进入：

```
.learnings/LEARNINGS.md
```

或者整理成：

```
DO_NOT_REPEAT.md
```

一个好的 Agent，不只是知道怎么做。

还要知道哪些坑不要再踩。

---

## 七、生成新 Agent 的导入包

最后，把提炼出来的内容整理成一个导入包。

推荐结构：

```
agent_import_pack/  
├── MEMORY_IMPORT.md  
├── USER_STYLE.md  
├── WORKFLOWS.md  
├── SKILLS_INDEX.md  
├── DO_NOT_REPEAT.md  
├── IMPORTANT_DIALOGUE_SAMPLES.md  
└── README.md
```

其中最重要的是 `README.md`。

它应该告诉新 Agent：

```
你正在接手一个旧 Agent 的工作资产。  
  
请先读取：  
1. MEMORY_IMPORT.md：了解长期项目与事实  
2. USER_STYLE.md：了解用户偏好与表达边界  
3. WORKFLOWS.md：了解常见任务流程  
4. DO_NOT_REPEAT.md：了解过去被纠正的问题  
5. SKILLS_INDEX.md：了解可复用技能  
  
不要把这些内容当作静态资料，而要作为后续回答和执行任务的行为依据。
```

这一步完成后，新 Agent 才不再是“从零开始”。

它至少拿到了旧 Agent 留下来的地图。

---

## 八、结语：备份只是保存过去，提炼才是延续能力

AI 时代，真正值得保存的不是某一次回答。

而是回答背后的能力结构。

一次对话可以变成一个经验。

十次同类对话可以变成一个流程。

一个稳定流程可以变成一个 Skill。

长期纠错可以变成 USER 偏好。

反复确认的事实可以变成 Memory。

这就是从旧 Agent 到新 Agent 的迁移逻辑。

不是把文件搬过去。

而是把能力提炼出来。

所以，当你下一次卸载 OpenClaw、迁移 Hermes、重装某个本地 Agent 时，不要只问：

> 数据有没有备份？

还要问：

> 能力有没有继承？

真正的 Agent 迁移，不是让新系统拥有旧文件。

而是让新系统接住旧协作关系里已经长出来的东西。