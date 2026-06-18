> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020203_Skill/020203_核心知识点/Skill能力封装与治理边界|Skill能力封装与治理边界]]
---
title: 撸了一个AI资讯周报的skills
author: 阳哥书房
date:
url: https://mp.weixin.qq.com/s?__biz=MzI2NjY5NzI0NA==&mid=2247516733&idx=1&sn=eca92452f73a3920a7bbf3b2f5bfbf55&chksm=eb45a5d735f9d5ff9467a8edfe735789af074065f5513d3202c62bbcdbc5f1a2766df689d21e&mpshare=1&scene=24&srcid=0126zUx8pZziwuLANGc06srB&sharer_shareinfo=af6ad43ecfea9a5b445b82358f701494&sharer_shareinfo_first=af6ad43ecfea9a5b445b82358f701494#rd
---

大家好，我是阳哥。

这几天，发了几个 各类 资讯的内容，之前也提到过，是用的 prompt 来生成的。

今年以来， agent skills 也是如火如荼。

抽了点时间，把这个 “AI 资讯周报” 也给做成了 skills 执行的过程。

这次是在 opencode 里执行的，当然，目前 opencode 里还没有标准的 skills 的设置。

OpenCode 会在这些位置寻找：

```
项目：.opencode/skill/<name>/SKILL.md

全局：~/.config/opencode/skill/<name>/SKILL.md
并兼容 Claude Code 路径：.claude/skills/...、~/.claude/skills/...
```

但这次的设置，是直接在 `weekly-ai-news` 项目里，新建了一个 `SKILL.md` 文件 。 也是可以运行的。文件夹格式如下图：

输出的效果跟之前的 prompt 生成的内容是类似的。

* [AI资讯周报：算力供给从“租用”走向“共建+电力绑定”](https://mp.weixin.qq.com/s?__biz=MzI2NjY5NzI0NA==&mid=2247516726&idx=1&sn=051831235c62c5f6c4c1d3bcced43296&scene=21#wechat_redirect)
* [AI一周进展：Physical AI 上桌、Agent 走向可控、算力与合规同步收紧](https://mp.weixin.qq.com/s?__biz=MzI2NjY5NzI0NA==&mid=2247516660&idx=1&sn=2e9a7a4dafa7e794625f22e673b98f3d&scene=21#wechat_redirect)

最核心的，是这个 `SKILL.md` 文件。

内容如下，大家可以来帮忙看看，还有哪些需要进一步完善的地方。

```
---
name: weekly-ai-news
description: 以固定周期生成可追溯的 AI 资讯周报：采集、去重、排序、综合与发布。
metadata:
  short-description: 每周 AI 资讯研究与周报
  version: 1.0
  author: Lemon
  date: 2026-01-17
---

# 每周 AI 资讯研究 (Weekly AI News)

固定周期（每周）生成结构化 AI 资讯周报，支持可配置主题、来源、语言与输出格式。

## 何时使用

- 需要每周稳定产出 AI 资讯周报
- 需要可追溯来源与可复用模板
- 需要对资讯进行去重、排序与总结

## 输入要求

- 主题范围（如：LLM、Agent、Multimodal）
- 时间范围（默认：近 7 天）
- 语言偏好（默认中文）
- 信息源偏好（官方博客 / 论文 / GitHub / 行业媒体）
- 输出格式（默认 Markdown）

## 输出契约

每个周期创建独立运行目录：`weekly-ai-news/runs/YYYY-MM-DD/`，包含：

1. `sources.jsonl` — 原始来源条目（标题、摘要、链接、发布日期、标签）
2. `items.json` — 去重与筛选后的条目清单（含评分与分类）
3. `report.md` — 周报正文（按模板输出）
4. `qa.md` — 质量检查清单与结果
5. `run_log.md` — 运行记录（范围、过滤规则、异常说明）

`report.md` 使用 block 变量拼接内容：
- `exec_summary_block`
- `milestones_block`
- `model_training_block`
- `inference_system_block`
- `agent_toolchain_block`
- `industry_obs_block`
- `risk_reg_block`
- `risk_notice_block`
- `footnotes_block`

## 工作流程

1. **澄清范围**：确认主题、时间范围、语言与来源偏好
2. **采集来源**：按来源列表收集候选条目
3. **标准化**：统一字段（标题、摘要、日期、来源、标签）
4. **去重聚类**：语义去重 + 事件聚类
5. **评分排序**：按影响力 / 新颖度 / 可信度评分
6. **生成周报**：按模板输出摘要与趋势
7. **质量检查**：完成 `qa.md` 并标注不确定性

## 输出要求

- 覆盖区间需明确标注（起止日期）
- 使用 block 变量输出各段内容（如 `exec_summary_block`、`milestones_block`、`model_training_block` 等）
- 参考资料区使用 `references_block`，按编号对齐展示，且以列表形式输出
- 正文引用使用编号格式（如 `[1]`），同一句可多引用（如 `[1][7]`）
- 同一来源全篇复用同一编号
- 执行摘要与关键里程碑必须包含来源编号引用（如 `[1]`）
- 研究与技术趋势采用“信号→原因→含义”三段式，并为每组添加小标题
- 研究与技术趋势的“信号/原因/含义”需以并列列表形式呈现
- 3.1/3.2/3.3 每节输出 2–4 条；不足 2 条用“延续性信号”兜底补齐
- 重要结论需标注为“事实/观点/推测”
- 条目数控制在 5–12 条（按主题规模调整）
- 涉及融资/合作条款需注明“以官方最终披露为准”
- 使用模板：`templates/report_template.md`

## 质量检查清单

- [ ] 覆盖区间符合要求且明确标注
- [ ] 主题覆盖充分且无明显遗漏
- [ ] 去重完成，避免重复报道
- [ ] 关键条目有可信来源并可追溯
- [ ] 结论与事实已明确区分
- [ ] 引用编号与来源列表一致
- [ ] 风险提示与不确定性已说明

## 安装方法

复制技能到本地 skills 目录：

\```bash
cp -R weekly-ai-news ~/.codex/skills/
\```

或创建软链接：
\```bash
ln -s $(pwd)/weekly-ai-news ~/.codex/skills/weekly-ai-news
\```

## 使用示例

\```
使用 weekly-ai-news 生成过去 7 天 AI 资讯周报，主题包含 LLM 与 Agents，输出中文 Markdown。
\```
```

**延伸阅读：**

* [Skills、Prompt 与 MCP 的区别与联系](https://mp.weixin.qq.com/s?__biz=MzI2NjY5NzI0NA==&mid=2247516690&idx=1&sn=7e2cfbd92c531a811393c8c532ab53d3&scene=21#wechat_redirect)
* [什么是 agent skills](https://mp.weixin.qq.com/s?__biz=MzI2NjY5NzI0NA==&mid=2247516650&idx=1&sn=d472bd3595995c5c6358da913650cde6&scene=21#wechat_redirect)
* [大模型的潜力需要自己去探索](https://mp.weixin.qq.com/s?__biz=MzI2NjY5NzI0NA==&mid=2247516645&idx=1&sn=597b1f9f765d96898935e66404f2e4ce&scene=21#wechat_redirect)
* [kimi 带来的惊喜，ppt 高颜值](https://mp.weixin.qq.com/s?__biz=MzI2NjY5NzI0NA==&mid=2247516610&idx=1&sn=ecefe63e0eb8519eb8816ceed0f0f7de&scene=21#wechat_redirect)
* [智谱GLM4.7的这个功能还是不错滴](https://mp.weixin.qq.com/s?__biz=MzI2NjY5NzI0NA==&mid=2247516604&idx=1&sn=bdde5d61fbec098a70ac86f370abac3b&scene=21#wechat_redirect)
* [一句话变出一套专业级PPT](https://mp.weixin.qq.com/s?__biz=MzI2NjY5NzI0NA==&mid=2247516591&idx=1&sn=4ba7af9e970d6ba9ee88ea198a30af73&scene=21#wechat_redirect)
* [对不起，我错了，GPT 制作 PPT还是有潜力的](https://mp.weixin.qq.com/s?__biz=MzI2NjY5NzI0NA==&mid=2247516579&idx=1&sn=a41c4ebf490cc5cfefbf914e41d3add6&scene=21#wechat_redirect)
* [ChatGPT 5.2 体验](https://mp.weixin.qq.com/s?__biz=MzI2NjY5NzI0NA==&mid=2247516554&idx=1&sn=df9d0f3d69f31d3ebc2b610dcd5e9a52&scene=21#wechat_redirect)
* [GPT 生成 PPT，一切尽在不言中](https://mp.weixin.qq.com/s?__biz=MzI2NjY5NzI0NA==&mid=2247516570&idx=1&sn=cd455ef3d1fa263d83d286b00b58d613&scene=21#wechat_redirect)
* [琢磨建立知识库：向AI请教](https://mp.weixin.qq.com/s?__biz=MzI2NjY5NzI0NA==&mid=2247516547&idx=1&sn=24f0e308b02b23acaf997b1ef72441b0&scene=21#wechat_redirect)
* [琢磨建立个人知识库](https://mp.weixin.qq.com/s?__biz=MzI2NjY5NzI0NA==&mid=2247516530&idx=1&sn=36217e7acb1b92bab088b3d0f8f7a945&scene=21#wechat_redirect)