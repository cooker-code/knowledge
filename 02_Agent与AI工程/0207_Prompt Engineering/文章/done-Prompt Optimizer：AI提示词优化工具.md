> 已吸收至：[[02_Agent与AI工程/0207_Prompt Engineering/0207_核心知识点/Prompt任务契约与评估闭环|Prompt任务契约与评估闭环]]
---
title: Prompt Optimizer：AI提示词优化工具
author: 学习软件知识
date:
url: http://mp.weixin.qq.com/s?__biz=Mzg3ODg3ODc4OA==&mid=2247484881&idx=1&sn=5282bf54fe733aeb73cfdb33a9e6e7c5&chksm=cee6e31bb93be7bacbf9ce10e9165caeed57667a0af9178efcd2fa20c97a3a3be2d9db75d171&mpshare=1&scene=24&srcid=04301SHY3hEkuULsLJgHRoDh&sharer_shareinfo=a715fd86cc4f000929aeef2b2a777842&sharer_shareinfo_first=a715fd86cc4f000929aeef2b2a777842#rd
---

## 📖 项目简介

**「Prompt Optimizer」** 是一个开源的AI提示词优化工具，旨在帮助用户编写更高效的AI提示词（Prompt），从而提升大模型（如OpenAI、Gemini、DeepSeek等）的输出质量。支持Web应用和Chrome插件两种使用方式，兼具易用性与灵活性。

---

## ✨ 核心特性

1. **「智能优化引擎」**

* 一键优化原始提示词，支持多轮迭代改进，显著提升AI回复的准确性与相关性。
* 提供优化前后的实时对比测试，直观展示改进效果。

2. **「多模型集成」**

* 支持主流AI模型，包括OpenAI（GPT-3.5/4）、Gemini、DeepSeek-V3等，并可扩展自定义API。

3. **「隐私与安全」**

* 纯客户端架构，数据直连AI服务商，不经过中间服务器，确保隐私安全。
* 本地加密存储历史记录和API密钥，避免敏感信息泄露。

4. **「跨平台支持」**

* Web应用（在线体验）和Chrome插件双端适配，满足不同场景需求。 https://prompt.always200.com

5. **「开发者友好」**

* 支持Docker快速部署，提供环境变量配置选项，便于本地或云端集成。

---

## 🛠️ 快速开始

### 1. 在线使用（推荐）

直接访问 Web版，无需安装即可体验。 https://prompt.always200.com

### 2. Chrome插件

从Chrome商店安装插件，随时优化浏览器中的提示词。

### 3. Docker部署

```
# 运行容器（默认配置）
docker run -d -p 80:80 --restart unless-stopped --name prompt-optimizer linshen/prompt-optimizer

# 运行容器（配置API密钥）
docker run -d -p 80:80 \
  -e VITE_OPENAI_API_KEY=your_key \
  --restart unless-stopped \
  --name prompt-optimizer \
  linshen/prompt-optimizer
```

### 3. Docker部署

```
# 1. 克隆仓库
git clone https://github.com/linshenkx/prompt-optimizer.git
cd prompt-optimizer

# 2. 可选：创建.env文件配置API密钥
cat > .env << EOF
VITE_OPENAI_API_KEY=your_openai_api_key
VITE_GEMINI_API_KEY=your_gemini_api_key
VITE_DEEPSEEK_API_KEY=your_deepseek_api_key
EOF

# 3. 启动服务
docker compose up -d

# 4. 查看日志
docker compose logs -f
```

## API配置

方式一：通过界面配置（推荐） 

1. 点击界面右上角的"⚙️设置"按钮 选择"模型管理"选项卡
2. 点击需要配置的模型（如OpenAI、Gemini、DeepSeek等）
3. 在弹出的配置框中输入对应的API密钥
4. 点击"保存"即可

支持的模型：

 OpenAI (gpt-3.5-turbo, gpt-4) Gemini (gemini-2.0-flash) DeepSeek (DeepSeek-V3) 自定义API（OpenAI兼容接口）

方式二：通过环境变量配置 Docker部署时通过 -e 参数配置环境变量：

```
-e VITE_OPENAI_API_KEY=your_key
-e VITE_GEMINI_API_KEY=your_key
-e VITE_DEEPSEEK_API_KEY=your_key
-e VITE_SILICONFLOW_API_KEY=your_key
-e VITE_CUSTOM_API_KEY=your_custom_api_key
-e VITE_CUSTOM_API_BASE_URL=your_custom_api_base_url
-e VITE_CUSTOM_API_MODEL=your_custom_model_name 
```

github地址：https://github.com/linshenkx/prompt-optimizer