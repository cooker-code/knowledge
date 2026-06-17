---
title: Skill Forge：我写的 AI Skill 工程化设计框架
author: 极寒AI科技生活
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg4MTk0NjEwNg==&mid=2247484052&idx=1&sn=a35055a5335bbd17c7d01565eaa04711&chksm=cee2afc2bdc7a3740a7f9f7344a47c6965f571405536cf517492a8267319668319abed41c204&mpshare=1&scene=24&srcid=0421BS3twlJ49n1zvA1cOtSs&sharer_shareinfo=7165dc73c8676b694fb63cc06bd159a3&sharer_shareinfo_first=7165dc73c8676b694fb63cc06bd159a3#rd
---

## ◆◆ 我为什么花时间写一个 Skill 设计框架

作为一个重度 AI Agent 用户，我每天和好几个 AI 编程助手打交道——OpenCode、Claude Code、Gemini。它们都有一个共同的能力：通过 **Skill**（技能包）来扩展行为。

但问题来了：写 Skill 这件事本身，就没有一个 Skill 来教你。

这就像一把锤子要打造自己一样——有点递归的荒诞感。我试过凭感觉写，结果 AI 经常跳过我的规则；我试过抄别人的格式，结果触发率惨不忍睹；我甚至试过把所有东西塞进去，结果 Token 爆了，AI 反而更懵。

> 写 Skill 不该靠感觉，该靠工程。

▲ Skill Forge — 从封面开始

▲ 写 Skill 的三大痛点：凭感觉、不触发、无法迭代

## ◆◆ Skill Forge 的核心思路

**Skill Forge** 是我自己写的一个 Skill 工程化设计框架。它的核心理念很简单：把「写 Skill」这件事本身，变成一个有方法论、可验证、可迭代的过程。

### ◆ 六条原则，解决六个具体问题

**决策树路由：**打开 SKILL.md，5 秒内知道下一步该做什么——不再面对空白文档发呆

**渐进式披露：**信息量与需求成正比。简单任务只看必要部分，复杂任务才展开细节

**CLI 委托：**确定性逻辑交给脚本处理，别让 AI 在正则匹配这种事上浪费 Token

**验证闭环：**每个动作都有检查步骤。写了规则？验证它是否被遵循

**Token 压缩：**表格代替段落，省 70% 的空间。SKILL.md 不是小说

**边界表格化：**用查找表代替散文描述，让 AI 精确匹配边缘情况

▲ 六大设计原则：每一条都来自实战踩坑

## ◆◆ 框架本身就是最好的 Skill

这点我觉得最有意思——**Skill Forge 框架本身的 SKILL.md，就是它所教原则的最佳实践**。

我在写这个框架的时候，就一直在用它自己的规则来约束自己：

1.**决策树路由：**SKILL.md 开头就是决策树，告诉你该用哪个子文档

2.**Token 压缩：**主文件控制在 500 行以内，详细内容拆到 references/

3.**验证闭环：**内置 trigger\_eval.json，25 个测试用例自动评估触发准确率

4.**进化日志：**每次修改都记录到 evolution-log.md，下次迭代有据可依

如果你看完觉得「这框架写得还行」，那恰恰证明了这套方法有效——因为它就是用自己的方法写出来的。

▲ 七阶段完整生命周期：从需求发现到自我进化

## ◆◆ 大多数人跳过了最关键的一步

我观察到一个现象：**大部分人写 Skill 直接从「起草」开始**，跳过了前面的需求发现和意图采集。

这就好比你还没问用户想要什么功能，就开始写代码了。结果当然是——你猜的需求和用户真实需求对不上。

所以 Skill Forge 特别加了 **Phase 0：需求发现**。里面有四个实用的技巧：

**投射法：**假设你已经有了这个 Skill，你会怎么用它？倒推需求

**否定法：**列出「用户绝对不会说的话」，反面就是你的触发词来源

**类比法：**找已知的好 Skill 做类比，提取共性模式

**场景还原：**回忆你实际遇到的使用场景，拆解每一步的真实表达

这一步多花 10 分钟，后面少改 3 小时。真的。

▲ 自证设计 × 自我进化：CSO 触发优化 + 进化日志

## ◆◆ 一个 Skill 走天下？

这是另一个实战教训。**OpenCode 用 ~/.config/opencode/skills/，Claude Code 用 ~/.claude/skills/，Gemini 用 ~/.gemini/skills/**——路径不一样，配置格式也不完全一样。

早期我写的 Skill 只能在 OpenCode 上跑。后来想迁移到 Claude Code，发现一堆硬编码路径要改。所以我这次在 Skill Forge 里直接做了三平台支持：

**.env.example：**模板配置文件，git 跟踪但不含敏感信息

**.env.local：**本地实际配置，gitignore 忽略

**变量化路径：**~/{user\_name}/$GIT\_SKILLS\_DIR 替代硬编码路径

现在一套 Skill 文件，三个平台都能用。

▲ 多平台支持：OpenCode / Claude Code / Gemini + 官方 skill-creator 集成

## ◆◆ 没有评估，就没有迭代方向

这是我踩过最大的坑之一。之前改完 Skill 就上线，然后……不知道变好了还是变差了。

Skill Forge 内置了一个评估系统：

**trigger\_eval.json：**25 个测试用例（12 正样本 + 13 负样本），覆盖各种说法变体

**方差分析：**多次运行取方差，避免偶然性误导

**skill-creator 集成：**直接对接 Anthropic 官方评估工具链

有了数据支撑，每一次优化都有依据，而不是「我感觉这样更好」。

▲ 从「凭感觉」到工程化：用经过验证的模式产出高质量 Skill

## ◆◆ 最后说几句真心话

写 Skill Forge 的过程，其实就是一次「元 Skill」的实验——用一个 Skill 来教人怎么写 Skill。过程中遇到的每一个问题（触发不准、Token 爆炸、跨平台不兼容、无法衡量效果），都被提炼成了框架里的规则。

如果你也在为写 Skill 头疼，不妨试试。代码已开源在 GitHub，欢迎提 issue 和 PR。

---

---

开源地址：github.com/includewudi/skill-forge

作者：极寒AI