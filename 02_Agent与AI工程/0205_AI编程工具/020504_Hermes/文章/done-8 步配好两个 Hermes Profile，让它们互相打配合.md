> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020504_Hermes/020504_核心知识点/Hermes协作与记忆治理边界|Hermes协作与记忆治理边界]]
---
title: 8 步配好两个 Hermes Profile，让它们互相打配合
author: AI赋能说
date: AI赋能说AI赋能说
url: https://mp.weixin.qq.com/s?__biz=MzI3NjE4OTAyMg==&mid=2247488725&idx=1&sn=60db1ef29da4f479bec67d045a12175d&chksm=ea433abdf9d58a97cbe1456ebe61f8e38c444a5a6c1691c57e5cd8b3e0eb0397ff5bdeaac374&mpshare=1&scene=24&srcid=05102BkADYupqwNix9sf3Yo1&sharer_shareinfo=202b330094db2042370e44780a66001a&sharer_shareinfo_first=202b330094db2042370e44780a66001a#rd
---

昨晚试了一件事。

让两个 Hermes Agent 协作。一个负责查资料，一个负责写文章。各有各的记忆，各有各的工具。跑通之后我盯着屏幕看了好一会。

一个人的电脑上，跑着一支小团队。

这篇教程带你走一遍。8 步，分 3 个阶段。跟着做完，你也能配好「研究员」和「写手」两个 Profile，让它们协作完成一个从调研到成稿的任务。

## 完成后的样子

两个 Profile 各自独立。通过文件系统传递结果。研究员写完调研放到共享目录，写手读取后开始写。

## 前提条件

* Hermes 已安装，版本 v0.6.0 或以上
* 终端能跑 `hermes` 命令
* 有一个可用的 API Key（OpenRouter 或其他 provider）

---

## 阶段一：创建两个 Profile

### 第一步：创建研究员 Profile

一条命令：

```
hermes profile create researcher
```

Hermes 会在 `~/.hermes/profiles/researcher/` 下创建独立的配置目录。

验证：

```
ls ~/.hermes/profiles/researcher/
```

看到 `config.yaml` 就对了。这个 Profile 有自己的配置空间，不会和默认 Profile 冲突。

### 第二步：创建写手 Profile

同样的操作：

```
hermes profile create writer
```

验证：

```
ls ~/.hermes/profiles/writer/
```

两个 Profile 创建完毕。各自独立。各自有自己的记忆文件、配置文件、Skill 目录。

### 第三步：给研究员配模型和工具

编辑研究员的配置：

```
vim ~/.hermes/profiles/researcher/config.yaml
```

写入以下内容（根据你的实际情况改 API Key 和模型名）：

```
name: researcher
description: "负责信息搜集和调研，输出结构化的调研报告"

provider:
  type: openrouter
  api_key: 你的OpenRouter-Key
  model: anthropic/claude-sonnet-4-20250514

tools:
  - web_search
  - web_fetch
  - file_write

memory:
  enabled: true
```

研究员用推理能力强的模型。工具只给搜索、抓取、写文件。不给它写代码的能力。职责清晰。

保存退出。

---

## 阶段二：配置写手并建立协作通道

### 第四步：给写手配模型和工具

编辑写手的配置：

```
vim ~/.hermes/profiles/writer/config.yaml
```

```
name: writer
description: "负责将调研材料转化为可读的文章"

provider:
  type: openrouter
  api_key: 你的OpenRouter-Key
  model: anthropic/claude-sonnet-4-20250514

tools:
  - file_read
  - file_write

memory:
  enabled: true
```

写手只需要读文件和写文件。不需要搜索。它的输入来自研究员的输出。

想了想，这里有个设计选择。写手可以用更便宜的模型。比如 `google/gemini-2.0-flash`。写作任务对推理要求没那么高。省钱。

### 第五步：创建共享工作目录

两个 Profile 之间怎么传递信息？最简单的方式：文件系统。

```
mkdir -p ~/hermes-workspace/shared
```

这个目录是两个 Agent 的「交接区」。研究员把调研结果写到这里，写手从这里读取。

不需要复杂的消息队列。不需要 API 调用。一个文件夹就够了。

### 第六步：给研究员写一个 Skill

让研究员知道把结果放哪里。

```
mkdir -p ~/.hermes/profiles/researcher/skills/research-output
```

创建 Skill 文件：

```
vim ~/.hermes/profiles/researcher/skills/research-output/SKILL.md
```

```
---
name: research-output
description: |
  调研输出规范。当完成调研任务时使用。将调研结果以结构化格式写入共享目录。
---

# 调研输出规范

完成调研后，将结果写入 ~/hermes-workspace/shared/ 目录。

## 输出格式

文件名：research-[主题关键词].md

内容结构：
- 一句话摘要
- 关键发现（3-5条）
- 信息来源列表
- 原始素材摘录

写完后在文件末尾加一行：`<!-- STATUS: READY -->`

这个标记告诉写手：素材准备好了，可以开始写。
```

保存。这个 Skill 让研究员的输出有固定格式。写手拿到就能用。

---

## 阶段三：跑一个协作任务

### 第七步：启动研究员执行调研

用 `--profile` 参数启动指定 Profile：

```
hermes --profile researcher
```

进入对话后，给它一个调研任务：

```
请调研「2024年本地AI Agent框架的发展趋势」，重点关注开源项目、多Agent协作、记忆系统三个方向。调研结果写入 ~/hermes-workspace/shared/research-local-ai-agents.md
```

等它跑完。验证输出：

```
cat ~/hermes-workspace/shared/research-local-ai-agents.md
```

看到结构化的调研内容，末尾有 `<!-- STATUS: READY -->` 标记。研究员的活干完了。

### 第八步：启动写手完成成稿

新开一个终端窗口。启动写手：

```
hermes --profile writer
```

给它写作指令：

```
请读取 ~/hermes-workspace/shared/research-local-ai-agents.md 的调研材料，写一篇1500字左右的文章。主题是本地AI Agent的多Agent协作趋势。语气平实，面向技术爱好者。输出到 ~/hermes-workspace/shared/article-local-ai-agents.md
```

等它写完。验证：

```
cat ~/hermes-workspace/shared/article-local-ai-agents.md
```

一篇基于调研素材的文章。从搜集到成稿，两个 Agent 各干各的。

## 完整流程一览

## 第一次做的建议

* 先用同一个模型跑两个 Profile。确认流程通了再换不同模型。省得同时排查模型问题和配置问题。
* 共享目录里的文件名要有规律。我用 `research-` 前缀表示调研，`article-` 前缀表示成稿。文件多了不会乱。
* 最容易被跳过但最重要的是第六步。没有 Skill 约束输出格式，研究员写出来的东西写手不一定能直接用。格式对齐是协作的基础。

## 容易踩的坑

**两个 Profile 用了同一个记忆文件。**每个 Profile 的记忆是独立的。但如果你手动把路径配成一样，它们会互相覆盖。确认各自的 `~/.hermes/profiles/xxx/MEMORY.md` 是分开的。

**研究员输出的文件路径写错了，写手找不到。**路径必须用绝对路径。不要用 `./shared/`，要用 `~/hermes-workspace/shared/`。两个 Profile 的工作目录可能不同。

**写手启动太早，调研还没写完。**文件系统是异步的。没有锁机制。解决办法就是那个 `<!-- STATUS: READY -->` 标记。让写手先检查标记再开始写。

**config.yaml 格式错了，Profile 启动报错。**YAML 对缩进敏感。用空格不用 Tab。冒号后面必须有空格。报错时先检查缩进。

**给 Profile 配了太多工具，Agent 行为不可控。**工具越少越好。研究员不需要写代码。写手不需要搜索。职责边界靠工具集来约束。

---

## 参考资料

1. Hermes Agent Multi Agent Profiles — Julian Goldie[1]
2. Hermes Multi Agent Workflow[2]
3. Hermes v0.6.0 Multi Agent Profiles[3]
4. Hermes Agent GitHub[4]

Reference

[1] 

Hermes Agent Multi Agent Profiles — Julian Goldie: *https://juliangoldie.com/hermes-agent-multi-agent-profiles/*

[2] 

Hermes Multi Agent Workflow: *https://juliangoldie.com/hermes-multi-agent-workflow/*

[3] 

Hermes v0.6.0 Multi Agent Profiles: *https://juliangoldie.com/hermes-v0-6-0-multi-agent-profiles/*

[4] 

Hermes Agent GitHub: *https://github.com/NousResearch/hermes-agent*

**下方是赋能君的AI学习交流永久免费星球，想学习更多内容，欢迎扫码加入。**

🙌 如果你阅读到这里，说明我们对信息的认可区域是有一定交集的，可以说我们是同道中人，所以如果你有自认为不错的信息获取渠道，欢迎留言或者私聊我，谢谢。

都看到这里了，就给个关注吧👀：

喜欢我的文章，可以请你右下角顺手来一波点赞&在看&分享三连么👉