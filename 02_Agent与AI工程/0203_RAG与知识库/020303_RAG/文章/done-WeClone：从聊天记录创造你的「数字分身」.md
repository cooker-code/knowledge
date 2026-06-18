> 已吸收至：[[02_Agent与AI工程/0203_RAG与知识库/020303_RAG/020303_核心知识点/RAG生命周期与评估边界|RAG生命周期与评估边界]]
---
title: WeClone：从聊天记录创造你的「数字分身」
author: 为郎
date:
url: https://mp.weixin.qq.com/s?__biz=MzI0MjIzMDU1OA==&mid=2650853669&idx=1&sn=f3dc4ba6d83185fb8e466175a9af2ea5&chksm=f3caf55f7109b750f02985b4b2044c1bd8728e03c4fa8e523c423eb6f1884e120eea332e4af5&mpshare=1&scene=24&srcid=04227bGA8Ti5JkVeSMOOhPK7&sharer_shareinfo=d7a3c05334b81f7ea78bd3bfc94c8e62&sharer_shareinfo_first=d7a3c05334b81f7ea78bd3bfc94c8e62#rd
---

你是否想过，如果有一个AI能模仿你的说话风格、记住你的习惯用语、甚至代替你聊天……

**WeClone**就是这样一个开源项目——它能分析你的聊天记录，训练出「另一个你」。

## 🎯 一句话介绍

WeClone 是一站式数字分身解决方案，通过分析你的聊天记录微调大模型，让AI学会「像你一样说话」。

## ✨ 核心功能

| 功能 | 说明 |
| --- | --- |
| 💬 **聊天记录微调** | 支持文字、图片，让大模型有"那味儿" |
| 🔗 **多平台绑定** | Discord、Telegram、Slack、飞书等 |
| 🛡️ **隐私保护** | 本地化部署，数据安全可控 |
| 🎨 **全链路方案** | 数据导出 → 预处理 → 训练 → 部署 |

## 📊 支持情况一览

### 数据源适配

| 平台 | 文字 | 图片 | 语音 | 视频 |
| --- | --- | --- | --- | --- |
| Telegram | ✅ | ✅ | ❌ | ❌ |
| WhatsApp | 🚧 | 🚧 | 🚧 | 🚧 |
| Discord | 🚧 | 🚧 | 🚧 | 🚧 |

### 部署平台

| 平台 | 状态 |
| --- | --- |
| 个人微信 | ✅ (基于 openclaw-weixin) |
| Telegram | ✅ |
| Discord | ✅ |
| Slack | ✅ |

## 🚀 快速上手

### 环境要求

> 💡 默认使用 Qwen2.5-7B-Instruct 模型 + LoRA 方法，约需 **16GB 显存**

显存需求参考：

| 方法 | 精度 | 7B模型 | 14B模型 | 30B模型 |
| --- | --- | --- | --- | --- |
| Full (bf16) | 16 | 60GB | 120GB | 300GB |
| LoRA/QLoRA | 8 | 10GB | 20GB | 40GB |
| QLoRA | 4 | 6GB | 12GB | 24GB |

> ⚠️ **7B模型效果一般，14B及以上效果更好**

### 安装步骤

**Step 1：克隆项目并安装依赖**

```
git clone https://github.com/xming521/WeClone.git
cd WeClone
uv venv .venv --python=3.12
source .venv/bin/activate  # Windows: .venv\Scripts\activate
uv pip install --group main -e .
```

**Step 2：配置项目**

```
cp settings.template.jsonc settings.jsonc
```

**Step 3：验证CUDA环境**

```
python -c "import torch; print('CUDA是否可用:', torch.cuda.is_available())"
```

## 📋 完整训练流程

### 1️⃣ 准备数据（Telegram为例）

1. 使用 Telegram Desktop 导出聊天记录
2. 选择照片类型，格式选 **JSON**
3. 将导出的文件夹放入 `./dataset/telegram`目录

### 2️⃣ 数据预处理

> ⚠️ 项目会自动过滤电话号码、邮箱、地址等隐私信息，但**无法保证100%过滤**，请注意保护个人信息！

```
# 修改 settings.jsonc 中的 language、platform、include_type
weclone-cli make-dataset
```

### 3️⃣ 微调模型

```
# 单卡训练
weclone-cli train-sft

# 多卡训练
uv pip install "deepspeed<=0.16.9"
deepspeed --num_gpus=你的显卡数量 weclone/train/train_sft.py
```

### 4️⃣ 测试效果

```
# 浏览器Demo测试
weclone-cli webchat-demo

# 或启动API服务
weclone-cli server
```

## 🤖 部署到聊天机器人

### 方式一：AstrBot

✨ 支持 Telegram、飞书等多平台

**步骤：**1. 部署 AstrBot 2. 执行 `weclone-cli server`启动API服务 3. 在 AstrBot 新增服务提供商（类型选择OpenAI） 4. 填写 API 地址和模型名称

> ⚠️ **重要**：微调后需关闭工具调用，发送指令 `/tool off_all`

### 方式二：LangBot

支持接入全球多种即时通讯平台

## 🛤️ 路线图

* 🚧 支持更多数据源
* 🚧 更丰富的上下文（对话历史、聊天对象信息、时间）
* 🚧 Memory 支持
* ✅ 多模态（已支持图片）
* 🚧 数据增强
* 🚧 GUI 支持
* 🚧 COT思考

## 💡 在线微调

没有高端显卡？试试 **大模型实验室 (Lab4AI)**：

🔗 

```
https://www.lab4ai.cn
```

> 🎁 新用户送50元代金券

## 📚 更多资源

* 📖 **项目文档**：https://github.com/xming521/WeClone
* ⭐ **GitHub**：搜索 xming521/WeClone
* 💬 **社群**：有部署好的 Qwen2.5VL 32B Bot 可体验

## ⚠️ 重要提示

1. WeClone **仍在快速迭代**，当前效果不代表最终效果
2. 效果取决于 **模型大小、聊天数据的数量和质量**
3. Windows 环境未严格测试，建议使用 **WSL**
4. 请务必注意 **保护个人隐私**，不要泄露个人信息！

## ❤️ 欢迎贡献

* 🐛 提交 Issue
* 🔧 提交 Pull Request
* 💬 新功能请先通过 Issue 讨论

```
# 开发环境
uv pip install --group dev -e .
pre-commit install
pytest tests  # 运行测试
```

*如果对你有帮助，别忘了去 GitHub 点个 ⭐ ！*

**项目地址**：

```
github.com/xming521/WeClone
```

 **许可证**：开源免费

更多：

[【赛博永生】张雪峰.skill](https://mp.weixin.qq.com/s?__biz=MzI0MjIzMDU1OA==&mid=2650853654&idx=1&sn=68ee188046925db77915a93c09b594ea&scene=21#wechat_redirect)

[Claude Code × 飞书CLI 完整教程](https://mp.weixin.qq.com/s?__biz=MzI0MjIzMDU1OA==&mid=2650853579&idx=1&sn=8690ff0c862fbba0eb58cdf5385409f5&scene=21#wechat_redirect)

[WeWrite：一个让公众号创作者「躺平」的 AI 神器](https://mp.weixin.qq.com/s?__biz=MzI0MjIzMDU1OA==&mid=2650853562&idx=1&sn=81740764cb7bc89e8f753ed575b3e06d&scene=21#wechat_redirect)