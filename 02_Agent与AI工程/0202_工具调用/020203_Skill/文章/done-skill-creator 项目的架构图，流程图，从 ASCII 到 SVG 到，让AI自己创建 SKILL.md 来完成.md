> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020203_Skill/020203_核心知识点/Skill能力封装与治理边界|Skill能力封装与治理边界]]
---
title: skill-creator 项目的架构图，流程图，从 ASCII 到 SVG 到，让AI自己创建 SKILL.md 来完成
author: 逻辑仓管AI运营社
date:
url: https://mp.weixin.qq.com/s?__biz=MzA3OTEwNjUwMw==&mid=2451581963&idx=1&sn=74e37020afa6e5fbd0307de88f8ef47a&chksm=89a0d1c201df394eadf746acb376ad0afffd23562944f8e251746082313b16b293a17b80b2c0&mpshare=1&scene=24&srcid=0214LWf2M9dNRc2ER3B4IzH0&sharer_shareinfo=050c52fbd93a40fb85cd14b65a062a7d&sharer_shareinfo_first=050c52fbd93a40fb85cd14b65a062a7d#rd
---

当产品经理(AI) 生成一堆架构图，流程图，的 ASCII 内容，不知道开发者是否真正认真审核阅读过，反正看起来就很高大上很合理。

但看着文档里面那堆 `+----+` 和 `|` 字符，我陷入了深深的思考：

所以让 AI 自己生成一个skill 来专门处理将文档中的 ASCII 图转换成 SVG 图，然后再转成 GIF 图。svg 效果非常不错，但gif就有点拉卡了。但可以进行优化。这里主要演示一下 /skill-creator 

---

## skill-creator

基于 `/skill-creator` 创建 skill `svg-arch-diagram` 的历史会话

### 完整时间线（从 history.txt 提取）

**第1会话：**
`/find-skills svg to png`

**第2会话：**
`/find-skills svg generator`

**第3会话：**
`为这个项目安装 svg-art svg-precision moai-tool-svg 这三个skill`

**第4会话：**
`调用多个 svg skill. 为这个 多 Agent 协作架构生成 svg 图片[Pasted text #1 +24 lines]`

**第5会话：**
`/skill-creator 根据上面流程新建这个过程的 skill, 方便我下层直接调用.`

**第6会话：**
`/svg-arch-diagram @docs/Product-Design-XX文档智能编制系统.md 所有架构或流程图.生成一张张带顺序的svg.可以添加数据流动动画效果.`

**第7会话：**
`修正图片大小,自适应.避免生成的内容超出画布大小.`

**第8会话：**
`将这个规则更新到 svg-arch-diagram SKILL.md 中,并新增一下要求:一个svg文件即包含该svg图片的所有代码. 比如:动画,节点,数据连线等等都需要在一个svg文件内完成`

**第9会话：**
`清空所有刚刚已经生成的svg以及代码.数据结构等文件. 重新使用更新后的skill执行生成.`

**第10会话：**
`/find-skills svg to gif`

**第11会话：**
`已经安装依赖,请每一张svg都对应帮我生成一张gif图片`

**第12会话：**
`更新这个skill.要求不要将数据生成到json文件中.直接在svg数据里面保存所有数据.`

**第13会话：**
`安装 svg2gif.js github 开源库, 并进行WebSerach调研这个库怎么使用,要求这个skill运行时使用svg2gif生成gif图片`

**第14会话：**
`验证 这个skill 的技术栈,运行时工具是否和SKILL.md定义的保持一致.`

**第15会话：**
`删除项目内所有之前生成的内容.然后使用这个skill重新生成,保留 svg 和 gif两个版本`

## 最终效果