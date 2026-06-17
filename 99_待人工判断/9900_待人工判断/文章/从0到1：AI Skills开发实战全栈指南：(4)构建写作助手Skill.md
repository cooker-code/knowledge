---
title: 从0到1：AI Skills开发实战全栈指南：(4)构建写作助手Skill
author: 于天惠
date: 
url: https://mp.weixin.qq.com/s?__biz=MzAxNzU2NDQwNA==&mid=2247484598&idx=1&sn=518ab70632694f3a261185142df8fee7&chksm=9aeb9a7c6e187dde2097139c4e977f57381fb5c717cdbf54f0660edba5c18b12c1230dfd55f7&mpshare=1&scene=24&srcid=02140TPJxwTdaRfwRamy9OVE&sharer_shareinfo=ba25e80f54fa535f5581cce15a5f92d3&sharer_shareinfo_first=ba25e80f54fa535f5581cce15a5f92d3#rd
---

# 构建写作助手Skill：

## 为公众号作者打造专属AI编辑器

> **作者：于天惠**  
> 本文为《从0到1：AI Skills开发实战全栈指南》系列第4篇

---

在上一篇，我们学会了创建通用型Skill。但真正的价值，在于**解决特定角色的痛点**。

今天，我们就为**公众号作者**量身定制一个AI写作助手——它不仅能生成初稿，还能优化标题、检查敏感词、适配排版规范，甚至模拟读者反馈。

更重要的是，这个Skill将体现**工程化思维**：模块化设计、多阶段校验、风格一致性保障。你学到的不仅是“怎么写”，更是“如何构建专业级内容生产系统”。

---

## 一、需求拆解：公众号作者的真实痛点

作为技术类公众号主理人，我深知以下困境：

| 痛点 | 传统AI方案缺陷 | Skill应提供的能力 |
| --- | --- | --- |
| **选题难** | 随机生成，脱离受众兴趣 | 基于历史爆款+行业趋势生成选题 |
| **标题平** | “浅谈XXX”式标题，打开率低 | 自动生成3种高点击率标题（悬念/数字/冲突） |
| **结构散** | 段落逻辑跳跃，缺乏主线 | 强制使用SCQA或金字塔结构 |
| **风格飘** | 时而严肃时而玩梗，人设模糊 | 锁定“专业但亲切”的语气 |
| **合规险** | 无意触碰敏感词（如“最”、“绝对”） | 自动扫描并替换违规表述 |

我们的目标：**让AI从“文字搬运工”升级为“主编级协作者”**。

---

## 二、Skill设计：分层架构与模块化

我们将 `wechat-writing-assistant` 设计为三层结构：

```
wechat-writing-assistant/  
├── SKILL.md                 # 主指令  
├── references/  
│   ├── tone-guide.md        # 语气规范（“专业但亲切”）  
│   ├── banned-words.txt     # 敏感词库  
│   └── structure-templates/ # 结构模板（SCQA/STAR等）  
├── scripts/  
│   ├── check-banned-words.py # 敏感词检测  
│   └── generate-headlines.py # 标题生成器  
└── assets/  
    └── emoji-palette.json   # 推荐表情符号（避免滥用）
```

### ▶ 核心思想：**主Skill协调，子模块执行**

* • `SKILL.md` 定义整体流程：“先选题 → 再列大纲 → 写初稿 → 优化标题 → 合规检查”
* • 具体任务由脚本或参考文档完成，确保可维护性

---

## 三、核心文件实现：SKILL.md 编写详解

在 `~/.claude/skills/wechat-writing-assistant/SKILL.md` 中：

```
---  
name: wechat-writing-assistant  
description: 为技术类公众号生成高打开率、结构清晰、合规安全的文章，包含选题、标题、正文全流程。  
version: 1.0.0  
author: yutianhui  
tags: [content, writing, wechat]  
---  
  
# 公众号写作专家  
  
## 目标  
产出符合「于天惠」人设的技术深度文：专业严谨但语言亲切，带适度emoji，无营销感。  
  
## 执行流程  
1. **选题生成**    
   - 参考references/trending-topics.md（需用户提前提供）    
   - 输出3个选题，附理由（如“近期LLM工程化关注度上升”）  
  
2. **大纲构建**    
   - 必须使用SCQA结构（Situation-Complication-Question-Answer）    
   - 每部分标注预期字数（总字数≤2500）  
  
3. **正文撰写**    
   - 语气遵循references/tone-guide.md    
   - 关键概念首次出现时加粗（**AI Skills**）    
   - 每800字插入1个相关emoji（从assets/emoji-palette.json选）  
  
4. **标题优化**    
   - 调用scripts/generate-headlines.py生成3种标题：    
     a) 悬念式（“为什么90%的团队用错了Prompt？”）    
     b) 数字式（“3个技巧让你的AI输出提升5倍”）    
     c) 冲突式（“别再瞎用Skills！这篇指南能救你”）  
  
5. **合规检查**    
   - 调用scripts/check-banned-words.py扫描全文    
   - 替换禁用词（如“最”→“非常”，“绝对”→“通常”）  
  
## 输出格式  
### 最终交付物  
- **推荐标题**：[选择最优标题]  
- **完整文章**：含Markdown格式  
- **敏感词报告**：共发现X处，已替换Y处  
  
## 注意事项  
- 禁止使用“揭秘”、“震惊”等标题党词汇  
- 技术细节需准确，不确定时标注“[待验证]”  
- 文末必须包含互动引导（“欢迎留言讨论”）
```

> ✅ **关键设计**：
>
> * • 流程分阶段，避免AI一次性处理过多任务
> * • 每步有明确输入/输出，便于调试
> * • 将“风格”、“合规”等抽象要求转化为可执行规则

---

## 四、子模块实现：让Skill真正可靠

### ▶ 1. 语气规范（references/tone-guide.md）

```
# 语气指南  
  
## 基调  
- 专业但不学术：用“我们”代替“笔者”，用“能跑”代替“可执行”  
- 亲切但不轻浮：可适度幽默，但避免网络梗（如“绝绝子”）  
  
## 示例  
✅ 好：“这个方案能帮你省下80%的调试时间。”  
❌ 差：“此方案具备显著的调试效率增益。”
```

### ▶ 2. 敏感词检测脚本（scripts/check-banned-words.py）

```
#!/usr/bin/env python3  
import sys  
import re  
  
# 加载敏感词库  
with open('../references/banned-words.txt', 'r') as f:  
    BANNED_WORDS = set(line.strip() for line in f if line.strip())  
  
REPLACEMENTS = {  
    "最": "非常",  
    "绝对": "通常",  
    "第一": "领先",  
    "免费": "无需付费"  
}  
  
def scan_and_replace(text: str) -> tuple[str, int]:  
    found_count = 0  
    result = text  
      
    for word in BANNED_WORDS:  
        if word in result:  
            found_count += len(re.findall(word, result))  
            if word in REPLACEMENTS:  
                result = result.replace(word, REPLACEMENTS[word])  
      
    return result, found_count  
  
if __name__ == "__main__":  
    input_text = sys.stdin.read()  
    cleaned, count = scan_and_replace(input_text)  
    print(f"发现{count}处敏感词，已处理。")  
    print(cleaned)
```

> 💡 **优势**：比纯Prompt更可靠，且可随时更新词库。

### ▶ 3. 标题生成器（scripts/generate-headlines.py）

该脚本接收文章摘要，返回三种标题。此处简化为核心逻辑：

```
def generate_headlines(summary: str) -> dict:  
    return {  
        "suspense": f"为什么{summary[:10]}...？",  
        "number": f"3个技巧让你{summary[:15]}",  
        "conflict": f"别再{summary[:8]}！这篇指南能救你"  
    }
```

实际项目中可接入NLP模型优化。

---

## 五、测试：模拟真实创作场景

### ▶ 测试指令

```
请用公众号写作Skill帮我写一篇关于AI Skills的文章，面向开发者，强调工程价值。
```

### ▶ 期望输出片段

```
### 推荐标题  
别再瞎用Skills！这篇指南能救你  
  
### 完整文章  
凌晨两点，你还在和AI“掰扯”……    
（正文略，含**AI Skills**加粗、每段适量💡🚀等emoji）  
  
### 敏感词报告  
共发现2处，已替换2处（“最”→“非常”）
```

### ▶ 调试技巧

* • **若标题不够吸引人**：优化`generate-headlines.py`中的模板
* • **若emoji过多**：调整SKILL.md中的“每800字1个”规则
* • **若结构松散**：在`structure-templates/scqa.md`中提供更详细示例

---

## 六、进阶：组合其他Skill，打造内容流水线

单个Skill强大，组合更无敌。例如：

1. 1. **先用`research-skill`**：自动抓取知乎/掘金热门话题
2. 2. **再用`wechat-writing-assistant`**：基于热点生成文章
3. 3. **最后用`seo-optimizer-skill`**：添加关键词、meta描述

通过主控Agent协调，实现：

> **热点发现 → 内容生成 → 多平台分发** 的全自动流程

---

## 七、给内容团队的架构建议

如果你是技术负责人，可这样推广：

### 1. **建立内容Skill仓库**

```
git clone your-company/content-skills ~/.claude/skills/
```

新人入职即获得全套写作工具。

### 2. **版本化内容规范**

当品牌调性调整时，只需更新`tone-guide.md`，所有AI输出自动同步。

### 3. **A/B测试标题效果**

记录不同标题的打开率，反哺`generate-headlines.py`的算法优化。

---

## 结语：让AI成为你的“数字分身”

一个好的写作Skill，不只是提高效率，更是**固化你的专业影响力**。

当你的经验被封装成Skill，即使你不在，AI也能以你的风格、你的标准、你的价值观持续产出内容。

而这，正是个人IP与企业知识资产化的终极形态。

> **下期预告**：《Skills + MCP：让AI调用真实世界服务（如微信支付、数据库）》  
> 我们将打通AI与外部系统，实现“AI自动发布公众号文章”闭环。

---