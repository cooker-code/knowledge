> 已吸收至：[[02_Agent与AI工程/0207_Prompt Engineering/0207_核心知识点/Prompt任务契约与评估闭环|Prompt任务契约与评估闭环]]
---
title: 一个强大的AI提示词优化工具！成熟的AI应该学会自己动！
author: 喵开发
date:
url: http://mp.weixin.qq.com/s?__biz=MzU2MDc5NTQxNQ==&mid=2247484489&idx=1&sn=c51ceab3950346b652867d39ef5091cd&chksm=fd164c237f17c7a8821802da42dcbca69210b7f66d703ca723925abdeaa35331c8b7b3a2c404&mpshare=1&scene=24&srcid=03101YqhkoY5iI2NhvMuJsDu&sharer_shareinfo=c5ad394d032cf218453a725a5ae1e7a4&sharer_shareinfo_first=c5ad394d032cf218453a725a5ae1e7a4#rd
---

👉 点击上方蓝字「喵开发」关注我，不错过每一篇推送！

 

各位铲屎官大家好，我是喵~~

这段时间大家都感受到了AI技术的大爆炸带来的改变，AI已经成为区分工作能力的因素之一。

AI应用能力的关键就是能否写出优秀的提示词，如果我们自己写不好提示词，为什么不让AI帮你写呢？

试试Prompt Optimizer[1]——让你写提示词像喝快乐水一样简单！

---

## 核心优势

**Prompt Optimizer** 是一个专治AI提示词编写困难的「赛博老中医」，支持多种大模型：

* • **Web应用** + **Chrome插件** 双端出击（摸鱼/搬砖两相宜）
* • **智能优化引擎**：自动诊断提示词问题，支持多轮迭代改进
* • **隐私安全架构**：数据直连AI服务商，不当中间商赚差价
* • **一键部署**：支持Docker一键部署，自带跨域解决方案
* • **一键优化**：输入"帮我写周报"，自动升级为"你是一个资深项目经理，请用Markdown格式生成本周工作汇报..."
* • **数据链路**：浏览器 ⇄ AI服务商（中间不经过第三方服务器）

---

## 🚀 快速开始

### 方式一：Docker一键起飞

```
# 基础版（适合本地测试）
docker run -d -p 80:80 --name prompt-optimizer linshen/prompt-optimizer

# 进阶版（配置API密钥）
docker run -d -p 80:80 \
  -e VITE_OPENAI_API_KEY=sk-你的密钥 \
  -e VITE_GEMINI_API_KEY=AIza你的密钥 \
  --restart unless-stopped \
  --name prompt-optimizer \
  linshen/prompt-optimizer
```

### 方式二：Chrome插件（科技专属）

1. 1. 点击前往Chrome商店[2]安装
2. 2. 点击浏览器工具栏图标
3. 3. 随时随地优化提示词

---

## ⚙️ API密钥配置

1. 1. 点击右上角 **⚙️设置** -> **模型管理**
2. 2. 选择要配置的模型，这里我选择DeepSeek，原因是更便宜，而且对中文更友好
3. 3. 输入密钥 -> 保存

#### 引用链接

`[1]` Prompt Optimizer: *https://github.com/linshenkx/prompt-optimizer*
`[2]` Chrome商店: *https://chrome.google.com/webstore/detail/prompt-optimizer/your-extension-id*

---

📚 **推荐阅读：**

* • 《[star 34.6k！通过DeepSeek实现AI自动化操作浏览器！](https://mp.weixin.qq.com/s?__biz=MzU2MDc5NTQxNQ==&mid=2247484481&idx=1&sn=2a58b0ecf46193ef8b3f63ea167df3cd&scene=21#wechat_redirect)》
* • 《[star 31.8k！LLM 时代为AI打造的智能网络爬虫](https://mp.weixin.qq.com/s?__biz=MzU2MDc5NTQxNQ==&mid=2247484465&idx=1&sn=d6489f5a6affcf907368ca66b21688bc&scene=21#wechat_redirect)》
* • 《[粉丝强烈推荐：MinerU——将PDF转化为机器可读格式的神器](https://mp.weixin.qq.com/s?__biz=MzU2MDc5NTQxNQ==&mid=2247484457&idx=1&sn=1e0a2111acb5afd2a6b39a53a9a306a8&scene=21#wechat_redirect)》

---

 

👍 如果你喜欢这篇文章，别忘了**点赞**和**在看**哦！