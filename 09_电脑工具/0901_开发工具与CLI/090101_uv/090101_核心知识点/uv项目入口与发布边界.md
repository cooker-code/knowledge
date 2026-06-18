# uv项目入口与发布边界

## 来源
- [[09_电脑工具/0901_开发工具与CLI/090101_uv/文章/done-Python 生态的 Rust 革命：UV + Ruff + Ty 极速工具链实战|Python 生态的 Rust 革命：UV + Ruff + Ty 极速工具链实战]]
- [[09_电脑工具/0901_开发工具与CLI/090101_uv/文章/done-uv 真快，但 Astral 可能重蹈 Cargo 十年前的覆辙|uv 真快，但 Astral 可能重蹈 Cargo 十年前的覆辙]]
- [[09_电脑工具/0901_开发工具与CLI/090101_uv/文章/done-【python】pyproject.toml详解&常用UV命令大全|【python】pyproject.toml详解&常用UV命令大全]]
- [[09_电脑工具/0901_开发工具与CLI/090101_uv/文章/done-【python】uv发布pypi实操|【python】uv发布pypi实操]]
- [[09_电脑工具/0901_开发工具与CLI/090101_uv/文章/done-一份超实用的 uv 命令总结|一份超实用的 uv 命令总结]]

## 核心问题
uv 不只是更快的 pip；它是否值得成为默认入口，取决于项目初始化、解释器固定、依赖锁定、CI 同步、构建发布和回滚验证能否一起闭环。

## 判断准则
- 新项目可以优先用 uv 统一 `init/add/sync/run/build/publish`，老项目迁移前必须先比较锁文件、Python 版本和 CI 行为。
- 发布包时，`uv publish` 只是上传链路的一段；必须补 TestPyPI、令牌权限、版本号不可复用、构建产物检查和发布后安装验证。
- Ruff/Ty 与 uv 同属 Rust 化 Python 工具链，但问题本体不同：uv 管环境和依赖，Ruff/Ty 管质量和类型，不能把性能收益混写。

## 认知偏差
| 常见错误认知 | 正确理解 |
|---|---|
| uv 很快，所以应该迁移 | 速度只是局部收益；迁移成本、锁文件、发布和排障才决定是否默认使用 |
| uv 可以替代所有 Python 工具 | 只能逐工作流替代，不能一次性宣称替代 pip/venv/pyenv/Poetry/PDM |

## 架构/流程图（如有）

```mermaid
flowchart LR
  A["来源文章"] --> B["主问题识别"]
  B --> C["工作流节点"]
  C --> D["验证/排障边界"]
  D --> E["长期准则"]
```

## 待验证缺口
- 用当前 uv 版本实测 `uv add`、`uv lock`、`uv tree --outdated`、TestPyPI 发布和重复版本失败行为。
