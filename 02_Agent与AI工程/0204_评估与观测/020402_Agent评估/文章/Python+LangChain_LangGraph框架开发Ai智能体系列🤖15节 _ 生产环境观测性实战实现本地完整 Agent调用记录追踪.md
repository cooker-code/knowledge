---
title: Python+LangChain/LangGraph框架开发Ai智能体系列🤖15节 | 生产环境观测性实战实现本地完整 Agent调用记录追踪
author: 极简工具盒
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzk0MzQyNDA0Mg==&mid=2247484399&idx=1&sn=12ce467bf9ec1d9db123774cc97612a8&chksm=c2b1e3d6549984d277d99519b21611ed5c8716984f7d62df275bd06be15a108cbb3c03acb6d8&mpshare=1&scene=24&srcid=0420iSkutiE3UrfVCIbgnbc0&sharer_shareinfo=1310425c77bbffe494e5e3d5baadf2b5&sharer_shareinfo_first=1310425c77bbffe494e5e3d5baadf2b5#rd
---

提醒：如果需要本节源代码请关注“极简工具盒”公众号，在文章末尾扫二维码加入技术交流群获取！

1. 学习目标

不依赖 LangSmith 云端，自己实现一套本地观测性：JSON 日志 + Token 计数 + 成本 Dashboard：

  1. 结构化日志设计（JSON + trace\_id 串联）

  2. Token 消耗实时统计与成本告警

  3. 错误追踪与根因定位

  4. 可视化看板设计（ASCII dashboard）

2.核心概念速览

├── 生产级观测性

│   ├── AgentLogger：JSON 结构化日志 + trace\_id 串联

│   ├── CostMonitor：Token 计数 + 成本告警 + ASCII Dashboard

│   └── CallRecord：通话记录数据模型

### 核心特点

前置条件：无 API Key 也能运行

端到端演示：日志 + 成本监控 + Dashboard

3.代码实现

环境：win11+python3.11+vscode编辑器

pip依赖：

importos

importtime

importjson

importlogging

importlogging.handlers

fromdatetimeimportdatetime

fromdataclassesimportdataclass, field, asdict

fromtypingimportOptional

fromdotenvimportload\_dotenv

load\_dotenv()

**3.1第一步实现**数据模型：CallRecord（通话记录）

**3.2第二步实现结构化日志器（JSON 格式 + trace\_id 串联）**

****3.3第三步实现成本监控（Token 计数 + 成本告警 + ASCII Dashboard）：****

******3.4第四步端到端演示：日志 + 成本监控 + Dashboard"""******

4.代码运行：

执行结果：

监控log：

---

**热点文章推荐：**

[🦞10分钟极速搭建OpenClaw龙虾智能体服务（Windows11版）“炒鸡详细”](https://mp.weixin.qq.com/s?__biz=Mzk0MzQyNDA0Mg==&mid=2247483819&idx=1&sn=fe0c957335f270444c95d44d791fa31e&scene=21#wechat_redirect)

[Python+langchain框架开发Ai智能体系列（一）](https://mp.weixin.qq.com/s?__biz=Mzk0MzQyNDA0Mg==&mid=2247483855&idx=1&sn=7c3e516d0c70c1269fec4fff1f25c92f&scene=21#wechat_redirect)

后续分享更系列教程关注“极简工具盒”公众号探索更多精彩内容，谢谢！

---