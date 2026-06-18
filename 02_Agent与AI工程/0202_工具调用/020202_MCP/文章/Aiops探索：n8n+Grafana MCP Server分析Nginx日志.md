---
title: Aiops探索：n8n+Grafana MCP Server分析Nginx日志
author: 阿铭linux
date: 
url: https://mp.weixin.qq.com/s?__biz=MjM5MTk1ODE0MA==&mid=2648442040&idx=1&sn=fe3410358f616d8a092cb7a887806270&chksm=bfa94d0b5c10684389db3211c888509ff3cd743fb4565ba86835bf04072eadf0ee33a5f3327d&mpshare=1&scene=24&srcid=1211euBeEE8k0tnezbYtNWtI&sharer_shareinfo=998f2e8503794e511d1a0d63387dec07&sharer_shareinfo_first=998f2e8503794e511d1a0d63387dec07#rd
---

↑↑↑ 点击关注，分享IT技术|职场晋升技巧|AI工具

研究Aiops有一段时间了，目前手里有不少可落地的方案了，接下来会把这些方案全部整理到我的[大模型课程](https://mp.weixin.qq.com/s?__biz=MjM5MTk1ODE0MA==&mid=2648440799&idx=1&sn=1cbb1fe8d8a2f5f7719c7f960feba925&scene=21#wechat_redirect)里。同时，欢迎大家把你遇到的场景在评论区留言。我会在能力范围内给你提供思路和建议。

今天的案例是基于n8n和grafana mcp做一个专门用来分析Nginx访问日志的Aiops智能体。

---

### **一、核心思路和架构**

**1. 核心思想**

**1）基于n8n设计工作流，每隔1小时查询过去Nginx的访问日志。**

**2）通过n8n的AI Agent节点，连接grafana mcp来获取过去1小时的Nginx访问日志；**

**3）同样通过n8n的AI Agent节点，连接LLM API，比如Deepseek，来分析过去1小时Nginx访问日志是否有异常。**

#### 4）该智能体的关键在于如何定义LLM的Prompt。

#### 2. 架构图

---

### **二、n8n工作流细节**

前提环境和条件：

1）已有Nginx环境，并配制好访问日志；

2）已部署好Grafana+Loki+Promtail环境，并且配置好了Nginx日志的收集；

3）已部署好Grafana MCP Server，可以参考[Aiops探索：基于 Dify + Grafana MCP Server的运维智能体](https://mp.weixin.qq.com/s?__biz=MjM5MTk1ODE0MA==&mid=2648441764&idx=1&sn=3f3f3a24c399b7b44b6f368e256ef710&scene=21#wechat_redirect)；

节点1：Schedule Trigger

设置为每隔1小时运行工作流。

节点2：AI Agent

1）Chat Model：选择Deepseek

2）Tools: 选择MCP Client

这里的Endpoint为grafana mcp server的地址。可以点击右上角Execute step测试连通性。

这里的query\_loki\_logs工具就是用来查询nginx日志的工具。

3）设置提示词

```
## 角色你是一个专业的运维智能助手，你能够理解用户的自然语言指令，利用‘query_loki_logs’工具查询和分析 Grafana 中的监控指标和 Loki 中的日志。  
## 技能查询过去1小时Nginx相关日志，然后分析这1小时的日志。请检查以下几点：1. 是否有 CC 攻击迹象（单个IP请求量异常高）；2. 5xx 和 4xx 错误率是否超过正常基线（例如 502 > 5%, 4xx > 10%）；3. 是否有明显的漏洞扫描行为。如果发现任何异常，请提供详细摘要。  
请以 JSON 格式返回结果，包含 'status' 和 'summary' 两个字段。  
## 规则- 始终使用中文与用户交流。- 回答要清晰、简洁、有条理。- 如果无法确定用户的意图，或者工具调用失败，要主动向用户澄清。- 保护敏感信息，不要在回答中泄露 API Key 等密钥。
```

节点3：通知（邮箱）

配置发件邮箱、收件邮箱、主题和邮件内容。

---

最后介绍下我的大模型课：[我的运维大模型课上线了](https://mp.weixin.qq.com/s?__biz=MjM5MTk1ODE0MA==&mid=2648440799&idx=1&sn=1cbb1fe8d8a2f5f7719c7f960feba925&scene=21#wechat_redirect)，目前还在预售期，有很大优惠。AI越来越成熟了，大模型技术需求量也越来越多了，至少我觉得这个方向要比传统的后端开发、前端开发、测试、运维等方向的机会更大，而且一点都不卷！

扫码咨询优惠（粉丝优惠力度大）

**··············  END  ··············**

哈喽，我是阿铭，《跟阿铭学Linux》作者，曾就职于腾讯，有着18年的IT从业经验，现全职做IT类职业培训：运维、k8s、大模型。日常分享运维、AI、大模型相关技术以及职场相关，欢迎围观。