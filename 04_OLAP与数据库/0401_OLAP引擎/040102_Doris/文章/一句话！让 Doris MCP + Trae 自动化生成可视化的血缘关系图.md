---
title: 一句话！让 Doris MCP + Trae 自动化生成可视化的血缘关系图
author: 一臻数据
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650488198&idx=1&sn=a40018a247c14970d784f82c59a7595e&chksm=f28ca66e628585999dfa773424fe22ac87f3b851bedea3453c50d4588cbe393a4606325b7338&mpshare=1&scene=24&srcid=0803tVM2wiNw6qET7OECpjAP&sharer_shareinfo=5e61bee4403894321903fb36b7946812&sharer_shareinfo_first=aaa1e4b9e0f0c4ab202c0c3b5d6fc947#rd
---

见字如面，我是一臻 90后新手奶爸，探索Doris x AI

点击关注 👇 免费获取数字AI知识库

> ❝
>
> 半夜，小明正呼呼大睡 💨 突然被一个紧急电话惊醒："数据有问题！用户投诉订单金额不对！" 
>
> 他揉着惺忪的睡眼，打开电脑，面对着密密麻麻的Doris表，开始了一场"`侦探游戏`"——这个库的表之间有什么关联？那个表的key字段和其它表有什么关系？
>
> **数据血缘**追踪，听起来很高大上，做起来却像在迷宫里找出口。传统的方式需要你一层层翻Schema，一步步查记录，就像用放大镜拼凑一幅巨大的拼图。运气好的话，半天能搞定；运气不好，可能要花上几天时间。 
>
> 但现在，只需一句话，就可以让 **Doris MCP + Trae** 自动化生成可视化的血缘关系图...

## 环境准备

听起来很酷，实现起来也比你想象的简单多了多。

在实测之前，我们需要准备如下环境：

**一、Python**

Python 需要至少 **3.12+** 的版本，可以使用pip、uv、conda等工具进行Python包管理。

个人常用的是conda管理，常用命令如下：

```
# python环境创建  
conda create -n 虚拟环境名 python=对应的python版本  
# 举栗  
conda create -n py312 python=3.12  
  
# 查看已经存在的py虚拟环境  
conda env list  
  
# 激活即切换至对应的py环境  
conda activate py312  
  
# 退出当前py虚拟环境  
conda deactivate  
  
# 查看当前py环境已有的py包  
conda list
```

**二、Apache Doris**

本文选择的 Apache Doris 版本为：2.1.10。

实测中可以选择自己环境中的库表，或者使用 Star Schema Benchmark(SSB)星型模型数据集🔗：`https://doris.apache.org/zh-CN/docs/benchmark/ssb`

**三、Doris MCP Server**

由于本次实测是跟IDE结合，所以推荐直接用git clone到本地的方式：

```
# 1. MCP Server 克隆到本地：  
git clone https://github.com/apache/doris-mcp-server.git  
cd doris-mcp-server  
  
# 2. 安装依赖：  
conda activate py312  
pip install -r requirements.txt  
  
# 3. 配置数据库信息  
cp .env.example .env  
vim .env  
  
## Doris FE connection settings  
DORIS_HOST=xxx  
DORIS_PORT=9030  
DORIS_USER=root  
DORIS_PASSWORD=xxx  
DORIS_DATABASE=xxx  
  
# 4. 启动doris mcp server服务  
cd /本地仓库的路径  
./start_server.sh &
```

出现如下日志`Uvicorn running on`则表示启动成功：

**四、Trae IDE**

推荐用海外版（默认可用Claude-4-Sonnet）🔗：`https://www.trae.ai/`

下载客户端后直接点点点，挂好🪜登录即可：

随后进行Doris MCP Server配置：

**1.** 手动添加MCP

**2.** 输入本地启动的Doris MCP Server配置：

```
{"mcpServers": {"doris_mcp_server": {"transport": "streamable_http", "url": "http://localhost:3000/mcp"}}}
```

**3.** 出现 ✅ 和 MCP Tool 列表则表示配置成功

并创建一个智能体进行Doris  MCP Server关联：

提示词如下(提示词过长，可后台发送 **0803DorisMCPCase** 自动获取)：

```
<role>  
你是一位专业的数据血缘分析专家，专精于Apache Doris环境下的数据血缘追踪和可视化。你的核心使命是帮助用户构建完整、准确、实时的数据血缘关系图，实现企业级数据治理。  
</role>  
  
<expertise>  
- 数据血缘关系发现与分析  
- 字段级和表级依赖关系追踪    
- 血缘可视化图形生成  
- 数据流转路径优化  
- 变更影响评估分析  
- 血缘质量监控与维护  
</expertise>  
  
<core_capabilities>  
......
```

## 实测效果

纸上得来终觉浅，我们来看看真实的测试效果👇

真就只需输入一句话，基于 **Doris MCP Server** 即可获得**库-表-字段的血缘关系图**：

```
请帮我梳理ssb数据库中每张表和相关字段的数据血缘关系，用适合的图形结合HTML清晰展现表间依赖、字段映射和数据流向。
```

整个过程不到5分钟，得到的血缘结果比手工分析的还要详细。

结果中不仅有血缘概览、星型模式血缘关系图、还有表结构关联的详情，如果有不满意之处还可以随时发送指令进行调整，可谓 Very Nice !

⚠️ 如果想输出更专业的图表展现，可以结合**AntV（MCP Server Chart）**🔗：`https://github.com/antvis/mcp-server-chart`

## 结语

MCP（Model Context Protocol）听起来很技术，其实就是AI和数据库之间的"`翻译官`"。

过去，AI只能听懂人话，数据库只会说机器语言，两者像是隔着一堵墙在对话。MCP的出现，让这堵墙瞬间消失了。

Doris MCP + Trae 这类的组合，其中的意义远超工具层面的改进。它代表着一种全新的 Data + AI 思维方式。它让数据不再是冰冷的数字，而是有温度的AI智能伙伴。它知道你的需求，理解你的困惑，并且随时准备为你提供最贴心的服务。

**诸如此类的 Doris + AI 新范式才刚刚开始，你准备好了吗？**

**完**

---

👇欢迎扫描下方二维码 👇 

备注 **666**免费领取资料  加入Doris官方群和PowerData数据社区❗️

#大数据 #开源 #OLAP #Doris #MCP #Trae #人工智能