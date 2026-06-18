> 已吸收至：[[02_Agent与AI工程/0201_Agent框架/020112_LangChain/020112_核心知识点/LangChain生产部署与持久化边界|LangChain生产部署与持久化边界]]
---
title: FastAPI+LangChain+Streamlit实现人机交互（HITL）
author: PyTorch研习社
date:
url: https://mp.weixin.qq.com/s?__biz=MzI2ODUyMTQyNA==&mid=2247499863&idx=1&sn=dc035b3b5394ed56463fdfac3b44dfeb&chksm=eb540c8dc6e9e340159a71b8397d1d0be234e6fb0d58b07cdb1fb633f6512d662717f467c446&mpshare=1&scene=24&srcid=1220I8CEVptqeu9CksQSMwRj&sharer_shareinfo=46a2ae2145474c252186546c8726b2d9&sharer_shareinfo_first=46a2ae2145474c252186546c8726b2d9#rd
---

随着人工智能和自动化系统在各行各业加速落地，机器正被赋予越来越多“决策权”。然而，现实世界远比算法训练时的环境复杂。一次误判的风控拦截，可能冻结正常用户的账户；一次错误的医疗模型判断，可能影响患者的治疗方案；在自动驾驶场景中，哪怕极小概率的异常，也可能带来严重后果。**当决策具备高风险、高责任或高度不确定性时，完全“无人值守”的自动化往往并不可靠。**

正因如此，**Human In The Loop（HITL，人机协作）** 成为连接算法与现实世界的重要机制。通过在关键节点引入人类的判断、审核和反馈，系统既能发挥机器在效率和规模上的优势，又能借助人类对上下文、伦理和异常情况的理解，避免“看似合理却实际错误”的决策。HITL 并不是对自动化的否定，而是让人类智慧成为智能系统的一部分，使其在真实环境中更加安全、可信和可控。

LangChain 的Human-in-the-Loop（HITL，人机协作）中间件允许我们为 Agent 的工具调用加入人工监督。当模型提出一个可能需要审核的动作时——例如写入文件或执行 SQL——该中间件可以暂停执行并等待人工决策。

它的工作方式是：将每一次工具调用与一套可配置的策略进行匹配检查。如果判定需要人工介入，中间件会触发一个 interrupt（中断），从而停止执行流程。此时，当前的执行图状态会通过 LangGraph 的持久化机制进行保存，确保执行可以被安全地暂停，并在之后恢复。

随后，由人工给出决策来决定下一步行为：

* 可以直接批准并按原样执行（approve），
* 可以在执行前修改该动作的参数（edit），
* 也可以拒绝执行并提供反馈信息（reject）。

Agent 在收集到审核人相应的决策（决策以列表形式提供，每个待审核的操作对应一个决策。）之后，再恢复执行整个流程。

在这个视频中我介绍了在《[AgentHub更新：LangGraph+千问实现Adaptive RAG系统](https://mp.weixin.qq.com/s?__biz=MzI2ODUyMTQyNA==&mid=2247499780&idx=1&sn=0b1610d8056f1056946e5d94ee932a4f&scene=21#wechat_redirect)》之后引入的 HITL 机制实现：

* 首先最大的改动在于前端页面，我新增了一个用于等待用户审核的弹框组件；
* 其次为了在页面上展示需要用户审核的信息，我在后端新增了展示中断的 InterruptMessage 消息类型；
* 第三在处理中断后的输入中新增了 Command 来恢复中断执行。

代码已经更新到了：

https://github.com/realyinchen/AgentHub

关于如何使用 LangChain + HumanInTheLoopMiddleWare 实现人机协作，你可以前往下面的链接观看：

https://articles.zsxq.com/id\_8alh7swoturl.html

或者加入 [LangChain/LangGrap](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI2ODUyMTQyNA==&action=getalbum&album_id=4272409246018568215#wechat_redirect)合集进行持续学习。

此外我还准备了 [提示工程](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI2ODUyMTQyNA==&action=getalbum&album_id=3905224169797156867#wechat_redirect) 合集，方便大家学习提示词工程。