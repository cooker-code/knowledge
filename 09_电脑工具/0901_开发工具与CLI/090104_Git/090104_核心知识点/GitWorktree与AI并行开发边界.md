# Git Worktree 与 AI 并行开发边界

## 来源
- [[09_电脑工具/0901_开发工具与CLI/090104_Git/文章/done-Git Worktree 全面指南：AI Agent 时代的并行开发利器|Git Worktree 全面指南：AI Agent 时代的并行开发利器]]
- [[09_电脑工具/0901_开发工具与CLI/090104_Git/文章/done-Git Worktree，让你的 AI 并行 Coding|Git Worktree，让你的 AI 并行 Coding]]
- [[09_电脑工具/0901_开发工具与CLI/090104_Git/文章/done-我是如何用 Git Worktree + Skill 实现多任务并行开发的|我是如何用 Git Worktree + Skill 实现多任务并行开发的]]
- [[09_电脑工具/0901_开发工具与CLI/090104_Git/文章/done-多Agent协作时代的Git规范|多Agent协作时代的Git规范]]
- [[09_电脑工具/0901_开发工具与CLI/090104_Git/文章/done-我把 worktree 做成了一个顺手的 CLI|我把 worktree 做成了一个顺手的 CLI]]
- [[09_电脑工具/0901_开发工具与CLI/090104_Git/文章/done-专为AI agent并行工作流打造,革命性Git Worktree管理工具Worktrunk|专为AI agent并行工作流打造,革命性Git Worktree管理工具Worktrunk]]
- [[09_电脑工具/0901_开发工具与CLI/090104_Git/文章/done-让 CLAUDE.local.md 在多个 worktree 间共享的 4 种方案|让 CLAUDE.local.md 在多个 worktree 间共享的 4 种方案]]

## 核心问题
AI Agent 并行开发让分支切换、工作区污染和上下文混淆变成高频风险；Worktree 的价值是把任务、目录、分支、所有者和收口证据绑定起来。

## 判断准则
- 一个任务至少绑定一个 branch、一个 worktree、一个 owner 和一份 scope；`main/master` 只做集成，不做开发。
- Worktree 不是“多开目录”这么简单，收口必须包含基线同步、测试、diff 审查、合并、清理和共享配置处理。
- 共享 `CLAUDE.local.md`、`.env`、缓存或本地规则时，要明确哪些放主仓、哪些放全局、哪些通过符号链接或脚本生成。
- Worktree 管理器、虚拟分支工具和原生命令各有边界：自动化越强，越要保留可回退的 Git 状态证据。

## 认知偏差
| 常见错误认知 | 正确理解 |
|---|---|
| Worktree 等于多 clone | Worktree 共享对象库但隔离工作区和索引，适合并行任务，不等于独立仓库 |
| AI 并行只要多开几个 Agent | 没有分支、scope、收口和破坏性操作规则，会放大冲突和返工 |

## 架构/流程图（如有）

```mermaid
flowchart LR
  A["来源文章"] --> B["主问题识别"]
  B --> C["工作流节点"]
  C --> D["验证/排障边界"]
  D --> E["长期准则"]
```

## 待验证缺口
- 把当前常用仓库沉淀一份 worktree 创建、同步、合并、清理和共享本地配置的项目级 SOP。
