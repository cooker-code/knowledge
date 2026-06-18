---
title: FastAPI+Streamlit打造一套智能体应用GUI展示平台
author: PyTorch研习社
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI2ODUyMTQyNA==&mid=2247499620&idx=1&sn=62733ed7cb63cee0eb43fe36477a4570&chksm=ebbe4b62d2cc15184a5e18ec319730573a0673f05d6278e23ae20dd159d9d767fecc334f4505&mpshare=1&scene=24&srcid=1130q3PSxg5QH4jSBVhSFLuY&sharer_shareinfo=72983f9e51ab7afd792feec074a4fc2c&sharer_shareinfo_first=72983f9e51ab7afd792feec074a4fc2c#rd
---

在《[LangChain1.0教程：给我们的RAG智能体加上记忆功能](https://mp.weixin.qq.com/s?__biz=MzI2ODUyMTQyNA==&mid=2247499545&idx=1&sn=f82604036df4d20ac783bd5f07915f27&scene=21#wechat_redirect)》中我们已经学会了如何在 Juyputer Notebook 中构造一个具有记忆能力的 RAG Agent。

虽然 Juyputer Notebook 对我们学习使用 LangChain/LangGraph 构建各种 Agent 应用来说很方便，但是十分不利于我们向他人展示我们的学习成果。

因此我决定在 https://github.com/JoshuaC215/agent-service-toolkit 这个项目的基础上由简入繁打造一套 Agent 应用 GUI 展示平台——AgentHub。

AgentHub具有以下特点：

✅ FastAPI 后端 — 稳健的 RESTful API 层，用于 Agent 调度与异步任务管理。

✅ Streamlit 前端 — 交互式网页界面，用于实验 Agent 与可视化推理图谱。

✅ LangChain/LangGraph 集成 — 轻松构建设计并连接多 Agent 推理工作流，并进行可视化。

✅ 流式与事件驱动 — 实时 token 流输出和 Agent 执行事件的可视化。

虽然我们有前端，但是整个 AgentHub 都是纯 Python 实现，所以大家不必担心不会前端知识。

项目源码位于：https://github.com/realyinchen/AgentHub

* 其中前端代码均位于 src/streamlit\_app.py 中。
* 整个后端代码位于 src/app 目录中：

+ 后端主函数位于 src/app/main.py；
+ app/agents 用于存放将来实现的各种 Agent;
+ app/api 中实现后端与前端交互的 API 接口，当前实现了：

- stream：流式传输
- invoke：非流式传输
- history：查询用户历史交互记录（前端暂未展示）

+ app/core 存放系统级配置信息，包括模型配置；
+ app/schema 存放了用户交互数据结构；
+ app/utils 存放各种工具方法，当前只有消息处理工具；

* 后端统一运行脚本位于 src/run\_backend.py；
* 前后端交互通过 src/agent\_client.py 完成。

这个项目刚刚起步，TODO list 还有一大堆。我尽量每天更新一点新功能上去，未来我将继续通过 Jupyter Notebook 介绍 LangChain/LangGraph 的基础知识以及构造各种 Agent 的方法，同时也会把这些 Agent 集成到 AgentHub 中。

欢迎大家贡献代码，我们一起将 AgentHub 发扬光大！

需要学习提示词工程的小伙伴也可以前往公众号主页查阅相关合集教程：