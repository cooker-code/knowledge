# Git 提交校验与破坏性操作护栏

## 来源
- [[09_电脑工具/0901_开发工具与CLI/090104_Git/文章/done-为什么很多团队开始上 commit 校验？|为什么很多团队开始上 commit 校验？]]
- [[09_电脑工具/0901_开发工具与CLI/090104_Git/文章/done-把 Git 提交变成“可执行规范”：Commit 规范体系与 Husky_Commitlint_Commitizen_Lint-staged 全链路介绍|把 Git 提交变成“可执行规范”：Commit 规范体系与 Husky_Commitlint_Commitizen_Lint-staged 全链路介绍]]
- [[09_电脑工具/0901_开发工具与CLI/090104_Git/文章/done-面试官：Git 如何撤回已 Push 的代码？问倒一大片。。。|面试官：Git 如何撤回已 Push 的代码？问倒一大片。。。]]

## 核心问题
提交规范、hook、commitlint 和撤回 push 的核心不是形式主义，而是把低级错误、临时状态和不可审计变更前移拦截。

## 判断准则
- commit 校验适合拦截格式、范围、临时文件、测试未跑、敏感信息等机械问题，不适合替代人工设计判断。
- 本地 hook、commit-msg 和 CI 应分层：本地拦快错，commit-msg 管消息契约，CI 管共享分支质量。
- 撤回已 push 代码前先判断是否共享分支；共享分支优先 revert，个人分支才考虑 reset + force-with-lease。
- 任何破坏性 Git 操作前必须先看 `git status`、当前分支、远端跟踪、未提交改动和团队协作边界。

## 认知偏差
| 常见错误认知 | 正确理解 |
|---|---|
| commit 校验只是流程洁癖 | 它是把可机械判断的问题前移，不是替代代码评审 |
| 撤回 push 就用 reset --hard 再强推 | 共享分支这样做会改写他人历史，默认应使用 revert |

## 架构/流程图（如有）

```mermaid
flowchart LR
  A["来源文章"] --> B["主问题识别"]
  B --> C["工作流节点"]
  C --> D["验证/排障边界"]
  D --> E["长期准则"]
```

## 待验证缺口
- 补一份项目级 Git 破坏性操作前置检查表，覆盖 reset、rebase、force push、clean 和 worktree remove。
