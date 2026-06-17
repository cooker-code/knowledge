---
title: 微软开源 SkillOpt：把 Skill 当神经网络一样训练
author: 代码麻辣烫
date: 一叶扁舟一叶扁舟
url: https://mp.weixin.qq.com/s?__biz=Mzk0NjY0MTc0NA==&mid=2247500346&idx=1&sn=a8d6d0b7df4d0a6fd5f959dd75ad0027&chksm=c20d005928afab4a6a3cc97d168cd3b52f237d192a438d79663f6eae2073edc1de0301df6327&mpshare=1&scene=24&srcid=0530DdfRuga9nDuN5EpoU0qR&sharer_shareinfo=0a741830b20c8c10d9412f3d7f20d205&sharer_shareinfo_first=0a741830b20c8c10d9412f3d7f20d205#rd
---

> 把 Skill 当成 Agent 的"外部参数"，像调神经网络一样调它

5 月 28 日，微软在 GitHub 上悄悄推了一个新仓库——**SkillOpt**。一周时间， star 冲到 2.5k ， arXiv 论文同步上线，项目页（ microsoft.github.io/SkillOpt ）做得很认真，配套还放了一个 demo 视频。

这件事值得拎出来讲一句的原因，不在于 star 数，而在于它处理 prompt 工程的思路跟过去几年那一波"教你怎么写好 Prompt"的内容完全不在一个层次上。

它不再问你"该怎么写 Skill"，它问的是：**能不能把 Skill 这个文档本身，当成一个可以被训练的东西**。

## 痛点其实大家都熟

做过 Agent 的人对这场景应该不陌生。

模型今天跑偏了，你往 Skill 文件里加一句"别这么干"。明天发现输出格式又乱了，再补一句"输出长这样"。 Skill 越写越长，规则越堆越多， Agent 的表现却没有一起涨——往往是某个新加的句子和老规则打架，两边一抵消，整体反而退步了。

更难受的是，你说不清楚问题出在哪。哪条规则真的有效？哪条其实白写了？哪条改坏了原来好用的行为？没办法量化、没办法回退，全凭手感。

按 SkillOpt 论文里的话讲：今天的 Agent skill 要么是手写的，要么是 LLM 一次性生成的，要么靠模型自己反思着改，没有一种像深度学习那样有优化器在盯——也就没有一种能可靠地比初始版本更好。

> Agent skills today are hand-crafted, generated one-shot, or evolved through loosely controlled self-revision, none of which behaves like a deep-learning optimizer for the skill, and none of which reliably improves over its starting point under feedback.

## 一个反常识的视角： Skill 不是 Prompt ，是参数

SkillOpt 的核心主张可以一句话讲完：**Skill 是冻结 Agent 的"外部状态"，应该像神经网络的权重一样被训练**。

把这句话拆开看。

模型本身不动（ frozen target model ），就用你现在用的 GPT-5 、 Claude 或者 Qwen ，权重一个字节都不改。但 Skill 这份 Markdown 文档——也就是你给 Agent 看的那份操作手册——会被一个独立的"优化器模型"反复修订，目标是让 Agent 在测试集上的得分越来越高。

整个过程刻意做得跟训练神经网络很像：有 epoch 、有 minibatch 、有学习率（控制每次能改几处）、有验证集（决定要不要接受这次改动）。

> Train agent skills like you train neural networks — with epochs, (mini-)batchsize, learning rates, and validation gates — but without touching model weights.

这种"把 Prompt 变成可训练对象"的视角不是 SkillOpt 第一个提的——TextGrad 、 GEPA 都走过类似的路。 SkillOpt 的不同在于，它把整套训练循环做得**特别像工程**：每一步都有边界、有回退、有记录、有持久化，最后能导出一份单独的 `best_skill.md` 文件直接部署。

## 四步循环： rollout → reflect → edit → gate

把这张图盯仔细。整个 SkillOpt 的训练循环就是这四个动词。

**Rollout （铺作业）**：用当前版本的 Skill ，让目标模型去跑一个 batch 的任务，全程记录消息、工具调用、验证器反馈、最终得分。这一步像神经网络的前向传播——拿数据过一遍，看看现状。

**Reflect （复盘）**：优化器模型接手，把刚才的 rollout 拆成成功 minibatch 和失败 minibatch ，分别复盘。成功的那批用来提炼"哪些做法被反复验证了"，失败的那批用来找规律——是不是老在同一个坑里栽。这一步对应反向传播，只不过梯度变成了自然语言上的归纳。

**Edit （动手改）**：优化器把复盘结论变成具体的 add / delete / replace 操作，提交到当前 Skill 上。这里有一个**关键限制**：每一步的改动数量被一个"文本学习率（ textual learning rate ）"卡住，论文里默认的 `lr=4`，意思是一次最多改 4 处。

为什么要卡这个？放开了改，模型就开始大刀阔斧重写整篇 Skill ，原本好用的规矩很容易被一起冲掉。卡死了改，模型动一动就要算账，反而会找最关键的那几条下手。这是把"小步迭代"硬编码进了优化过程。

**Gate （验证门）**：改完之后，拿一份**没见过**的验证集去重新评分。比之前好，留下；没好，立刻退回到上一版。

这四步循环跑完一个 epoch ，再来一轮。

## 三个看起来没什么用的细节，其实在"承重"

光有上面四步还不够。论文里花了大量篇幅讲控制项的消融实验，结论是：**学习率、拒绝缓冲、慢更新这三个看起来不起眼的设计，是让 Skill 训练能稳住的关键**。

| 控制项 | SearchQA | SpreadsheetBench | LiveMath |
| --- | --- | --- | --- |
| 默认（ lr=4 ） | 87.1 | 77.5 | 61.3 |
| 去掉学习率 | 84.6 | 75.7 | 57.3 |
| 默认 | 87.1 | 77.5 | 61.3 |
| 去掉拒绝缓冲 | 85.5 | 72.9 | 58.9 |
| meta skill + slow update | 87.1 | 77.5 | 61.3 |
| 两个都去掉 | 86.3 | **55.0** | 59.7 |

最右下那个 55.0 分，是把 meta skill 和 slow update 一起拿掉之后， SpreadsheetBench 直接从 77.5 跌到 55——掉了 22 分。

这三件事各自是什么？

**学习率（ textual learning rate ）**：前面提过的那个"每步最多改几处"。把它拿掉， Skill 训练就会被大改写吃掉，新规矩没记住，老规矩反而被覆盖。

**拒绝缓冲（ rejected-edit buffer ）**：被验证集打回去的改动不会消失，会被存进一个"失败档案"。下次优化器再想改，会先翻一翻这个档案——这条路已经试过、走不通的，就不会反复地往原地撞。本质上是把负反馈也用起来。

**慢更新 + meta skill （ slow update + meta skill ）**：每跑完一个 epoch ，做一次"大复盘"，把整份 Skill 重新捋一遍。同时优化器自己也维护一份"我作为优化器学到了什么"的元 Skill 。这给 Skill 训练加了一个长程的反思层，避免优化器自己也陷在某条短期套路里。

把这三个控制项都关掉，剩下的就是普通的"模型自己反思着改"——也就是过去一年大家普遍在做的那种 self-revision ，效果会肉眼可见地差。

## 它真的有用吗： 52/52

光说机制没用。论文给了一张大表，覆盖了 7 个目标模型、 6 个 benchmark 、 3 种执行环境（ Direct chat / Codex / Claude Code ），一共 52 个评测格子。

> Best or tied on all 52 evaluated (model, benchmark, harness) cells.

每一格都是第一或者并列第一。

挑几个有意思的数字：

•GPT-5.5 在直接对话下平均 +23.5 分，扔进 Codex 里 +24.8 分，扔进 Claude Code 里 +19.1 分；

•表格类任务（ SpreadsheetBench ）的提升最夸张， GPT-5.5 在 Codex 下 +57.5 、在 Claude Code 下 +58.3——基本等于把它从"勉强能用"拉到了"可以交付"；

•Qwen3.5-4B 这种相对小的模型，在 ALFWorld 上 +50.7 。这个挺关键，说明 Skill 优化对小模型更有杠杆——它们更需要外部的 procedure 来弥补能力短板。

参与对比的 baseline 包括 Human 写的 Skill 、 LLM 一次性生成的 Skill 、 Trace2Skill 、 TextGrad 、 GEPA 、 EvoSkill——SkillOpt 在每一类任务上都越过了最强的那个 baseline ，最少 +1.7 （ DocVQA ），最多 +9.2 （ LiveMath ）。

更让人意外的是，**这一切都没动模型权重一个比特**。

## 那张训练曲线图看到了什么

如果你训练过模型，看这张图会有种熟悉感。

横轴是 epoch checkpoint ，纵轴是分数，三条线分别是训练 rollout 、验证选择、未见测试。 SearchQA 那条线慢慢爬升、波动收敛； SpreadsheetBench 跳得更剧烈，但每次跳完都被验证集"按住"——这就是 gate 在干活。

LiveMath 那条最有意思：训练 rollout 的得分有时候会反而下来，但验证选择的曲线没跟着掉。原因是优化器 propose 了一个改动， rollout 看着数字变好了，结果验证集一上发现没用，就被打回去了——你能从图里看到这个"提议-否决"的过程被沉淀成了一个个可复现的 checkpoint 。

这个体验跟读 TensorBoard 上的 train/val 曲线非常像。差别只是这里训的不是权重，是一份 Markdown 文件。

## ALFWorld 那个例子：留下的就那么三句话

论文里有一段挺直观的演示。让 GPT-5.4-mini 当目标模型、 GPT-5.5 当优化器，去跑 ALFWorld 这个具身 Agent benchmark 。从一个非常基础的 Skill 起步，跑到 epoch 3 那次 slow update 之后， hard 模式得分从 70.9% 干到 85.8%——涨了将近 15 个点。

最后被留下、起决定作用的核心规矩，论文里直接列了出来，就这么三条：

> Count any generic target receptacle instance as valid.  
> Keep a strict numbered searched set and do not re-check observed locations.  
> Broaden search after several misses in one location type.

翻译过来：把任何一个泛指的目标容器都当成有效目标；维护一个编号的"已搜过"集合，不要重复检查同一个位置；同一类位置连续找不到几次之后，扩大搜索范围。

这三句话单看普通到不行——任何一个写过搜索逻辑的工程师都能想到。关键不在于"这三句话很厉害"，而在于：**它是从一堆失败 rollout 的归纳里被挑出来、又通过验证集筛选过的，所以它是真的有用，不是听起来有道理而已**。

这个差别就是 SkillOpt 想干的事：把 Skill 从"凭直觉写"，变成"被数据筛选出来"。

## 一个意外的彩蛋： Skill 真的能迁移

论文里还做了一组迁移实验，结果挺反直觉。

•**跨模型**：拿 GPT-5.4 训出来的 LiveMath Skill ，直接给 GPT-5.4-nano 用，分数 +15.2 ；

•**跨执行环境**：拿在 Codex 里训出来的 SpreadsheetBench Skill ，直接放进 Claude Code ，分数 +31.8 ；

•**自己当自己的优化器**：让 GPT-5.4-nano 当目标模型，也让它自己当优化器（不借助更强的模型）， SpreadsheetBench 还是涨了 +10.4 。

最后那条特别值得品。它说明 SkillOpt 不只是"用更强的模型蒸馏出 Skill 给更弱的模型"——即使优化器和目标是同一个模型，只要那套约束还在（学习率、缓冲、验证门），它依然能挖出有效的改动。

更实际的层面是：**优化阶段烧多少 token 是优化阶段的事，部署阶段只用一份 best\_skill.md**。优化器的记忆、消融过程、被拒绝的改动统统留在训练侧，目标模型在生产环境只读那一份文档。这种"训练贵、推理便宜"的范式跟传统 ML 是一样的。

## 怎么上手：三步装好

GitHub 仓库（ github.com/microsoft/SkillOpt ）目前没有 pip 包，得克隆下来本地装：

```
git clone https://github.com/microsoft/SkillOpt.git
cd SkillOpt
pip install -e .
```

需要 Python 3.10+。环境变量按需配， OpenAI / Azure OpenAI / Anthropic / 本地 vLLM 跑的 Qwen 都支持。

数据准备的格式很标准——一个目录下面切 `train/`、`val/`、`test/` 三个子目录，每个里面放一份 JSON 。每条任务自带答案，框架就是用这套答案在 gate 阶段打分。仓库里没自带数据集，得自己按 schema 准备。

启动一次训练就是一行 `python scripts/train.py`，挂上 config 和数据路径就行。重跑同样的命令会自动从最新 checkpoint 续上，不需要再传 resume 标志。

如果想看训练过程的可视化，还有个 WebUI （基于 Gradio ）：

```
pip install -e ".[webui]"
python -m skillopt_webui.app
```

默认开在 7860 端口，加 `--share` 还能开公网。

## 写在后面

把这件事放进更大的脉络里：上一篇我们刚聊过 Anthropic 的 Dynamic Workflows——把"编排过程"代码化；这一篇 SkillOpt 干的是另一件事——把"行为规范"参数化。

两件事是同一个方向上的不同分支：**让 Agent 系统里那些原本靠人类经验积累的部分，变成可被训练、可被验证、可被回退的工程对象**。

这个趋势挺值得留心。过去两年，行业花了很多时间往"模型本身"上堆能力——更长的上下文、更强的推理、更大的工具集。 SkillOpt 这类工作在提一个不一样的问题：**模型已经够强了，剩下要解决的是怎么用好它**。而怎么用好，本身可以被工程化。

至于这个项目本身能跑多远，得看微软后续是不是会把它整合进 Azure 的 Agent 服务里，或者社区会不会在它之上长出更多 benchmark 之外的实战 Skill 库。但即便它最后只停留在论文阶段，它把 Skill 训练这件事讲清楚的方式，已经够其它项目抄一阵子了。

仓库地址： https://github.com/microsoft/SkillOpt  
论文地址： https://arxiv.org/abs/2605.23904  
项目页： https://microsoft.github.io/SkillOpt/