> 已吸收至：[[02_Agent与AI工程/0207_Prompt Engineering/0207_核心知识点/Prompt任务契约与评估闭环|Prompt任务契约与评估闭环]]
---
title: 开发Prompt分享几例
author: 编程小杂铺
date:
url: https://mp.weixin.qq.com/s?__biz=MzIyMzI1NTMyNg==&mid=2247487769&idx=1&sn=0dca1feaf8b4d0ad0c1e4167bef0c5ac&chksm=e94fdcb22da50e55033e2532950bd7f461073ce2ccbff48d94e3e5075fef2afb51283bec295c&mpshare=1&scene=24&srcid=1110297ZwS9r4b3crJPhFetQ&sharer_shareinfo=62da54758019e4878b875e085589c771&sharer_shareinfo_first=62da54758019e4878b875e085589c771#rd
---

> ❝
>
> 多学多用。
>
> ❞

用 AI 确实能省一些力气。程序世界里的几乎一切，都可以使用上 AI。

## 例子

### 临时任务：根据数据生成数据示例

有一个 confluence 表格数据，想通过表格数据来生成对应的 json 字符串。以往这种临时任务往往需要编写脚本运行程序来得到，现在可以用 AI 直接生成最终结果。

提示词：根据文件 xxx.txt （从 conf表格复制，上传给 deepseek）生成一个 json 数据。 字段是每一个数字后的第一个值， 值是每一个数字后的最后一个值。

```
1   

datatime  

uint64  

事件发生的agent时间，

毫秒级别   

1608617924401

2   

datatype   

string   

大事件类型  

account
```

类似的任务： 给定一个 json 字符串，根据 json 字符串生成对应的结构体。

### 生成枚举

先搭一个架子和一个枚举。 然后要求 AI 遵照枚举示例补全所有枚举。

```
type AbnormalEventCategory string
AbnormalEventCategoryLogin             AbnormalEventCategory = "LOGIN"
// 其他枚举

type AbnormalEventCategoryInfo struct {
 Category AbnormalEventCategory
 Desc     string
}

var AbnormalEventCategoryMap = map[AbnormalEventCategory]AbnormalEventCategoryInfo{
 AbnormalEventCategoryLogin: {
  Category: AbnormalEventCategoryLogin,
  Desc:     "异常登录活动",
 },
```

然后在这里 tab,tab,tab ... AI 就会自动按照相同的格式生成所有枚举。

### 仿写代码

仿照现有例子写新代码，避免拷贝粘贴的问题（单词拼错，引用错误）。仿写能力可以说是 AI 的一个神器。可以直接让AI遵照项目现有做法，遵循规范，不需要写一堆遵循规则。例子里就含有规则。这里要注意的是，一定要用良好实践的例子。AI 举一反三的能力还是不错的。

仿照 list\_host\_monitor\_net\_connect 的 server.addRoutes, /list\_host\_monitor\_net\_connect\_whitelist.handler.go， /list\_host\_monitor\_net\_connect\_whitelist\_logic.go 模式，给形如 list\_host\_xxx\_whitelist\_topic.go ，都加上对应的 xxx\_req, xxx\_resp, xxx\_handler.go 和 servers.AddRoutes, list\_host\_monitor\_net\_connect已经加了就不用再写了。

这里可以 tab,tab,tab, ...

有一个，可以生成类似的 N 个。 何其省力！

‍

仿照 getHostMonitorConfig ，实现 list\_host\_xxx\_logic.go 里的 ListHostMonitorXXX 方法。

仿照 ListHostMonitorCreateEvilItem 给所有ListXXX 方法的 查表语句添加 offset 语句和根据 offset 查询。

### 相似批量修改

AI可以安全地做相似的批量修改，省点体力。修改多个时可能需要一点时间，这时候你可以去休息。

将 internel/app 下的 go 文件的文件名全部改成小写+下划线形式。

用 activities.go, fields.go 里的常量字段替代工程里使用的常量值。

将 getAbnormalEventDesc 里的的常量提取到 /internal/app/constants/fields.go 生成常量并替换成常量的引用。

将 类似 event[constants.FieldPname].(string) 都改成 GetStringFromEvent 的调用。

给 ListxxxDO 和  qt\_baseline\_xxx 表添加 Remark 和 Reason 字段。

### 单个改批量

提示词：自动登录有时非常频繁，calcInCheck 方法针对每一个登录事件处理，redis 读写比较频繁。优化 calcInCheck 方法，能够进行批量处理。实际上可以一次性把所有登录事件的 agentId 和 loginTimes 推入 redis 对应的 key（将多个redis操作合并为pipeline 操作） ，然后再取出进行判断。实现这种思路。

### 单测生成

仔细阅读 .cursur/rules/biz/ut.mdc ，给 HandleOneElement 添加单测， 追加到internal/ids\_response/test/operation\_utils\_test.go 末尾，不要修改其它不相干代码。

### 生成 API 文档

读取 routes.go 及引用的 Handler 代码，根据 xxxLogic 的  XXXReq,  XXXResp 字段定义及 routes 的 Path 等，输出一份 API 文档。要求包括：

1. 响应路径 URL
2. 请求参数
   字段名 - 字段类型
3. 返回结果
   字段名 - 字段类型

### 开发需求

GetDuration 实现当前距 startTime 有多大间隔，比如 3 天 2 小时 25 分。 实现这个函数。在 /internel/test/utils/ 写个单测验证这个函数。

现在有个需求，要求拿到每个主机的最近更新时间。

1. 最近更新时间通过最近发生异常时间和最近基线更新时间取最大值。
2. 最近发生异常时间，通过 agentId 从 doris.xxx\_event 表查到这个 AgentId  的所有异常事件的时间取最大值。注意，这里要求批量处理，意味着必须拿到指定 agentIds 的最近发生时间，得到 AbormalEventMaxTime[agentId] = abnormalEventMaxTime
3. 最近基线更新时间，通过 agentId 从 pg.[xxx\_login,xxx\_process\_create,xxx\_bash\_elevation,xxx\_abnormal\_script\_run,xxx\_network\_connect] 取出最大值。 注意，这里同样要求批量处理，即取出所有 agentIds 的这些表中记录的时间，然后针对每个 agentId 取对应记录的最大时间。得到 baseLineMaxTime[agentId] = baselineMaxTime
4. 针对每一个 agentId ，将对应的  abnormalEventMaxTime 和 baselineMaxTime 比较拿到最大值，得到 maxTime[agentId] = lastUdpateTime.
5. 将 lastUdpateTime 与当前时间比较，得到相对时间比如 X 天 Y 小时 Z 分前。 最终得到 relativeTime[agentId] = lastUpdateTime。 实现 getLastUpdateTime 方法‍

## 要点

写开发提示词，跟日常提示词不同。要求：专业、精确、结构化、丰富上下文，这样才能让 AI 尽可能快速逼近所期望的结果。

* 专业：尽可能使用常用专业术语，尤其是设计模式。比如用选项器模式实现多维查询条件，使用工厂模式和策略模式实现。必要的话，需要明确性能要求和相应的技术手段。
* 精确：精确描述需求，不要有含糊。输入是什么（给出示例），输出是什么（给出示例）。必要的话给出方法签名和输入输出示例。
* 结构化：用有内在逻辑性的 1,2,3,..., 串联起来表达完整的需求和要求。
* 丰富上下文：如果要做复杂任务，尽可能提供项目的总体思路、用到的常量、方法、服务、存储、工具、技术栈等
* 要求和期望：需要生成哪些内容，使用什么工具，达到什么标准。

当你发现 AI 走偏的时候，要及时提醒，加入新的上下文或者提示，让 AI 转回到正常的轨道上。指望写一段提示词，然后让 AI 能够一马平川地顺溜地把所有东西都生成且生成很合理，可能是痴心妄想。另外，AI 的输出可能会很啰嗦，AI 可能节省了你写的时间，却增加了你阅读的时间。毕竟，生成内容往往不是一次性的，后续还需要经常维护。‍

最后，同样重要的是，选择一个合适省心的模型。我用的是claude3.7。

Prompt 是一项表达性的活动。要多熟悉软件开发和设计里的语汇（批量、线程池、对象池、设计模式、架构模式等），才能写出更好的提示词。其次，熟能生巧。使用 AI 类似锻炼肌肉，多用才能更好。写好Prompt是使用AI的最基本功底。