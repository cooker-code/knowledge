> 已吸收至：[[03_数据工程与数仓/0308_元数据血缘与治理/030803_数据血缘/030803_核心知识点/血缘图谱与AI助手应用边界|血缘图谱与AI助手应用边界]]
---
title: 闭着眼！用文档做高分知识图谱！全自动！数据发现+建模+抽取+入库！一键方案！附体验链接
author: 一意AI增效家
date:
url: https://mp.weixin.qq.com/s?__biz=MzU4NDk0MDAwMw==&mid=2247487776&idx=1&sn=84db0f9918a7fea19abe8655dda8c649&chksm=fca89fac26d8b5b5f28d9a754f62b281f6c99fc8355a02bb77379f9bc3a0a7945e1bc06b9a42&mpshare=1&scene=24&srcid=1121Da9OJJkqFl5rlQZyKo3T&sharer_shareinfo=c2f195a97684a9d6329f26e8e9b7f5a0&sharer_shareinfo_first=c2f195a97684a9d6329f26e8e9b7f5a0#rd
---

过去2年！

雄哥做了大量的MAS+知识图谱内容！

从最靠基本功的文档预处理、LLM抽取、评估消歧、高速通道入库！

我们发现，每个项目任务不同，[MAS](https://mp.weixin.qq.com/s?__biz=MzU4NDk0MDAwMw==&mid=2247486640&idx=2&sn=a38738e21b149ad0726a79beae33ce7e&scene=21#wechat_redirect)所需知识不同，数据处理的方法也，全不相同！

这！无法标准化交付！

于是，雄哥想，有没有通用的+自动干活的方案？

1，可以根据任务目标，智能定位文档中，有价值的那些

2，聚焦价值数据，匹配任务，精准设计数据模型，完成建模

3，用建模方案，自动把文档中，有价值的部分，抽取为知识图谱

4，消岐+评估知识图谱，实现自动化入库到本地neo4j中

不求满分，85分即可！

为达成以上目标，伙计做了出来，给兄弟们开放

一切确认好了，接下来就是一键导入到知识图谱

他会自动抽取数据，并完成入库！

这相当于，把整个过程，开箱！

与行业专家的交流会很高效，大家都可以摸到数据过程，并迭代他！

行业专家无需懂技术，技术也无需深入业务！

你可以在这里体验：

因有多人在用，若需上传知识图谱，请勾选“清空数据”，这样就能用你自己的数据，构建；

完全自然语言+大白话，提需求，Agent会自动干活！

直接为知识图谱类项目落地，提速至少4周+！

他背后是如何实现的？

共4步！也是今天核心内容！

① 数据发现与分析（用LLM找到数据中有价值部分）

② 数据智能建模（智能评估数据与任务共性，设计数据模型）

③ 一键抽取评估入库（以数据模型抽取需要的数据，全自动到neo4j中）

④ 什么模型？什么知识图谱？...边跑代码边聊细节（纯本地实现）

稍后，雄哥以此展开今日内容

现在，一起看看用《青年政策》表格（会员提供），做的数据模型：

```
{  "nodes": [    {      "label": "Policy",      "properties": [        {          "name": "policyId",          "type": "int",          "column_mapping": "序号",          "alias": null,          "is_unique": true,          "part_of_key": false        },        {          "name": "title",          "type": "str",          "column_mapping": "政策标题",          "alias": null,          "is_unique": false,          "part_of_key": false        },        {          "name": "issueDate",          "type": "str",          "column_mapping": "发文日期",          "alias": null,          "is_unique": false,          "part_of_key": false        },        {          "name": "documentNumber",          "type": "str",          "column_mapping": "发文字号",          "alias": null,          "is_unique": false,          "part_of_key": false        },        {          "name": "content",          "type": "str",          "column_mapping": "政策相关原文",          "alias": null,          "is_unique": false,          "part_of_key": false        }      ],      "source_name": "file"    },    {      "label": "Organization",      "properties": [        {          "name": "name",          "type": "str",          "column_mapping": "发文机构",          "alias": null,          "is_unique": true,          "part_of_key": false        }      ],      "source_name": "file"    },    {      "label": "PolicyType",      "properties": [        {          "name": "name",          "type": "str",          "column_mapping": "类型",          "alias": null,          "is_unique": true,          "part_of_key": false        }      ],      "source_name": "file"    },    {      "label": "PolicyStatus",      "properties": [        {          "name": "name",          "type": "str",          "column_mapping": "状态",          "alias": null,          "is_unique": true,          "part_of_key": false        }      ],      "source_name": "file"    }  ],  "relationships": [    {      "type": "ISSUED_BY",      "properties": [],      "source": "Policy",      "target": "Organization",      "source_name": "file"    },    {      "type": "HAS_TYPE",      "properties": [],      "source": "Policy",      "target": "PolicyType",      "source_name": "file"    },    {      "type": "HAS_STATUS",      "properties": [],      "source": "Policy",      "target": "PolicyStatus",      "source_name": "file"    }  ],  "metadata": null}
```

基本达到专业水准，可用！

* **结构清晰**

  核心实体（Policy）与维度实体（Organization、Type、Status）分离，符合规范化设计
* **语义明确**

  节点标签与关系类型命名直观，准确表达政策“由谁发布、属于什么类型、处于什么状态”
* **可落地性强**

  每个属性明确对应源表字段（`column_mapping`），便于从表格数据一键导入
* **主键设计合理**

  `policyId` 唯一标识政策，`name` 在维度节点上设为唯一，避免重复
* **轻量高效**

  无冗余属性，关系无复杂字段，适合快速构建与查询

现在，我们组个展开，他背后是怎样做的！

第一部分：数据发现与分析

---

雄哥在之前的内容也讲过

数据发现，是知识工程中最重要工作

没有之一

先盘点手上有哪些数据，做数据清单，这些清单都对应着哪些功能实现

还没看的兄弟，在这里看：

[上线！“灵丹0”-从海量文档中构建高质量知识图谱！科学方法论+实战！官方wiki！](https://mp.weixin.qq.com/s?__biz=MzU4NDk0MDAwMw==&mid=2247487713&idx=1&sn=1901307348ee3ffc140e89f36120339c&scene=21#wechat_redirect)

雄哥把这里的所有方法，浓缩精华

都写到了Agent的SOP中，他可以用这些方法，发现洞见，并指导一步步地生成

先，停下来理解特征之间的关系，哪些功能不需要，哪些节点因缺失数据变得没价值，等等！

你不用是数据工程师，但有LLM帮你，弥合差距！

但用户的任务，是各种各样的！

这个用户干医疗的、那个干电力的，领域不同，任务不同，侧重点也不同！

我们接入一个功能：支持自然语言/大白话，描述任务

Agent会为数据建模阶段生成丰富的上下文/背景信息，作为 Python 字典或 UserInput 对象

这个输入

可以写：任务的描述

也可以写：每个数据标签的描述功能

更可以写：任何希望图数据模型处理的示例+参照

这里的每一个输入，都是一个Agent数据建模时的考虑权重

稍后第四部分，也会讲代码如何实现

充分挖掘数据，有了数据报告和指引，下一个工序，就是数据建模了

第二部分：数据智能建模

---

这背后，是一个非常复杂的算法

我们会用到2个内容，作为输入：

1，数据分析报告

2，用户的任务输入

为了满足任务需要，我们指导LLM做出数据模型，强制LLM输出JSON与Pydantic！

生成的数据模型会自动遵循 Neo4j 的命名规范：

* **节点标签（Label）**

  使用 **PascalCase**（如 `Person`、`MedicalEquipment`）
* **关系类型（Relationship Type）**

  使用 **SCREAMING\_SNAKE\_CASE**（如 `BELONGS_TO`、`REQUIRES_MAINTENANCE`）
* **属性（Property）**

  使用 **camelCase**（如 `createdAt`、`lastInspectionDate`）

LLM返回的结构化JSON与数据模型，会与数据源进行验证

验证通过后，我们还会再做一次强制的验证

确保：

关系源和目标存在性、唯一属性，以及节点/关系属性名称映射到源数据中

经过双重验证后，确保高质量数据模型

也是在确保高质量的知识图谱

打卡[灵丹0](https://mp.weixin.qq.com/s?__biz=MzU4NDk0MDAwMw==&mid=2247487713&idx=1&sn=1901307348ee3ffc140e89f36120339c&scene=21#wechat_redirect)的朋友，应该就知道是如何做的

因为我们做的是85分的通用方案，各种各样的数据都会有，想让LLM做得更好，更贴切，也可以直接自然语言+大白话告诉Agent！

他会综合意见，重新再做！

直到做到高分为止，你也可以根据版本迭代，v1/v2/v3...

支持历史记录保存

第三部分：一键抽取评估入库

---

上面

相当于圈了地，还没开始做任务，种小麦

接下来这步，就是精准拿到小麦种子

然后在一片空地，找到属于自己那一亩

种下去

以前，用各种各样的方法去处理数据

现在，我们把这些处理方法都打包起来

一键实现抽取入库

neo4j的数据导入，需要生成 Cypher 代码来创建约束并加载数据

我们通过 GraphDataModeler 类的 current\_model 属性传递当前数据模型

这一步会发生几件事：

1，链接本地的neo4j（需要填写本地的neo4j的用户名/密码）

2，通过数据模型批量化抽取

3，导入到neo4j中

整个过程非常快

接下来，我们会演示他是通过什么代码实现

第四部分：边跑代码，边聊细节

---

首先，你需要下载代码和使用说明书

小伙伴已经把它都传到会员盘了，你可以直接找到工程师-小胖，即可获得链接

今天我们会从最基础的环境搭建开始，然后安装依赖，然后一键运行！

运行起来后，我们展开关键基本的代码实现！

还不是永久会员？

一意已持续输出技术内容2年+啦！会员700+！永久会员100+！

成为永久会员即可解锁过去2年全部的深度拉练，代码+视频+图文，而且一直在更新，内容涵盖大模型微调、推理优化、RAG优化、知识图谱、多智能体系统等！

在这联系！

4.1 运行过程

你要准备环境，清单如下

操作系统：三大系统均可，无指定

大模型：所有兼容OpenAI API的key，可本地ollama/vllm

环境管理：miniconda

知识图谱：neo4j

数据建模可视化：graphviz

前端：streamlit

如果你还未准备好AI环境，可以在这里搭建

[【灵丹0】从海量文档构建知识图谱-环境搭建篇](https://mp.weixin.qq.com/s?__biz=MzU4NDk0MDAwMw==&mid=2247487600&idx=1&sn=bd74f5f2e099a5d7b0e43bed86e17ce8&scene=21#wechat_redirect)

搭建好之后，跟着指令继续！

创建conda环境！

指定Python3.11，名称11kg

```
conda create -n 11kg python=3.11 -y
```

激活该环境！

```
conda activate 11kg
```

进入放代码的目录中！

```
cd y/y/11kg
```

安装依赖！

工程师把项目的依赖，都打包起来了

使用阿里源，安装好依赖

```
pip install -r requirements.txt -i https://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com
```

把你的api和本地neo4j的账号密码，填写到.env的文件中！

```
NEO4J_USERNAME="neo4j"NEO4J_PASSWORD="password"NEO4J_URI="neo4j://localhost:7687"NEO4J_DATABASE="neo4j"OPENAI_API_KEY="sk-dd1b45b***834e1dd7344e29ce7fff4b"OPENAI_BASE_URL="https://a***.com/v1"MODEL_NAME="qwen3-max"
```

程序会自动加载！

运行streamlit！

```
streamlit run app.py --server.headless true --server.port 8514
```

会自动跳转到浏览器，服务端口就在8514

你就来到了这个界面了

你无需懂数据，甚至无需懂业务！

你只需要讲想做什么！

LLM会自动帮你补齐专业的知识！

发展是在太快了，后面一意会再做一个知识图谱专题，就准备过年啦！

明年，我们有新的内容计划！

期待与你共成长！