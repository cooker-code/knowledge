> 已吸收至：[[02_Agent与AI工程/0201_Agent框架/020117_DataAgent/020117_核心知识点/Jupyter数据分析Agent工作流边界|Jupyter数据分析Agent工作流边界]]
---
title: 不用招数据分析师了！GitHub炸星项目「AI Data Science Team」10倍速搞定清洗建模，一个人就是一支队伍
author: 吾想
date:
url: https://mp.weixin.qq.com/s?__biz=MzU4Nzk3NTc4MA==&mid=2247484500&idx=1&sn=e45743f7d77ca7235c87d0f54d9fd1d5&chksm=fcacd12bee31162cc8e32d45997e2e405efa15e3e666370e6c5835dce9cf4c8cc93315cc057e&mpshare=1&scene=24&srcid=1229HQEqjvbiY3QuabILwNHj&sharer_shareinfo=0ac21af34469c091682366e50476fddb&sharer_shareinfo_first=0ac21af34469c091682366e50476fddb#rd
---

还在一个人通宵写EDA、调特征、跑模型？最近在GitHub上发现了一个新鲜的项目（目前已3.2K Star数）「AI Data Science Team」。把数据科学全流程打包成一支AI特工队——数据清洗、可视化、特征工程、AutoML、MLflow追踪全流程自动跑，10分钟交付一份可复现报告，老板直呼“招人多余”！

## 它到底是个啥？

一句话：**一支用Python搭起来的“AI数据科学团队”**

* 10个专属Agent：数据加载、清洗、可视化、特征工程、SQL、AutoML、MLflow...每人专职一件事
* 1个「AI Pipeline Studio」：Streamlit可视化拖拽界面，点几下就能跑完完整建模流程
* 全程代码可追溯 + MLflow实验管理，再也不怕“谁改了我的模型”
  MIT协议，商用0门槛，star数3天破千，评论区全是“真香”！

## 10分钟体验全流程：从CSV到模型只需3行命令

## ① 克隆

```
git clone https://github.com/business-science/ai-data-science-team.gitcd ai-data-science-teampip install -e .
```

② 启动可视化工作室

```
streamlit run apps/ai-pipeline-studio-app/app.py
```

浏览器自动打开→上传CSV→勾选要跑的Agent→点**Run**，去冲杯咖啡回来就能看到：

* 自动清洗日志 ✅
* EDA图表文件夹 ✅
* 排行榜（AUC最高模型已标绿）✅
* 一键下载pkl+代码 ✅

## Agent能力速览

|  |  |  |
| --- | --- | --- |
| Agent | 代表技能 | 一键交付物 |
| **Data Loader** | 支持CSV/Excel/SQL/Parquet | 标准化DataFrame + 数据字典 |
| **Data Cleaning** | 缺失值、异常值、重复行自动识别 | 清洗脚本 + 清洗报告 |
| **Visualization** | 交互式Plotly、seaborn | HTML版EDA报告 |
| **Feature Engineering** | 时间滑窗、交叉特征、target encoding | 新特征矩阵 + 重要性排名 |
| **H2O AutoML** | 20+算法+网格搜索 | Leaderboard + 最佳模型pkl |
| **MLflow** | 参数、指标、模型、数据版本 | 可复现实验页面 |
| **SQL Agent** | 自然语言→SQL→结果 | 直接用中文问“上个月销量TOP3是谁” |

## 本地大模型也能跑！

支持**OpenAI**和**Ollama**双后端：

```
# 本地8G显卡即可from langchain_ollama import ChatOllamallm = ChatOllama(model="llama3.1:8b")
```

公司内网无API也能用，数据不出本地，安全合规。

## 适用场景（已验证）

* **中小企业**：没预算请全职数据科学家，实习生+该项目=月报+预测模型一条龙
* **外包团队**：提案阶段10分钟跑出原型报告，客户现场直呼专业
* **学生/研究者**：课程大作业直接生成可复现代码+图表，导师再也不要求补EDA
* \*\* teaching/training\*\*：工作坊演示AutoML流程，学员看得懂点得到

作者已开放**Next-Gen AI Workshop （官网：learn.business-science.io/ai-register）**

**项目地址：https://github.com/business-science/ai-data-science-team**

**历史好文：**

**[多代理系统架构深度解析：Supervisor 与 Swarm 如何重塑 AI 复杂任务处理？](https://mp.weixin.qq.com/s?__biz=MzU4Nzk3NTc4MA==&mid=2247484179&idx=1&sn=4f9ae61e55a06d7eefb96bed26898be0&scene=21#wechat_redirect)**

**[快来查看你的ChatGPT年度报告吧！](https://mp.weixin.qq.com/s?__biz=MzU4Nzk3NTc4MA==&mid=2247484426&idx=1&sn=829707dbf8be50f3308af56ef24bc94b&scene=21#wechat_redirect)**

**[先Embedding再Chunking！RAG分块新范式火了， semantic chunking 让检索精度飙升！](https://mp.weixin.qq.com/s?__biz=MzU4Nzk3NTc4MA==&mid=2247484291&idx=1&sn=484ba7ee97391dd879c4e78a59f213ac&scene=21#wechat_redirect)**

**[00后程序员靠「氛围编程」月入5万：不刷LeetCode，不卷算法，全靠GitHub这个「军火库」](https://mp.weixin.qq.com/s?__biz=MzU4Nzk3NTc4MA==&mid=2247484285&idx=1&sn=28cbddc604d36a1317bd6cd2c5e490bd&scene=21#wechat_redirect)**

**[C盘只剩1G，我用AI“暴力”清出15.9G空间！ Trae这波操作太离谱了](https://mp.weixin.qq.com/s?__biz=MzU4Nzk3NTc4MA==&mid=2247484280&idx=1&sn=b9b46d274e50236fb1961aae90525117&scene=21#wechat_redirect)**

**[Claude Skills 不是文件夹,而是 AI 时代的「岗位说明书」](https://mp.weixin.qq.com/s?__biz=MzU4Nzk3NTc4MA==&mid=2247484213&idx=1&sn=924e22fd55f66c36003778463e845cb0&scene=21#wechat_redirect)**