---
title: NanoBot源代码详解-5：定时任务、消息总线及心跳机制
author: 亍云旁观
date: 
url: https://mp.weixin.qq.com/s?__biz=MzA5MjM2MzE3MA==&mid=2247484086&idx=1&sn=3199a226dfdf996684e1a3b99c8d1910&chksm=9155192c4503cd097c9aa88d58203154edd7de81da6a217ce128dcdcff8be30b9f0a9126321a&mpshare=1&scene=24&srcid=0215CinOtOBLsKgT1pAh2fHp&sharer_shareinfo=6b16cbfc7ba98ab4e425dddbda4ec6a1&sharer_shareinfo_first=6b16cbfc7ba98ab4e425dddbda4ec6a1#rd
---

## 定时任务

核心的文件结构

```
nanobot/  
├── cron/  
│   ├── __init__.py     # 公共接口导出  
│   ├── types.py        # 类型定义（数据模型）  
│   └── service.py      # 调度服务实现  
├── agent/tools/cron.py # AI代理工具集成  
└── cli/commands.py     # 命令行接口（部分）
```

types.py中定义了五个数据模型

* CronSchedule: 定义任务的调度规则。
* CronPayload: 定义任务触发时要执行的内容。
* CronJobState：定义任务的状态
* CronJob: 表示一个完整的定时任务。
* CronStore：任务内容持久化，保存到磁盘。

CronService 是整个模块的大脑，负责管理任务生命周期和调度执行。

nanobot自己实现了定时任务，并没有使用系统自带的定时任务系统。

任务的持久化就是将相关信息保存到本地的JSON文件中。 所以在 `nanobot/cli/commands.py` 中，cron\_app 定义了 add, list, remove, enable, run 等命令。这些命令实例化 CronService，直接操作底层的 JSON 存储文件，无需启动后台服务即可修改配置。

Cron 任务在等待时， 唤醒机制完全依赖于 Python asyncio 事件循环（Event Loop）的 内部定时器 。 CronService计算出距离下一个任务还需要多久（例如delay\_s秒），然后创建一个内部任务tick()，在CronService类中，

```
def _arm_timer(self) -> None:  
       """Schedule the next timer tick."""  
        ...  
       async def tick():  
           await asyncio.sleep(delay_s)  
          ...
```

在时间流逝了delay\_s后，事件循环检测到定时器到期，恢复tick()任务的执行。代码从 await 之后继续运行，调用 `_on_timer()` 执行任务。

## 消息总线

### 数据模型 (Data Models)

`/nanobot/bus/events.py`中定义了两种标准化的消息类

* InboundMessage (入站消息) : 接收来源于各个 Channel（如 Telegram, WhatsApp）产生的消息，如果是通知消息，channel就是system。
* OutboundMessage (出站消息) :由 AgentLoop 产生，发送到Channel通道

### 消息总线 (Message Bus)

/nanobot/bus/queue.py中实现MessageBus类，这个类是整个系统的通信中枢，基于Python的asyncio.Queue 实现。

```
class MessageBus:  
    def __init__(self):  
        # 双向消息队列  
        self.inbound: asyncio.Queue[InboundMessage]    # 入站消息队列  
        self.outbound: asyncio.Queue[OutboundMessage]  # 出站消息队列  
  
        # 出站消息订阅者（按渠道分组）  
        self._outbound_subscribers: dict[str, list[Callable]] = {}  
        self._running = False
```

在初始化时就就在内部创建了两个独立的异步队列：

* inbound : Channel -> Agent (外部消息流入)
* outbound : Agent -> Channel (AI 回复流出)

核心方法

| 方法 | 功能 | 调用者 |
| --- | --- | --- |
| `publish_inbound(msg)` | 将渠道消息发布到入站队列 | `Channel._handle_message()` |
| `consume_inbound()` | 阻塞，直到获取下一条入站消息 | `AgentLoop.run()` |
| `publish_outbound(msg)` | 将响应发布到出站队列 | `AgentLoop, CronService` |
| `consume_outbound()` | 阻塞，直到获取下一条出站消息 | `ChannelManager._dispatch_outbound()` |
| `subscribe_outbound(channel, callback)` | 订阅特定渠道的出站消息 | - |
| `dispatch_outbound()` | 分发出站消息到订阅者（后台任务） | - |
| `inbound_size` / `outbound_size` | 获取队列大小（监控用） | - |

上面dispatch\_outbound- 是一个后台运行的 Task。它不断从 outbound 队列取出消息。它查找 `_outbound_subscribers` 字典，找到订阅了该 channel 的所有回调函数，并执行它们。

```
while self._running:  
  msg = await asyncio.wait_for(self.outbound.get(), timeout=1.0)  
  subscribers = self._outbound_subscribers.get(msg.channel, [])  
  for callback in subscribers:  
      await callback(msg)
```

## 心跳机制

在nanobot/heartbeat目录下，nanobot项目提供了一个心跳机制。

不过这个心跳机制和我们平时理解的从中保活功能不一样。它其实是一个定期唤醒代理的主动式服务，用于检查和处理预定义的任务。它通过读取工作区中的 HEARTBEAT.md 文件，让代理定期执行其中列出的任务，实现自我管理和自动化工作流程。

默认情况下，它每隔30分钟，发送一个提示词给AI Agent，让它检查一下HEARTBEAT.md文件里是否有要执行的任务。

```
# Default interval: 30 minutes  
DEFAULT_HEARTBEAT_INTERVAL_S = 30 * 60  
  
# The prompt sent to agent during heartbeat  
HEARTBEAT_PROMPT = """Read HEARTBEAT.md in your workspace (if it exists).  
Follow any instructions or tasks listed there.  
If nothing needs attention, reply with just: HEARTBEAT_OK"""  
  
# Token that indicates "nothing to do"  
HEARTBEAT_OK_TOKEN = "HEARTBEAT_OK"
```

它会先检查文件中是否包含可执行的内容，如果有，则将内容发送给Agent，让它执行，如果没有则直接返回一个ok就行了。

看起来和定时任务的功能相同。但是与定时任务的区别在于使用场景不同。

| 特性 | Heartbeat (心跳) | Cron (定时任务) |
| --- | --- | --- |
| **触发源** | **文件内容** (`HEARTBEAT.md`) | **时间规则** (Cron 表达式 / 间隔) |
| **执行内容** | **不确定** (取决于文件里写了啥) | **确定** (Payload 里预定义的 Prompt) |
| **持久化** | 无需持久化 (实时读文件) | 需要持久化 (`cron.json`) |
| **交互性** | **人机异步协作** | **机器自动化** |
| **成本控制** | **高** (文件空就不唤醒 LLM) | **低** (时间到了必须唤醒 LLM 执行) |
| **代码位置** | `nanobot/heartbeat/` | `nanobot/cron/` |

因为触发源是文件，所以在运行的任何时候，你都可以修改这个文件，临时增加要执行的内容到HEARTBEAT.md文件中。

心跳机制相当于AI“每隔一段时间看看有没有新指示”，而任务则是AI “每隔一段时间必须做某件具体的事”。

当然，你一定要用cron代替heartbeat，也是可以是，毕竟，这里的心跳不是传统意义上的心跳。你可以在cron里加上一个任务，这个任务就是把上面那段提示词提交给agentloop。但是缺点是有两个：

一是浪费token，本来用heartbeat的话，如果文件里没有新指示，你本可以不调用AI。

二是如果任务非常多，或者任务特别费时，如果运行时间超过30分钟，但cron时间一到又调用文件，重复执行，反而造成混乱。

但是用心跳机制则不用担心这个问题，因为任务执行时，

```
if self._running:  
   await self._tick()
```

它会停在 `await self._tick()` 这一行， 计时器暂停 。 只有在任务结束后再继续计时30分钟。

## 结语

看代码的部分，暂时告一段落。看代码是为了更好地理解它是怎么运作的，而了解运作机制的目的还是更好地应用。

总结一下，可以把它分成四个模块

### 1. 网关gateway

它是所有交互的入口。你可以从命令行，也可以从telegram、whatapp、飞书或其他聊天工具（需要自己开发接口）发消息，网关都会把这些不同平台的消息格式统一起来，然后交给后面的系统处理。核心就是AgentLoop，它常驻后台，负责身份验证、会话管理，还内置了定时任务调度。某种程度上，它就是整个系统的中枢。

### 2. 大模型LLM管理器

系统自己不是AI，它只这里提供接口，连到别人的大模型上。它做的事情其实是根据不同的情况，动态地生成提示词，把用户指令、可用技能、相关记忆和当前状态整合在一起发给模型。之所以说它是管理器，因为它其实是不仅仅是一个转发者，同时还是一个调度和决策者。

### 3. 记忆系统

用文件来保存记忆，会话日志用 JSONL 一行一条记录，方便查询。而且还有每日记忆和长期记忆，用来保存对用户的总结。再加上一个 SOUL.md，可以Agent的人格和设定。这样随着交互的增加，Agent的知识和个性会越来越明确。

### 4. 技能系统

这相当于Nanobot的手脚。SKILL.md文件里保存了技能，但在文件中技能只是告诉大模型在什么情况下调用哪个工具，以及参数怎么填。也就是说，技能是给模型看的使用说明书，工具则是实际干活的可执行代码。比如Python脚本。

这就是一个把AI从在线顾问变成一个可服务于人类的机器人的第一步啊。