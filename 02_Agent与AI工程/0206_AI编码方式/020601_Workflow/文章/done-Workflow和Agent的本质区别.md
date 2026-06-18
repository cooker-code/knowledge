> 已吸收至：[[02_Agent与AI工程/0206_AI编码方式/020601_Workflow/020601_核心知识点/Workflow编排与验证闭环|Workflow编排与验证闭环]]
---
title: Workflow和Agent的本质区别
author: newtype AI
date:
url: https://mp.weixin.qq.com/s?__biz=MzkyNzU0MzQwOQ==&mid=2247484537&idx=1&sn=51003b55345cbd6f7dc004baffb3f721&chksm=c3ff916d0cfdd9c38b7e0a7e90fb5c557940ceef84142971f0fb749331c3c4a054f454cd1f68&mpshare=1&scene=24&srcid=1030z9qkkqR9phl7IxAxZiwf&sharer_shareinfo=cd5854d3713b52dd18f3ff47b80622f2&sharer_shareinfo_first=cd5854d3713b52dd18f3ff47b80622f2#rd
---

Workflow的本质是定义一系列任务，以及任务之间的先后顺序、并行关系和触发条件。它的样子就像一张流程图或蓝图。

所以，Workflow的核心是“依赖关系”，侧重点是“做什么”和“按什么顺序做”。

Agent的本质是一个能够感知环境、进行自主决策并执行动作的实体。它的行为是根据实时输入和内部逻辑动态生成的，表现为一系列按时间展开的动作。

所以，Agent的核心是“决策与执行”，侧重点是“此时此刻该怎么办”。

Workflow和Agent不是替代关系，而是相辅相成的关系。一项工作必然包含两点：1、空间结构，对应Workflow；2、逻辑结构，对应Agent。

只不过，二者的抽象层次不同。Workflow是更高层次的业务流程抽象，而Agent是更低层次的执行智能抽象。一个复杂的Workflow可能由多个功能各异的Agent协作完成。

在最初，简单、固定的任务用Workflow自动化。后来，流程中遇到复杂任务，它们包含了大量不确定性、需要与外部环境动态交互、需要自然语言理解的需求，这时就引入Agent来“填充”这些节点。而这个，就是所谓的Agentic Workflow。