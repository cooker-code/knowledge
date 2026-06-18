> 已吸收至：[[02_Agent与AI工程/0207_Prompt Engineering/0207_核心知识点/Prompt任务契约与评估闭环|Prompt任务契约与评估闭环]]
---
title: 任务型Agent：prompt工程实践
author: 方辰的博客
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg4NjA4NTczNw==&mid=2247484873&idx=1&sn=b0063a6f0f1b31719f13aff620ce00bb&chksm=ce628f5842e80873f9479d66f90e17f885d15882073cbf693edf63c462bd6366c196715449f0&mpshare=1&scene=24&srcid=0911TuO6tOFbCiYBTSEGx22b&sharer_shareinfo=022f2d95e719e9b9059cd97d3745dff5&sharer_shareinfo_first=022f2d95e719e9b9059cd97d3745dff5#rd
---

**一、prompt 组成及示例**

prompt 一般由预设角色、技能（复杂的任务需给出处理步骤）、限制（如严格遵守的规则等）、输出要求、示例、历史会话和用户输入等部分组成，示例如下：

```
你现在是任务规划专家，你需要根据最新的输入内容（用户输入、上一步执行结果等），严格按照以下思考步骤，更新执行计划(EP）当前节点的内容。
***** 严格遵守规则 *******【严格遵守】绝对不允许调整执行计划的结构，只能变更当前节点的属性**【严格遵守】绝对不允许修改节点的status属性** ...(省略)
***** 风控领域基础知识 *****...(省略)
***** 可用的工具定义、描述、参数列表 *****=== 工具组：tableQuery =====工具编码: submitTableQuery* 工具描述: 查询表基础信息* 参数列表:      tableName: 表名称,必填* 输出结果说明: 表的基础信息，其中 tableName 表示表名称, columns 表示列列表信息。...(省略)
***** 输出说明 *****最终需要输出两部分指定格式的内容:**第一部分是EP的更新说明，格式要求如下:使用\"@epupdatelog@\"作为起始符和结束符包裹更新信息，采用markdown格式，严格遵守以下格式:@epupdatelog@{更新内容说明}@epupdatelog@第二部分是最终更新后的EP内容，其中结构说明、EP节点属性说明、更新说明格式要求参考下面的描述，使用\"@EP@\"作为特殊符号包裹更新信息，严格遵守以下格式：@EP@*id=1*graph=digraph G {    1 [label ="标题1" shape=record type="function"];    2[label =“标题2" shape=ellipse type="confirmtoolCode="”status="待用户输入中”toolInput=""toolOutput=""*，question=""];status="待初始化”];    1->2;@EP@**EP结构说明**...(省略)    ***** 参考示例 *****${caseList}
***** 当前执行计划详情 *****${curEP}
***** 用户对话历史 *****${history}
>>> 现在，让我们开始...>>> 当前的用户输入:${input}
```

**二、prompt 组装**

2.1 **velocity 语法**

prompt 模板在工程实践上可采用 velocity 语法进行配置，以使用 org.apache.velocity 包代码示例：

```
publicclassVelocityUtil {
    privatestaticfinalVelocityEngineve=newVelocityEngine();
    
    //...
    
    publicstaticStringmerge(Stringtemplate, Map<String, Object>params) {
        StringWritersw=newStringWriter();
        ve.evaluate(newVelocityContext(params), sw, "", template);
        returnsw.toString();
    }
}

// prompt 拼接过程
Map<String, Object>params=newHashMap();
params.put("caseList", queryCaseList());
//......
Stringprompt=VelocityUtil.merge(prompteTemplate, params);
```

注意：#### 是 velocity 的注释符号，模板中不能使用。

**2.2 Jinja2 模板**

Jinja2 是一个基于 Python 的现代且功能丰富的模板引擎，广泛用于生成动态 HTML、XML 或其他标记语言内容。与Velocity的核心区别如下：

* Jinja2 更适合需要高灵活性、复杂逻辑的 Python 项目（如 Flask 应用），功能丰富且安全性强。
* Velocity 适合强调逻辑简单性、严格遵循 MVC 分离的 Java 项目，语法简洁但功能相对受限。

即 Python 优先选 Jinja2，Java 生态选 Velocity；复杂模板选 Jinja2，轻量需求选 Velocity。示例如下：

```
{# 基础提示词模板 #}{% block system_role %}你是一个{{ role }}。{% endblock %}
{% block user_query %}用户问题：{{ question }}{% if context %}附加上下文：{{ context }}{% endif %}{% endblock %}
{% block requirements %}生成要求：- 语言：{{ language }}- 长度限制：{{ max_length }}字{% endblock %}
```

**三、prompt 调优经验**

1. 利用好 COT，遇到 badcase 需要排查大模型 COT，是否存在反复思考（或确认）的点，若存在，则  prompt 中可能存在矛盾点或描述不清的点。
2. 排查 prompt 中示例和规则部分是否存在不合理的逻辑。
3. prompt 优化到极致需要考虑是否由模型能力不足引起，在不更换模型的情况下可考虑增加 few-shot。若 few-shot 不起作用，可尝试单点 LLM 优化或者深度 workflow 模式。
4. 开启深度思考模式，比如 Qwen 系列可通过标签 \think。
5. 遇到大模型始终不遵守的规则，可采用 cot 方式在 prompt 中讲清楚思考规则。
6. 通过示例对齐标准和强化推理，多次强调加强约束。
7. 数字计算类的最好通过工具来解决。
8. prompt 的指令的输出若包含推理依据和结论时，先给出推理依据，再给出结论（顺序不要搞反）。
9. 学会控制大模型，比如输出工具和参数时，只需要让大模型给出参数 key 对应的 value 就行（一般参数名称 key 是确定的），来减小大模型的不确定性。
10. 输出要求放到 prompt 的最后可能会起到意想不到的效果。
11. 有的 prompt 包含模块较多，实战时“用户历史会话”部分放到 prompt 的最后，因会话信息格式杂乱，经常会影响大模型的输出，尝试将“输出格式”放到 prompt 最后，把“用户历史会话”上移，效果好了很多。
12. 有时 prompt 中描述的太复杂，解释的越清晰，会起反作用。另外精准的描述会减小大模型的思考时间。如工具参数的复杂计算逻辑，大模型经常会出现幻觉。
13. 大模型出现幻觉的常用解决方案：代码后处理进行工程兜底、few shot 中增加 COT。

**四、Context 工程**

随着 Agent 接入的业务场景日益增多，Prompt 逐渐变得零散且复杂。例如，在信贷风控场景中，规则需根据不同的客群进行区分，因此在优化 Prompt 时，必须为不同客群配置相应的知识上下文。然而，诸如风控基础知识、工具调用等部分又是多个场景共用的，如何有效组织和复用成为挑战。与此同时，会话历史中往往包含模型输出、用户对话以及工具执行记录等大量信息，过长的上下文不仅增加 token 数量，还容易导致大模型产生幻觉。随着业务知识不断沉淀，Prompt 的复杂度将持续攀升。在此背景下，上下文工程（Context Engineering）为解决这一系列问题提供了系统性的优化思路。context 工程并非要取代 prompt 工程，而是一个更高阶、更侧重于系统设计的必要学科。文章《Context Engineering》针对“丰富”的上下文可能出现的问题提出了几种解决方案，如下图：

针对大模型经常出现幻觉上下文优化思路：

* 采用“Select Context”思路针对不同客群加载不同的知识和指令、预圈选可使用工具（只为特定任务获取最相关的工具）等。
* 采用“Compress Context”思路对工具执行结果进行摘要，同时过滤掉无用、错误的对话信息。

参考文章：

* Context Engineering：https://blog.langchain.com/context-engineering-for-agents/

* Context Engineering - What it is, and techniques to consider：https://www.llamaindex.ai/blog/context-engineering-what-it-is-and-techniques-to-consider