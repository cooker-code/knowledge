---
title: Github-LLaMA Factory：百种大语言模型一站式高效微调平台
author: Zzz小生
date:
url: https://mp.weixin.qq.com/s?__biz=Mzk2NDQ3ODc4Mw==&mid=2247484769&idx=1&sn=d2fbaffa004f998371e07ad89d0485f7&chksm=c509c4dca392f4b44a7a6751a8dd567086b870c200ae4f7456789341da9c40a7d1bc2cc5f49a&mpshare=1&scene=24&srcid=0106yvQ9YaRf8l2h1Jx0NFZc&sharer_shareinfo=b10729e7dd96ac9bf37d6e02bc75dc7c&sharer_shareinfo_first=b10729e7dd96ac9bf37d6e02bc75dc7c#rd
---
> 已吸收至：[[01_LLM与大模型/0104_微调与训练/0104_核心知识点/微调训练数据与方法边界准则|微调训练数据与方法边界准则]]

Github-LLaMA Factory：百种大语言模型一站式高效微调平台

```
https://github.com/hiyouga/LLaMA-Factory
```

## 1. 项目的主要功能和目的

**LLaMA Factory** 是一个功能强大、易于使用的开源框架，旨在**简化大型语言模型（LLMs）的微调过程**。其核心目标是让研究人员和开发者能够以极低的代码成本（甚至零代码），轻松地对超过100种主流大语言模型进行各种高效的微调。

**主要目的包括：**

* **降低微调门槛**：提供命令行工具（CLI）和图形化Web界面（Web UI），让没有深厚编程背景的用户也能进行模型微调。
* **支持广泛的模型和任务**：覆盖从LLaMA、Qwen、DeepSeek到多模态模型（如LLaVA、Qwen-VL）等众多架构，支持预训练、指令微调、奖励建模、偏好对齐（DPO、KTO、ORPO）等多种任务。
* **提升训练效率和资源利用率**：集成多种先进的优化算法和低资源适配技术（如LoRA、QLoRA、GaLore等），使得在消费级GPU上微调超大模型成为可能。
* **提供完整的工具链**：从数据准备、模型训练、评估到部署（支持OpenAI风格API、vLLM加速），提供端到端的解决方案。

## 2. 使用的主要技术或编程语言

* **核心编程语言**：Python
* **深度学习框架**：PyTorch
* **关键依赖库**：

+ `transformers` (Hugging Face)：模型加载与处理。
+ `datasets` (Hugging Face)：数据集管理。
+ `accelerate`：分布式训练。
+ `peft`：参数高效微调（LoRA等）。
+ `trl`：强化学习与对齐训练（PPO, DPO）。
+ `bitsandbytes` / `hqq` / `gptq` / `aqlm`：量化支持（QLoRA）。
+ `flash-attention-2` / `unsloth`：训练加速。
+ `vllm` / `sglang`：推理加速。
+ `gradio`：构建Web UI。

* **部署与容器化**：Docker
* **配置管理**：YAML文件

## 3. 项目的结构概览

```
LLaMA-Factory/
├── src/                    # 核心源代码
│   ├── llamafactory/      # 主包目录
│   │   ├── data/          # 数据处理和模板
│   │   ├── engine/        # 训练、评估引擎
│   │   ├── extras/        # 常量、枚举等
│   │   ├── model/         # 模型相关代码
│   │   └── train/         # 训练流程控制
│   ├── api_demo.py        # OpenAI风格API服务
│   └── train_web.py       # Web UI启动脚本
├── data/                   # 示例数据集和数据集信息配置
├── examples/               # 丰富的配置示例（训练、推理、合并等）
├── docker/                 # Docker构建文件（支持CUDA、NPU、ROCM）
├── scripts/                # 实用脚本（如API测试）
├── tests/                  # 测试代码
├── assets/                 # 图片等资源
├── pyproject.toml         # 项目元数据和依赖
├── requirements.txt       # 依赖列表
└── README.md              # 项目详细说明
```

## 4. 项目核心代码及使用指南

### 核心代码模块

* \*\*`src/llamafactory/engine/trainer.py`\*\*：定义了各种训练器（SFT, RM, PPO, DPO等），是训练流程的核心。
* \*\*`src/llamafactory/model/patcher.py`\*\*：负责模型加载、适配和补丁应用（如FlashAttention、LoRA）。
* \*\*`src/llamafactory/data/template.py`\*\*：定义了不同模型的对话模板，确保输入格式正确。
* \*\*`src/llamafactory/hparams.py`\*\*：管理和解析所有训练超参数。

### 快速使用指南（三步微调Llama 3）

1. **安装**：

   ```
   git clone --depth 1 https://github.com/hiyouga/LLaMA-Factory.git
   cd LLaMA-Factory
   pip install -e ".[torch,metrics]" --no-build-isolation
   ```
2. **微调**（使用LoRA）：

   ```
   llamafactory-cli train examples/train_lora/llama3_lora_sft.yaml
   ```

   只需修改YAML配置文件中的 `model_name_or_path`（模型路径）和 `dataset`（数据集）即可开始训练。
3. **推理与对话**：

   ```
   llamafactory-cli chat examples/inference/llama3_lora_sft.yaml
   ```
4. **使用Web UI（零代码）**：

   ```
   llamafactory-cli webui
   ```

   在浏览器中打开界面，可视化地配置参数、启动训练、监控进度并进行对话测试。
5. **部署为API服务**：

   ```
   API_PORT=8000 llamafactory-cli api examples/inference/llama3.yaml infer_backend=vllm
   ```

   即可提供与OpenAI兼容的ChatCompletion API。

## 5. 潜在的用途或应用场景

* **学术研究**：快速实验不同微调算法、数据组合对模型性能的影响。
* **行业应用**：

+ **垂直领域模型定制**：为法律、医疗、金融等领域创建专业知识丰富的助手。
+ **多模态应用**：开发图像理解、视觉问答、文档信息提取等应用。
+ **工具调用与智能体**：微调模型使其能可靠地使用外部工具和API。
+ **内容生成与创作**：定制化生成营销文案、故事、代码等。

* **模型优化与压缩**：利用QLoRA等技术，在有限资源下优化大模型。
* **教育与入门**：作为学习大语言模型微调的理想实践平台。

## 6. 值得注意的特点或创新点

* **“Day-N” 模型支持**：对Llama 3、Qwen 2.5、Gemma 3等前沿模型实现发布当日或次日即支持。
* **算法集成先锋**：率先集成**GaLore**、**BAdam**、**APOLLO**、**Adam-mini**、**Muon**、**OFT**、**DoRA**、**PiSSA**、**LoRA+** 等众多最新优化算法。
* **全栈效率优化**：

+ **训练加速**：集成Unsloth、FlashAttention-2、Liger Kernel，显著提升速度、降低显存。
+ **推理加速**：支持vLLM和SGLang后端，推理速度提升高达270%。
+ **资源节省**：通过QLoRA（2/3/4/8-bit）、FSDP等技术，实现在单张消费级GPU上微调700亿参数模型。

* **统一且灵活的配置**：通过YAML文件统一管理所有参数，支持从本地、Hugging Face Hub、ModelScope、Modelers Hub甚至云存储加载数据和模型。
* **强大的生态与社区**：被**Amazon、NVIDIA、Aliyun**等巨头官方采用，拥有活跃的中英文社区、详细文档、博客和在线课程。
* **多模态与多任务**：无缝支持视觉、音频等多模态模型的微调，以及工具调用、长文本处理等复杂任务。

---

**总结**：LLaMA Factory 不仅仅是一个微调工具，它更是一个**不断进化、集成了最新研究成果的“大模型微调工厂”**。它极大地 democratize 了大语言模型的定制化能力，让任何开发者都能以极低的成本，将最先进的大模型适配到自己的特定需求和场景中，是当前开源社区中最活跃、最全面的LLM微调解决方案之一。
