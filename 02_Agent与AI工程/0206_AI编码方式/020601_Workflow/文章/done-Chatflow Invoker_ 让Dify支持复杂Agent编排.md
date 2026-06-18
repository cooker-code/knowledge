> 已吸收至：[[02_Agent与AI工程/0206_AI编码方式/020601_Workflow/020601_核心知识点/Workflow编排与验证闭环|Workflow编排与验证闭环]]
---
title: Chatflow Invoker: 让Dify支持复杂Agent编排
author: 熵减矩阵
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg2MTc1NDAxMA==&mid=2247484446&idx=1&sn=f6524a24941ed65187adc85a605fa0fd&chksm=cfc8c0a35cf71ed37b40376f75d9e3dfa4a2f322a3002ffd32434334e193618081596e8676ff&mpshare=1&scene=24&srcid=09115zBol0ZDZlYotOIB7jaa&sharer_shareinfo=203be6cc82a314782a5d4098bdf069b9&sharer_shareinfo_first=203be6cc82a314782a5d4098bdf069b9#rd
---

## 前言

> Dify是一个开源的 LLM 应用开发平台，提供从 Agent 构建到 AI workflow 编排、RAG 检索、模型管理等能力，可以让你轻松构建和运营 AI 应用。

Dify是本人目前用过的最顺手，也是设计最优雅的AI工作流编排平台。开发团队对于新技术的支持很快，迭代速度堪比生产队的驴。

不可避免地，在深度使用的时候发现了Dify的一些问题，有的用了迂回的办法去绕过，有的已经给官方提了fix的PR。这里跟大家分享一下其中一个问题的解决方案：如何在Dify下实现任意Chatflow的组合调用。

## 背景

当前Dify并不支持多Chatflow编排和跨Chatflow的调用，这意味着所有业务逻辑都必须在一个Chatflow画布中完成，当场景变得复杂时，画布将变得难以维护。

尽管Dify提供了将Chatflow转换为Workflow，并发布为Tool节点这种变通的调用方案，但这种方法存在以下限制：

* • **无法实现流式输出**：Workflow作为Tool节点调用时，不支持Chatflow原有的流式输出能力，影响用户体验。
* • **无法实现多个输出节点**：Workflow不像Chatflow那样支持多输出节点，进一步限制了复杂业务场景下的数据处理和展示。

因此我开发了一款插件：Chatflow Invoker，可以解决Dify在多Chatflow编排上的限制，支持将本地/远程的Chatflow转换为流程编排中的节点，实现跨Chatflow调用，让应用开发更加灵活和高效。

它可以做到：

* • **实现Chatflow的模块化**：将复杂业务逻辑拆分为多个独立的Chatflow，提高代码复用性和可维护性。
* • **支持跨Chatflow调用**：在不同的Chatflow之间无缝调用，实现更灵活的业务流程编排。
* • **保持流式输出体验**：确保在多Chatflow调用场景下依然能够享受Dify原有的流式输出能力。

## 用法介绍

目前支持两种调用方式：本地Chatflow调用和远程Chatflow调用。

### 本地Chatflow调用

输入参数：

* • APP ID（必填）：需要调用的Chatflow的APP ID，可以从Dify的Chatflow页面URL中获取。
* • Prompt（必填）：要发送的Prompt。
* • Inputs JSON（可选）：Chatflow开始节点的输入参数，JSON字符串格式。
* • Conversation ID（可选）：Chatflow会话ID，需要基于之前的聊天记录继续对话，必须传之前消息的 conversation\_id。

这里模拟了一个简单的场景。

首先打开待调用Chatflow的URL，从中获取APP ID

例如：https://dify/app/f011f58c-b1ce-4a9b-89b2-f39fce8466a8/workflow

这里的 `f011f58c-b1ce-4a9b-89b2-f39fce8466a8` 就是APP ID

Inputs JSON这里设置为需要收到一个user的参数。

在回复节点这里，需要选择stream\_output来获取流式输出结果。

测试执行，成功调用其他Chatflow，并且支持流式输出。

### 远程Chatflow调用

为了进一步扩大Dify的灵活性，本插件还支持了远程Chatflow调用。可以让你不再局限于单个Dify实例，而是根据业务需要来自由组合，实现分布式调用。

输入参数：

* • URL（必填）：需要调用的远程Dify的URL，例如：http://127.0.0.1:5001/v1/chat-messages
* • API Key（必填）：需要调用的远程Chatflow的API Key，第一次需要从侧边栏 访问API 中生成一个。
* • Prompt（必填）：要发送的Prompt。
* • User（必填）：Chatflow用户标识，用于定义终端用户的身份，方便检索、统计。
* • Inputs JSON（可选）：Chatflow开始节点的输入参数，JSON字符串格式。
* • Conversation ID（可选）：Chatflow会话ID，需要基于之前的聊天记录继续对话，必须传之前消息的 conversation\_id。

首先在需要被调用的Chatflow里申请一个API Key

然后将地址和API填入到插件中

输入Prompt，即可远程调用Chatflow，并同时具有流式输出的效果

远程调用还有一个好处是可以在被调用Chatflow里看到调用的日志，而本地调用则不会有日志保存。

## 最后

开发Dify插件的参考文章是比较少的，AI也缺少相关训练数据，不过好在官方的仓库里有大量的样例可以学习。

在本地Chatflow调用这里，本来是想用app-selector的方式去实现，但是试了一下发现app-selector在Tool场景下似乎有BUG，能出现界面但无法选中，所以最后还是改成了手动填写的方式。

目前Chatflow Invoker已经在Dify官方仓库上线，可以搜索安装使用。

源码地址：https://github.com/yzddmr6/chatflow\_invoker