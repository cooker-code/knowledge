# Git 工具生态与账号安全边界

## 来源
- [[09_电脑工具/0901_开发工具与CLI/090104_Git/文章/done-有了 Git，为什么还要安装 gh？|有了 Git，为什么还要安装 gh？]]
- [[09_电脑工具/0901_开发工具与CLI/090104_Git/文章/done-GitButler：专为 AI 时代设计的版本控制工具|GitButler：专为 AI 时代设计的版本控制工具]]
- [[09_电脑工具/0901_开发工具与CLI/090104_Git/文章/done-GitAgent：将 AI 智能体的“灵魂”交还给开发者|GitAgent：将 AI 智能体的“灵魂”交还给开发者]]
- [[09_电脑工具/0901_开发工具与CLI/090104_Git/文章/done-【解】GitHub 双因素认证（2FA）技术报告|【解】GitHub 双因素认证（2FA）技术报告]]

## 核心问题
GitButler、gh、GitAgent、GitHub 2FA 等工具都围绕 Git 工作流提效，但它们分别作用在分支管理、平台 API、Agent 资产版本化和账号安全层，不能混为一个“Git 工具推荐”。

## 判断准则
- `gh` 的价值是把 GitHub PR、Issue、认证和仓库操作纳入 CLI；它不替代 Git，只补平台 API。
- GitButler/虚拟分支类工具适合多改动流和 AI 提交辅助，但必须验证导出的真实 Git 分支、冲突处理和团队协作兼容。
- GitAgent 的可取点是把 Agent 配置、规则、技能和记忆版本化；它本体更接近 Agent 工程，放在 Git 节点只记录 Git-native 资产治理边界。
- GitHub 2FA、Passkey、恢复码和 Token 管理是工具链入口安全；自动化 CLI 接入前必须先明确认证方式和恢复路径。

## 认知偏差
| 常见错误认知 | 正确理解 |
|---|---|
| Git 工具越多越现代 | 工具只在降低切换成本、增强审计或减少误操作时有价值 |
| GitHub 账号安全是一次性设置 | 2FA、恢复码、SSH key、PAT 和 OAuth 授权都需要周期性审计 |

## 架构/流程图（如有）

```mermaid
flowchart LR
  A["来源文章"] --> B["主问题识别"]
  B --> C["工作流节点"]
  C --> D["验证/排障边界"]
  D --> E["长期准则"]
```

## 待验证缺口
- 实测 GitButler 虚拟分支导出到普通 Git 分支后的冲突处理；整理 gh 与 GitHub Token 权限最小化清单。
