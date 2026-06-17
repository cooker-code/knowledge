---
title: OmniSQL: 大规模合成高质量的Text-to-SQL数据
author: Paper易论
date: 
url: http://mp.weixin.qq.com/s?__biz=MzIzOTAxMjk4Ng==&mid=2454620752&idx=1&sn=776bd75555ae4ca4bb35d35ddac47f2b&chksm=ff6cc3e4bc6db2877d1adab9ed172b2ff93f212b3ca83adcda6404ee384599ea29ef2b10bb4b&mpshare=1&scene=24&srcid=0307APaDrLEr2AC1UovsQbOa&sharer_shareinfo=b6c42f950cc332c4134851a2f0a1b91d&sharer_shareinfo_first=b6c42f950cc332c4134851a2f0a1b91d#rd
---

我们提出了一个自动且可扩展的 Text-to-SQL 数据合成框架，如下图所示：

    基于此框架，我们推出了首个百万级别的 Text-to-SQL 数据集SynSQL-2.5M，包含超过250万个多样且高质量的数据样本，涵盖了来自多个领域的16,000多个数据库。

    在SynSQL-2.5M的基础上，我们推出了OmniSQL，这是一系列强大的 Text-to-SQL 模型，提供三种规模：7B、14B和32B。在微调过程中，我们还整合了来自Spider和BIRD的训练集，这些数据集提供了高质量的人工标注数据。

（论文和代码将在几天内发布，敬请期待！）

下 载

性 能

我们在多个数据集上评估了OmniSQL，包括标准基准（Spider和BIRD）、具有挑战性的领域特定基准（Spider2.0-SQLite、ScienceBenchmark、EHRSQL）以及三个鲁棒性基准（Spider-DK、Spider-Syn、Spider-Realistic）。评估结果如下：

"Gre"指的是贪婪解码，"Maj"表示8次主要投票

    Spider (dev)、Spider-Syn和Spider-Realistic使用测试套件准确率（TS）指标进行评估，其余数据集使用执行准确率（EX）指标进行评估。

    在每个模型规模下，我们将 OmniSQL 与相似或更大规模的开源LLMs进行比较。我们的发现如下：

    合成数据显著增强了基础模型的Text-to-SQL能力。 比较Qwen2.5-Coder模型和 OmniSQL 显示，使用 SynSQL-2.5M 微调后，在大多数数据集上的性能有所提升。具体来说，在贪婪解码策略下， OmniSQL -7B在其基础模型Qwen2.5-Coder-7B-Instruct的基础上平均提升了+8.4%（从52.8%提升至61.2%）。类似地， OmniSQL -14B和 OmniSQL -32B在其基础模型Qwen2.5-Coder-14B-Instruct和Qwen2.5-Coder-32B-Instruct上分别平均提升了2.9%（从59.3%提升至62.2%）和2.4%（从60.8%提升至63.2%）。值得注意的是， OmniSQL -7B在该领域内为7B规模的LLMs树立了新的标准。此外， OmniSQL -14B和 OmniSQL -32B代表了最新的Text-to-SQL能力。

    OmniSQL 在标准基准测试中始终表现出领先性能，包括Spider（开发集）、Spider（测试集）和BIRD（开发集）。 特别是， OmniSQL -7B模型在Spider测试集上的准确率为87.9%（贪婪解码），超过了Spider排行榜上最好的公开方法DAIL-SQL + GPT-4 + 自我一致性（86.6%） (D. Gao et al. 2024) ，提高了1.3%。此外，通过简单的多数投票策略， OmniSQL 在Spider测试集上的性能进一步提高到88.9%，88.3%，和89.8%，分别为7B、14B和32B模型规模。另外，我们观察到随着模型规模的增加，Spider上的性能提升并不一致。这可能是由于Spider是一个相对简单的基准测试，在此情况下较小规模但强大的模型已经接近最优性能。相比之下，BIRD比Spider更具挑战性，因此我们可以看到随着 OmniSQL 规模的增加，性能有轻微提升。值得注意的是， OmniSQL -32B在BIRD开发集上的多数投票准确率达到67.0%，使其与Distillery + GPT-4o（67.2%）相当，后者是在BIRD训练集上微调GPT-4o的结果。

    OmniSQL 在特定领域基准测试（包括Spider2.0-SQLite、EHRSQL和ScienceBenchmark）中展示了出色的领域泛化能力。 在最具挑战性的基准测试Spider2.0-SQLite上，尽管模型参数有限， OmniSQL 仍然表现出与较大模型相当的准确性。特别是对于7B规模的基线LLMs，其准确率仅在0.7%到3.7%之间，而 OmniSQL -7B达到了10.4%（贪婪解码）的准确率，突显了小型模型处理复杂Text-to-SQL问题的潜力。在EHRSQL和ScienceBenchmark上， OmniSQL 始终优于相同规模的竞争者，并与领先的LLMs表现相当。特别是在涉及医疗领域数据库的EHRSQL上， OmniSQL -32B取得了最佳性能，多数投票准确率为46.8%，超过了GPT-4o（多数投票准确率为45.5%）1.3%。这些结果表明， OmniSQL 能够在不进行大量领域特定微调的情况下，有效地泛化到专业领域的数据库。

    有趣的是，一些开源LLMs（例如Qwen2.5-Coder-32B-Instruct和Meta-Llama-3.1-70B-Instruct） 在标准数据集上的表现显著优于封闭源代码LLMs。然而，它们在特定领域数据集上的优势减弱。这可能是因为这些开源模型在预训练或微调过程中纳入了流行Text-to-SQL基准测试（例如Spider和BIRD）的训练集。虽然这使得它们在熟悉的数据集上表现出色，但在遇到跨域数据集时效果下降。相比之下， OmniSQL 表现出持续的强性能，无论是在标准还是特定领域基准测试中都表现出色。

    OmniSQL显著优于类似规模的基线LLM，甚至在许多数据集上超越了领先的模型如GPT-4o和DeepSeek-V3。

    这些分数是由单个LLM实现的，没有额外的设计，如模式链接、SQL修订和SQL选择。我们相信通过整合这些技术，准确率可以进一步提高。

vLLM 和 Transformers 快速入门  
以下是一些示例代码片段，可以快速使用OmniSQL执行 Text-to-SQL。

提示模板  
OmniSQL使用的提示模板定义如下：

```
input_prompt_template = '''Task Overview:You are a data science expert. Below, you are provided with a database schema and a natural language question. Your task is to understand the schema and generate a valid SQL query to answer the question.  
Database Engine:SQLite  
Database Schema:{db_details}This schema describes the database's structure, including tables, columns, primary keys, foreign keys, and any relevant relationships or constraints.  
Question:{question}  
Instructions:- Make sure you only output the information that is asked in the question. If the question asks for a specific column, make sure to only include that column in the SELECT clause, nothing more.- The generated query should return all of the information asked in the question without any missing or extra information.- Before generating the final SQL query, please think through the steps of how to write the query.  
Output Format:In your answer, please enclose the generated SQL query in a code block:```-- Your SQL query```  
Take a deep breath and think step by step to find the correct SQL query.'''
```

    替换"db\_details"和"question"的占位符即可开始。注意，"db\_details"格式化为数据库中表的CREATE TABLE语句（即DDL）。您可以在DDL中添加数据库值和列描述，使用SQL注释。外部知识可以与自然语言问题连接并放置在"question"占位符中。OmniSQL目前仅支持SQLite，因为SynSQL-2.5M中的SQL查询是使用SQLite方言合成的。

我们在examples文件夹中提供了示例提示。

使用 vLLM 进行推理

以下代码片段展示了如何使用vLLM与OmniSQL。

```
from vllm import LLM, SamplingParamsfrom transformers import AutoTokenizer  
prompt = input_prompt_template.format(db_details = "...", question = "...")model_path = "seeklhy/OmniSQL-7B"tokenizer = AutoTokenizer.from_pretrained(model_path)sampling_params = SamplingParams(    temperature = 0,     max_tokens = 2048,    n = 1)  
llm = LLM(    model = model_path,    dtype = "float16",     tensor_parallel_size = 1,    max_model_len = 8192,    gpu_memory_utilization = 0.92,    swap_space = 8,    enforce_eager = True,    disable_custom_all_reduce = True,    trust_remote_code = True)  
chat_prompt = tokenizer.apply_chat_template(    [{"role": "user", "content": prompt}],    add_generation_prompt = True, tokenize = False)  
outputs = llm.generate([chat_prompt], sampling_params)  
for output in outputs:    responses = [o.text for o in output.outputs]    print(responses[0])
```

确保您的环境中已正确安装vLLM。

使用Transformers进行推理

您也可以选择使用Transformers进行推理。

```
import torchfrom transformers import AutoTokenizer, AutoModelForCausalLM  
prompt = input_prompt_template.format(db_details = "...", question = "...")model_path = "seeklhy/OmniSQL-7B"tokenizer = AutoTokenizer.from_pretrained(model_path)model = AutoModelForCausalLM.from_pretrained(    model_path,    torch_dtype=torch.bfloat16).to("cuda:0")  
chat_prompt = tokenizer.apply_chat_template(    [{"role": "user", "content": prompt}],    add_generation_prompt = True, tokenize = False)  
inputs = tokenizer([chat_prompt], return_tensors="pt")inputs = inputs.to(model.device)  
output_ids = model.generate(    **inputs,    eos_token_id = tokenizer.eos_token_id,    max_new_tokens = 2048)  
input_len = len(inputs.input_ids[0])output_ids = output_ids[0][input_len:]  
response = tokenizer.batch_decode([output_ids], skip_special_tokens = True)[0]print(response)
```

局限性  
    SynSQL-2.5M是一个专注于SQLite数据库引擎的英文数据集，因此在多语言和多SQL方言场景中的性能可能有限。但是，您可以使用我们提出的框架合成新的数据样本以适应您的场景。合成新数据集后，您可以使用OmniSQL进行进一步的微调，因为它是 Text-to-SQL 能力的一个强大起点。

原论文：https://arxiv.org/pdf/2503.0224

GitHub地址：

https://github.com/RUCKBReasoning/OmniSQL