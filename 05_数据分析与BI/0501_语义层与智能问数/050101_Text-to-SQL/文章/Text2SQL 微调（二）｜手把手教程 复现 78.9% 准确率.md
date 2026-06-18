---
title: Text2SQL 微调（二）｜手把手教程 复现 78.9% 准确率
author: EosphorosAI
date: 
url: http://mp.weixin.qq.com/s?__biz=MzkwMjU1MDM0Nw==&mid=2247484000&idx=1&sn=43d4fe3c6c7e6becf696c9f9def91683&chksm=c0a28444f7d50d529ee0fba7f78d6e85bdb94c90690ab5b9699170778f43260a4df635c7daba&mpshare=1&scene=24&srcid=1218dxjEtyQJDyzj0tBHoLQh&sharer_shareinfo=e0be092a76ee492ebfd68a4a53a87a94&sharer_shareinfo_first=e0be092a76ee492ebfd68a4a53a87a94#rd
---

本文基于社区项目中专注于 Text-to-SQL 微调的子项目 DB-GPT-Hub，和大家分享如何在 13B 的开源大模型 CodeLlama 上进行微调，实现在 Spider 评估集上的执行准确率达到 **0.789** (超过第三方评估 GPT-4 的 0.762 )。本文由社区核心成员石健、张洪洋共同完成，是关于 DB-GPT-Hub 系列文章的第二篇文章。

**DB-GPT-Hub 项目简介**：一个利用 LLMs 实现 Text-to-SQL 解析的项目，主要包含数据处理、模型选择与构建和微调权重等步骤，通过这一系列的处理可以在提高 Text-to-SQL 能力的同时降低模型训练成本，让更多的开发者参与到 Text-to-SQL 的准确度提升工作当中，最终实现基于数据库的自动问答能力，让用户可以通过自然语言描述完成复杂数据库的查询操作等工作。

**环境搭建**

项目运行环境所需依赖的库均在 requirements.txt 进行了说明

```
git clone https://github.com/eosphoros-ai/DB-GPT-Hub.gitcd DB-GPT-Hubconda create -n dbgpt_hub python=3.10 conda activate dbgpt_hubpip install -r requirements.txt
```

本教程所用模型为 CodeLlama-13b-Instruct-hf，模型可以从 HuggingFace、魔塔等处下载。以 HuggingFace 为示例，下载命令为：

```
cd Your_model_dirgit lfs installgit clone git@hf.co:codellama/CodeLlama-13b-Instruct-hf
```

**数据处理**

**0****1**

**数据集收集**

本项目案例数据主要以 Spider 数据集为示例 ：

* **简介**：Spider 数据集是一个跨域的复杂 text2sql 数据集，包含了自然语言问句和分布在 200 个独立数据库中的多条 SQL，内容覆盖了 138 个不同的领域。
* **下载**：从阅读原文中的链接下载数据集到项目目录,即位于 `dbgpt_hub/data/spider`中。

**0****2**

**数据处理代码执行**

DB-GPT-Hub 使用的是信息匹配生成法进行数据准备，即结合表信息的 SQL + Repository 生成方式，这种方式结合了数据表信息，能够更好地理解数据表的结构和关系，适用于生成符合需求的 SQL 语句。

项目已经将相关处理代码封装在对应脚本中，可以直接一键运行脚本命令，在 `dbgpt_hub/data/`目录中将得到生成的训练集 `example_text2sql_train.json`和`example_text2sql_dev.json`

```
## 生成 train 数据 和 dev(eval) 数据,sh dbgpt_hub/scripts/gen_train_eval_data.sh
```

其中训练集中为 8659 条，评估集为 1034 条。

生成的训练集中数据格式形如：

```
    {        "db_id": "department_management",        "instruction": "I want you to act as a SQL terminal in front of an example database, you need only to return the sql command to me.Below is an instruction that describes a task, Write a response that appropriately completes the request.\n\"\n##Instruction:\ndepartment_management contains tables such as department, head, management. Table department has columns such as Department_ID, Name, Creation, Ranking, Budget_in_Billions, Num_Employees. Department_ID is the primary key.\nTable head has columns such as head_ID, name, born_state, age. head_ID is the primary key.\nTable management has columns such as department_ID, head_ID, temporary_acting. department_ID is the primary key.\nThe head_ID of management is the foreign key of head_ID of head.\nThe department_ID of management is the foreign key of Department_ID of department.\n\n",        "input": "###Input:\nHow many heads of the departments are older than 56 ?\n\n###Response:",        "output": "SELECT count(*) FROM head WHERE age  >  56",        "history": []    }
```

在 `dbgpt_hub/data/dataset_info.json` 中配置训练的数据文件，json 中对应的 key 的值默认为 example\_text2sql，此值即在后续训练脚本 train\_sft 中参数 `--dataset` 需要传入的值， json中的`file_name` 的值为训练集的文件名字。

**0****3**

**数据处理代码逻辑**

数据处理的核心代码主要在 `dbgpt_hub/data_process/sql_data_process.py` 中，核心处理 class 是 `ProcessSqlData()`，核心处理函数是 `decode_json_file()`。

decode\_json\_file() 首先将 Spider 数据中的 table 信息处理成为字典格式，key 和 value 分别是 db\_id 和该 db\_id 对应的 table、columns 信息处理成所需的格式，例如：

```
{  "department_management": department_management contains tables such as department, head, management. Table department has columns such as Department_ID, Name, Creation, Ranking, Budget_in_Billions, Num_Employees. Department_ID is the primary key.\nTable head has columns such as head_ID, name, born_state, age. head_ID is the primary key.\nTable management has columns such as department_ID, head_ID, temporary_acting. department_ID is the primary key.\nThe head_ID of management is the foreign key of head_ID of head.\nThe department_ID of management is the foreign key of Department_ID of department.}
```

然后将上述文本填充于 config 文件中 INSTRUCTION\_PROMPT 的 {} 部分，形成最终的 instruction， INSTRUCTION\_PROMPT 如下所示：

```
INSTRUCTION_PROMPT = "I want you to act as a SQL terminal in front of an example database, you need only to return the sql command to me.Below is an instruction that describes a task, Write a response that appropriately completes the request.\n ##Instruction:\n{}\n"
```

最后将训练集和验证集中每一个 db\_id 对应的 question 和 query 处理成模型 SFT 训练所需的格式，即上面数据处理代码执行部分所示的数据格式。

如果你想自己收集更多数据进行训练，可以利用本项目相关代码参照如上逻辑进行处理。

**模型 SFT 训练**

## 

为了简便起见，本复现教程以 LoRA 微调直接运行作为示例，但项目微调不仅能支持 LoRA 还支持 QLoRA 以及 deepseed 加速。

复现效果的训练脚本 `dbgpt_hub/scripts/train_sft.sh` 详细参数如下所示：

```
CUDA_VISIBLE_DEVICES=0 python dbgpt_hub/train/sft_train.py \    --model_name_or_path Your_download_CodeLlama-13b-Instruct-hf_path \    --do_train \    --dataset example_text2sql_train \    --max_source_length 2048 \    --max_target_length 512 \    --finetuning_type lora \    --lora_target q_proj,v_proj \    --template llama2 \    --lora_rank 64 \    --lora_alpha 32 \    --output_dir dbgpt_hub/output/adapter/code_llama-13b-2048_epoch8_lora \    --overwrite_cache \    --overwrite_output_dir \    --per_device_train_batch_size 1 \    --gradient_accumulation_steps 16 \    --lr_scheduler_type cosine_with_restarts \    --logging_steps 50 \    --save_steps 2000 \    --learning_rate 2e-4 \    --num_train_epochs 8 \    --plot_loss \    --bf16
```

`train_sft.sh` 中关键参数与含义介绍：

* model\_name\_or\_path ：所用 LLM 模型的路径。
* dataset ：取值为训练数据集的配置名字，对应在 dbgpt\_hub/data/dataset\_info.json 中外层 key 值，如 example\_text2sql。
* max\_source\_length ：输入模型的文本长度，本教程的效果参数为 2048，为多次实验与分析后的最佳长度。
* max\_target\_length ：输出模型的 sql 内容长度，设置为 512。
* template：项目设置的不同模型微调的 lora 部分，对于 Llama2 系列的模型均设置为 llama2。
* lora\_target ：LoRA 微调时的网络参数更改部分。
* finetuning\_type : 微调类型，取值为 [ ptuning、lora、freeze、full ] 等。
* lora\_rank : LoRA 微调中的秩大小。
* loran\_alpha: LoRA 微调中的缩放系数。
* output\_dir ：SFT 微调时 Peft 模块输出的路径，默认设置在 dbgpt\_hub/output/adapter/路径下 。
* per\_device\_train\_batch\_size ：每张 gpu 上训练样本的批次，如果计算资源支持，可以设置为更大，默认为 1。
* gradient\_accumulation\_steps ：梯度更新的累计steps值。
* lr\_scheduler\_type ：学习率类型。
* logging\_steps ：日志保存的 steps 间隔。
* save\_steps ：模型保存的 ckpt 的 steps 大小值。
* num\_train\_epochs ：训练数据的 epoch 数。
* learning\_rate : 学习率，推荐的学习率为 2e-4。

如果想基于 QLoRA 训练，可以在脚本中增加参数 `quantization_bit` 表示是否量化，取值为 [ 4 或者 8 ]，开启量化。

对于想微调不同的 LLM，不同模型对应的关键参数 lora\_target 和 template，可以参照项目的 README.md 中相关内容进行更改。

**模型预测与评估**

**0****1**

**模型预测**

模型训练结束后，对训练好的模型进行预测，可以直接运行项目脚本目录中的 `predict_sft.sh`。

预测运行命令：

```
sh ./dbgpt_hub/scripts/predict_sft.sh
```

项目目录下`./dbgpt_hub/` 下的 `output/pred/`，此文件路径为关于模型预测结果默认输出的位置(如果没有则需建立)。

本教程 `predict_sft.sh` 中的详细参数如下：

```
echo " predict Start time: $(date)"## predictCUDA_VISIBLE_DEVICES=0 python dbgpt_hub/predict/predict.py \    --model_name_or_path Your_download_CodeLlama-13b-Instruct-hf_path \    --template llama2 \    --finetuning_type lora \    --checkpoint_dir Your_last_peft_checkpoint-4000  \    --predicted_out_filename Your_model_pred.sql  
echo "predict End time: $(date)"
```

其中参数 `--predicted_out_filename` 的值为模型预测的结果文件名，结果在 `dbgpt_hub/output/pred` 目录下可以找到。

**0****2**

**模型评估**

对于模型在数据集上的效果评估,默认为在 `spider` 数据集上。  
运行以下命令：

```
python dbgpt_hub/eval/evaluation.py --plug_value --input  Your_model_pred.sql
```

由于大模型生成的结果具有一定的随机性，和 temperature 等参数密切相关(可以在 `/dbgpt_hub/configs/model_args.py` 中的 GeneratingArguments 中进行调整）。在项目默认情况下，我们多次评估的效果，执行准确率均在 0.789 及以上。部分实验和评测结果我们已经放在项目 `docs/eval_llm_result.md` 中，仅供参考。

DB-GPT-Hub 基于 CodeLlama-13b-Instruct-hf 大模型用 LoRA 在 Spider 的训练集上微调后的权重文件已经放出，目前在 spider 的评估集上实现了约为 0.789 的执行准确率，权重文件 `CodeLlama-13b-sql-lora`  在 HuggingFace 上可以找到。

**附录**

本文实验环境为基于一台带有 A100(40G) 的显卡服务器，总训练时长 12h 左右。如果你的机器资源不够，可以优先考虑缩小参数 `gradient_accumulation_steps` 的取值，另外可以考虑用 QLoRA 的方式微调(训练脚本 `dbgpt_hub/scripts/train_sft.sh`中增加 `--quantization_bit 4`)，从我们的经验看，QLoRA 在 8 个 epoch 时和 LoRA 微调的结果相差不大。

**0****1**

DB-GPT 框架

https://github.com/eosphoros-ai

**0****2**

Text2SQL 微调

https://github.com/eosphoros-ai/DB-GPT-Hub

**0****3**

DB-GPT-WEB

https://github.com/eosphoros-ai/DB-GPT-Web

**0****4**

Awesome-Text2SQL

https://github.com/eosphoros-ai/Awesome-Text2SQL

**0****5**

第三方评估GPT-4

https://www.numbersstation.ai/post/nsql-llama-2-7b

**0****6**

训练的权重文件

https://huggingface.co/eosphoros