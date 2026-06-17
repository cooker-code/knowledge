---
title: 0代码！教会你用Doris+DeepSeek+Dify搭建ChatBI系统（附完整DSL）
author: 一臻数据
date: 
url: http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650487687&idx=1&sn=1e34b95a0acbe4b4868db894d8affb3e&chksm=f2a7c878627cb05fc310a9d2e829639b4a64b6245a568eebc389ad0df123d132bf2201260d39&mpshare=1&scene=24&srcid=0424FYOunIZU6L7tgLL04bZg&sharer_shareinfo=659b68800b404135b229771a2b8bc032&sharer_shareinfo_first=659b68800b404135b229771a2b8bc032#rd
---

点击上方 蓝字 关注一臻数据👆 

免费领取 DeepSeek➕数据AI知识库 🔗 一起共建共进

> ❝
>
> 晚间7点半，A总风风火火地闯进Doris社群："可以结合Dify做个Doris ChatBi？" 
>
> "好滴，安排！" 
>
> 今天，来一起学习如何用Doris+DeepSeek+Dify，≈0代码搭建一套Doris ChatBI系统。 
>
> 另外，4/1 今晚还有 `Doris x AI 的专场直播`！分享与 `ChatBI、MCP、RAG` 热门 AI 模型的融合，欢迎大家来聊👇

## 前言

`Apache Doris`作为一款基于 MPP 架构的高性能、实时分析型数据库，`DeepSeek`作为国产Top的大语言模型，`Dify`作为88K✨的AI应用开发平台，三者合力打造的ChatBI系统，让对话式BI变得触手可及。

整个Doris ChatBI系统工作流程非常清晰：

`用户提需求 → DeepSeek进行Text2SQL → Doris执行查询 → DeepSeek分析 → 可视化BI展示`

接下来，直接实战体验 ⬇️

## 从0搭建ChatBI系统

##### 步骤一：准备Doris环境并导入数据

首先安装Apache Doris，并生成导入TPC-H数据。这个数据集包含了**客户、订单和供应商**等维度的核心表，非常适合演示ChatBI功能。

🔗 Doris环境部署：`https://doris.apache.org/zh-CN/docs/install/deploy-manually/integrated-storage-compute-deploy-manually`

🔗 TPC-H数据导入：`https://doris.apache.org/zh-CN/docs/benchmark/tpch`

##### 步骤二：准备Dify环境

可以基于Docker搭建：`https://github.com/langgenius/dify`

也可以直接使用Dify Cloud：`https://cloud.dify.ai/apps`

环境初始化后，`创建空白应用`，选择`Chatflow`并进行简单配置后`创建`。

##### 步骤三：Dify流程编排

Dify实现的Doris ChatBI流程如下：

1️⃣ **Input**

需求输入，由对话窗口输入，不用做额外的配置操作。

2️⃣ **Text2SQL**

LLM节点，本次配的是`DeepSeek V3`，主要作用是定义自然语言转SQl的`核心规则`、`数据库表信息映射`、`查询技巧`、`查询示例`、`注意事项`和`输出格式`。

例如（演示版，详细配置见`DSL`附件）：

```
# 你是数据分析专家，精通Apache Doris，能够根据用户的问题生成高效的SQL查询， 详细规则如下  
  
## 核心规则  
1. 仅使用提供的表和字段  
2. 确保SQL语句兼容Doris语法  
3. 输出一个完整的SQL语句，无注释  
...  
  
## 数据库表结构（TPC-H 决策支持基准）  
### 1. customer（客户表表）  
...  
  
## 查询技巧  
### 聚合函数  
- COUNT(): 计算数量  
- AVG(): 计算平均值  
- SUM(): 计算总和  
- MAX()/MIN(): 获取最大/最小值  
...  
  
## 查询示例  
### 1. 客户订单分布数量查询  
...  
  
## 注意事项   
1. 合理使用JOIN条件  
2. 注意日期格式的一致性  
3. 使用适当的sql语句以提高查询效率  
  
## 输出格式  
1. 只能输出一个结果的sql语句  
2. 其它非sql内容必须过滤掉再输出
```

3️⃣ **SQL Formatting**

由于LLM生成的结果可能会带换行符或者一些强行解释文字导致SQL没法直接执行，所以通常会有做一些SQL格式化动作：

```
import re    
  
def main(text2sql: str) -> dict:  
    text2sql = text2sql.replace('```sql\n', ' ').replace('\n```', ' ').replace('\n', ' ').strip()  
    text2sql = re.sub(r'(LIMIT \d+;).*', r'\1 ', text2sql, flags=re.IGNORECASE)  
    return {  
        "text2sql": text2sql,  
    }
```

4️⃣ **Doris Execute**

这块可以直接用`Database`插件的`SQL Execute`，但需要在安装完插件后，配置授权一下可通信的Doris集群URL，例如：

```
mysql+pymysql://{user}:{passwd}@{fe_ip}:9030/tpch
```

5️⃣ **Json Result**（可选）

查询结果进行JSON格式规范：

```
{{ json_result }}
```

6️⃣ **Doris ChatBI**

LLM节点，配的是`DeepSeek V3`，主要作用是定义Doris查询结果转可视化BI的`核心规则`、`可视化图表Echart格式定义`、`处理流程`、`分析维度`、`注意事项`和`输出格式`。

例如（演示版，详细配置见`DSL`附件）：

```
# Doris ChatBI数据分析专家工作指南  
  
## 核心规则  
1. 直接分析已提供数据，默认数据已满足查询条件。  
2. 整理SQL查询结果：  
   - 以Markdown表格格式输出，放置在输出开头。  
   - 以ECharts图表配置项格式输出，放置在最后。图表配置应尽量简洁，避免过多冗余配置项。  
...  
  
## 数据处理流程  
1.接收JSON格式查询结果  
2.验证数据完整性  
3.进行统计分析  
4.生成分析报告  
  
## 常见分析维度  
1.订单分析  
- 订单数量  
- 订单分布  
- 订单趋势  
  
2.客户分布  
- 下单数量  
- 地区分布  
- 消费分布  
...  
  
## 特殊情况处理  
- 空数据集：直接返回"没有查询到相关数据"  
- 异常值：如实报告，不作主观判断  
- 数据缺失：说明缺失情况，不补充假设数据  
  
## 输出格式  
如果上游数据库查询没有结果，则直接结合echarts返回 一个空白图，图中告知：没有查询到相关数据；  
如果有数据则结合echarts，将数据用适合的图形进行可视化展示
```

6️⃣ **Result**

`直接回复`节点，返回上一步的结果直接输出到对话框。

至此，基于`Doris+DeepSeek+Dify`，搭建完成了一套≈0代码的`Doris ChatBI`系统：

## 结语

Doris ChatBI的魅力不仅在于简化数据获取，更在于能够非常直观地改变了人与数据的交互方式。从"`我需要写SQL`"到"`我想知道答案`"，它让数据分析回归本质—**解决业务问题**。

一位Doris用户形象地总结道："`以前Doris数据分析像是在翻译两种语言，虽然快但是绕。而现在，我可以直接用母语和数据对话。`"

但还不够，Doris ChatBI的未来：**不只是查询工具**。

今晚直播间，除了会分享Data Agent、RAG和ChatBI与Doris的结合，还会带来**Doris MCP Server & Client**的设计、应用及展望，敬请期待！

完整DSL文件可以私信或进Doris x AI群@一臻获取👇

**完**

---

一臻数据致力于大数据AI时代的前沿内容分享，会持续分享更多有趣有用有态度的知识。同时也欢迎大家**投稿，共建共进**，帮助圈友们冲破认知壁垒，实现自我提升！

另外，整理了份 **一臻数据知识库**，其中包含 **Apache Doris**和**Data+AI****的学习资料、学习课程、白皮书、研究报告、行业标准**和**实践指南** 等内容，会持续更新，欢迎**关注公众号，免费领取**。

🔗 欢迎扫描下方二维码 ⬇️ 备注 **666**免费领取资料  加入Doris官方群和全球最活跃的PowerData数据社区❗️

---

往期推荐

[*走进*开源，拥抱开源](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650483656&idx=1&sn=300e90f5017ebb3d97d3e98d26d52ff7&chksm=f374e9aec40360b85c87a26d9d1af93b2807ad1c54340676ce3173fb0508b3be7ca9595182f0&scene=21#wechat_redirect)

[*大数据*平台开发规范示例](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650482778&idx=1&sn=6a4a6b3bf16ab818aa38d222ce46fed6&chksm=f374ea3cc403632a0f6ef1a9728393b459c3d19926f8e9672467f278e9f56abb010b198d2b34&scene=21#wechat_redirect)

[*大数据*仓库开发规范示例](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650483237&idx=1&sn=824d2125280a009dddeec3f0aa60c4f6&chksm=f374e843c40361551cbbf48c7ad58fb054246a1abc5a60166a29aa7c7a547903848873149902&scene=21#wechat_redirect)

[*Flink CDC* 1.0至3.0回忆录](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650483092&idx=1&sn=396c5873c5b2bf6d66532daeae4b445f&chksm=f374ebf2c40362e4ebac29580dcb7add14c9980290769de9366924d98ad27b3e5dc3f1095613&scene=21#wechat_redirect)

[3分钟！教会你用Doris+*DeepSeek*搭建RAG知识库（喂饭级教程）](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650487529&idx=1&sn=fb6a4c74e6d159d2695be7b8bf4e2b36&chksm=f374f88fc403719945a51d78fe56a2ebe8c264347cf8d56619d2757ffad77dec2a2972c7268a&scene=21#wechat_redirect)

[3步！教会你用Doris+*DeepSeek*搭建ChatBI系统（保姆级教程）](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650487577&idx=1&sn=6e7c50529a2c42cf3955570180b8a5d2&chksm=f374f97fc40370692ec22c265976448c1e90b69129cbe9a1802f824efec427aeafa56a89d8dd&scene=21#wechat_redirect)

[全网最全Doris+*DeepSeek*使用手册（客服/图表/PPT/贺岁诗）！学会了Doris熟练度提高90%【建议收藏】](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650487419&idx=1&sn=e06c84f7cfb006b7818aa572aa8a1b55&chksm=f374f81dc403710b0e5961a02f21fc7ae04d0fbf79ae6114acc07b47861cd9aae907e33148a8&scene=21#wechat_redirect)

[*深夜*无需加班，Apache Doris让数据自己会跑](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650485454&idx=1&sn=a73a4a6e78b610cdc47d5e31293d871a&chksm=f374f0a8c40379be78b40cab31721c31f485af5272299ef7f8dbfdd30cff7dc6fcc27b49eb4e&scene=21#wechat_redirect)

[我用*X2Doris*干翻了3000张表，老板还以为我组了个团队](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650486392&idx=1&sn=f5bc442775d9478e64c13189f05c96e2&chksm=f374f41ec4037d084781c5072fe63b2338269b2530eb060091a136362d5f5c5a5716eba2302a&scene=21#wechat_redirect)

[*超强*满血不收费的AI绘图教程来了（在线Stable Diffusion一键即用）](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650482860&idx=1&sn=ad75b434b7eae3511017c67d14dad143&chksm=f374eacac40363dc192372fbdd36ff76ad8cbdd866df5a3287c3e48476f4c9c88f2ef3e5f3db&scene=21#wechat_redirect)

点击下方蓝字关注一臻数据