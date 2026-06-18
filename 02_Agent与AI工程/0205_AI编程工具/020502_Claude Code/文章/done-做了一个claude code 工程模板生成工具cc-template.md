> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: 做了一个claude code 工程模板生成工具cc-template
author: 魔都小码农
date: s xiaozhans xiaozhan
url: https://mp.weixin.qq.com/s?__biz=MzkwMTMwMDkyMA==&mid=2247484119&idx=1&sn=101a6d4133c0c7c56a5ca3c8425566e6&chksm=c1ad5e786c3ba896ce49a4bbfdb6333f9456e5bb4c735bc9c0b0fd86ec1db4839e801ab4be5b&mpshare=1&scene=24&srcid=0525EevvOpCwC85fpcCqoAXH&sharer_shareinfo=f312d4bf94619cabbd91cb245bf4a2df&sharer_shareinfo_first=f312d4bf94619cabbd91cb245bf4a2df#rd
---

现在是真不想在自己电脑上做东西，token消耗不起。昨晚刚用了一会字节的coding plan，就达到5小时限量，然后早上继续用的时候，可能是上下文太大了，3轮对话就又达到限量了，后面薅羊毛使用了trae和codebuddy中的免费额度才断断续续完成了任务。

这两天过星期，上海这边一直下雨，就想着在家做一个claude code 项目工程模板生成工具，既能让自己熟悉claude code的使用，也能帮不熟悉的小伙伴一键生成可用的项目模板。

github：https://github.com/kainan-tek/cc-template

先贴一下readme文档，后面有使用实例。

```
# CC Template
Claude Code 项目工程模板生成工具，通过模块化配置解决重复配置和质量不稳定两大痛点。
## 它做什么？
当你使用 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 开发项目时，通常需要编写 `CLAUDE.md` 来指导 AI 的行为、配置 `.claude/settings.json` 来控制权限、设置 `.pre-commit-config.yaml` 来保证代码质量。每次新项目都要重复这些工作，而且配置质量参差不齐。
本模板通过**模块化**解决这个问题：
1. **选择模块** → 自动生成 `CLAUDE.md`、`settings.json`、`.pre-commit-config.yaml`、`.gitignore` 等配置文件2. **变量替换** → 项目名称、技术栈等信息自动填入模板3. **依赖自动补全** → 选了 `lang-python` 会自动带上 `core`
## 快速开始
```bash# Python 项目（推荐新手使用预设）python scripts/init.py /path/to/new-project --preset preset-python
# C++ 项目python scripts/init.py /path/to/new-project --preset preset-cpp
# Shell 项目python scripts/init.py /path/to/new-project --preset preset-shell
# 最小配置（仅 core 模块）python scripts/init.py /path/to/new-project --preset preset-minimal```
运行后会交互式地询问项目名称、描述、技术栈等信息，然后在目标目录生成配置文件。
## 模块
模块分为三种类型：**core**（必选基础）、**general**（通用规范）、**language**（语言规范）。
| 模块 | 类型 | 说明 ||------|------|------|| core | core（必选） | 行为准则（Karpathy 四原则）、通用编码规范、项目信息模板 || git-convention | general | Conventional Commits 规范、branch 命名、commit-msg hook || testing | general | 测试策略、命名规范 `test_<功能>_<场景>_<预期>`、覆盖率要求 || code-review | general | 审查标准、PR 描述模板、pre-push 提醒 || api-design | general | 按 `api_style` 变量选择 REST/GraphQL/gRPC 规范 || automation | general | 按 `ci_platform` 变量选择 GitHub Actions/GitLab CI、MCP 服务器配置 || security | general | 安全编码准则、gitleaks 密钥检测、`.env.example` 模板 || lang-python | language | Python 编码规范、pytest/unittest、ruff/black 格式化 || lang-cpp | language | C++ 编码规范、clang-format || lang-shell | language | Shell 编码规范、ShellCheck、bats 测试 |
## 使用方式
```bash# 交互式（逐步询问项目信息和模块变量）python scripts/init.py /path/to/project
# 使用预设（一键配置，最常用）python scripts/init.py /path/to/project --preset preset-python
# 指定模块（依赖自动补全，如 lang-python 会自动带上 core）python scripts/init.py /path/to/project --module core,testing,lang-python
# 追加模块到已有项目（自动检测已安装模块，跳过重复内容）python scripts/init.py --module security /path/to/existing-project
# 预览模式（只看会生成什么文件，不实际写入）python scripts/init.py --diff --preset preset-python /path/to/project
# 非交互模式（CI/脚本调用，所有变量使用默认值，全局变量通过 --var 传入）python scripts/init.py --preset preset-python --non-interactive \  --var PROJECT_NAME=my-api --var PROJECT_DESCRIPTION="My API" \  /path/to/project```
### 生成的文件
| 文件 | 说明 ||------|------|| `CLAUDE.md` | Claude Code 行为指导文件，按章节聚合各模块的规范片段 || `.claude/settings.json` | Claude Code 权限配置（如允许 `Bash(pytest)`） || `.claude/commands/*.md` | 自定义 slash commands（如 `/commit`、`/pytest`） || `.pre-commit-config.yaml` | Git pre-commit hooks 配置 || `.gitignore` | Git 忽略规则 || 其他模板文件 | 如 `pyproject.toml`、`CMakeLists.txt`、`.env.example` 等 |
### 已有项目接入
自动判断写入目标：**共享文件不存在时写共享文件，存在时自动回退到 local 文件**。无需手动指定，直接运行即可。
| 文件 | 共享文件不存在 | 共享文件已存在 ||------|-------------|-------------|| 行为指导 | 写 `CLAUDE.md` | 写 `CLAUDE.local.md`（已有时询问追加/覆盖） || 权限配置 | 写 `.claude/settings.json` | 深度合并到 `.claude/settings.json`（只加不删） || `.pre-commit-config.yaml` | 创建 | repo URL 去重后追加 || `.gitignore` | 创建 | 行级去重后追加 || 其他模板文件 | 创建 | 跳过 |
> `CLAUDE.md`/`settings.json` 提交 git（团队共享）；`CLAUDE.local.md` gitignored（个人本地）。
## 预设
| 预设 | 包含模块 ||------|---------|| preset-minimal | core || preset-python | core + git-convention + testing + code-review + security + lang-python || preset-cpp | core + git-convention + testing + code-review + security + lang-cpp || preset-shell | core + git-convention + testing + code-review + security + lang-shell |
## 开发
```bashpip install -e ".[dev]"pytest```
```

在生成CLAUDE.md文件的其中一条行为准则中使用了github上星标很高的Andrej Karpathy 大神的 LLM 编码行为准则，翻译后的规则如下：

```
## 行为准则
基于 Andrej Karpathy 的 LLM 编码观察，遵循以下四原则：
### 1. 先思考再编码
- 不确定时先问，不静默选择解读- 多种理解全部呈现，由用户决定- 有更简方案直说，不为了"完整性"添加复杂度- 不清楚就停，宁可多问一次也不猜错
### 2. 简单至上
- 不添加未请求的功能- 单次使用不做抽象，三行重复代码优于过早抽象- 200 行能变 50 行就重写- 不为假设的未来需求预留扩展点
### 3. 精准修改
- 不顺手改周边代码- 匹配现有风格，不引入风格不一致的改动- 无关死代码提及但不删- 自己造成的废弃代码必须清理
### 4. 目标驱动执行
- 用测试定义成功标准- 多步骤任务先列计划（步骤 → 验证检查）- 每步验证后再进入下一步
```

说一下怎么使用吧

```
python scripts/init.py E:\Project\git_proj\AudioDemo\audio_test_client --preset preset-cpp
```

这个是将模板内容添加到已有项目audio\_test\_client，最后的preset-cpp表示是C++项目，如果是python项目，改成preset-python即可。新项目也是一样的操作。

生成的CLAUDE.md文件内容如下，已经尽量精简了。

```
<!-- module:core:1.0.0:project-info -->## 项目信息
- **项目名称**：Audio Test Client- **项目描述**：专业的 Android 系统级音频测试工具，专为车载 AAOS 音频开发与测试设计，支持录音、播放、回环测试和参数配置- **技术栈**：C++; AAOS Audio; Android AudioRecord 和 AudioTrack Native API
<!-- module:core:1.0.0:behavior-guidelines -->## 行为准则
基于 Andrej Karpathy 的 LLM 编码观察，遵循以下四原则：
### 1. 先思考再编码
- 不确定时先问，不静默选择解读- 多种理解全部呈现，由用户决定- 有更简方案直说，不为了"完整性"添加复杂度- 不清楚就停，宁可多问一次也不猜错
### 2. 简单至上
- 不添加未请求的功能- 单次使用不做抽象，三行重复代码优于过早抽象- 200 行能变 50 行就重写- 不为假设的未来需求预留扩展点
### 3. 精准修改
- 不顺手改周边代码- 匹配现有风格，不引入风格不一致的改动- 无关死代码提及但不删- 自己造成的废弃代码必须清理
### 4. 目标驱动执行
- 用测试定义成功标准- 多步骤任务先列计划（步骤 → 验证检查）- 每步验证后再进入下一步
<!-- module:core:1.0.0:coding-standards -->## 编码规范
- 避免嵌套超过 3 层- 公共 API 必须有文档- 复杂逻辑必须加注释说明意图- 错误必须被处理，不允许静默忽略
<!-- module:lang-cpp:1.0.0:coding-standards -->### C++ 编码规范
- 构建目标命名加项目前缀（如 `<project>_<library>`），避免冲突- 依赖管理使用构建系统的标准方式（CMake: `FetchContent` / Android: `android_library deps`）- 命名遵循 Google C++ Style Guide- 前向声明优先于 `#include`
<!-- module:core:1.0.0:testing -->## 测试规范
- 每个功能必须有对应的测试- 测试必须独立，不依赖执行顺序或其他测试的副作用- Bug 修复必须先写失败测试，再修复- 测试命名应描述场景和预期，而非实现细节
<!-- module:testing:1.0.0:testing -->## 测试策略
- 单元测试覆盖核心逻辑，目标覆盖率 ≥ 80%- 比例建议：单元 70% / 集成 20% / E2E 10%- 命名规范：`test_<功能>_<场景>_<预期>`（如 `test_login_invalid_password_returns_401`）- Bug 修复必须加回归测试- 不为测试而测试，每个测试应验证明确的行为
<!-- module:lang-cpp:1.0.0:testing -->### C++ 测试规范（Google Test）
- 测试目录：`tests/`- 测试文件命名：`test_<module>.cpp`- 测试函数命名：`TEST(<Feature>, <Scenario>_<Expected>)`- EXPECT 断言（非致命）优先于 ASSERT（致命）- 使用 TEST_F 定义 fixture 测试，共享测试数据- 使用 EXPECT_* 做正常验证，ASSERT_* 做前提条件检查
<!-- module:git-convention:1.0.0:git-convention -->## Git 规范
- Commit 格式：`<type>(<scope>): <description>`- Type：feat(新功能) / fix(修复) / docs(文档) / refactor(不改变行为) / test(测试) / chore(构建/工具)- 分支命名：`<type>/<description>`（如 `feat/user-auth`）- PR 标题与 commit message 格式一致
<!-- module:code-review:1.0.0:code-review -->## 代码审查
### 审查标准
- 审查优先级：正确性 > 安全性 > 可维护性 > 其他- 只列问题项，严重程度：严重/建议/可选
### PR 描述模板
PR 描述应包含以下内容：
- **变更说明**：简要描述本次变更的目的和内容- **变更影响范围**：列出受影响的模块/功能- **测试**：描述如何验证本次变更
<!-- module:security:1.0.0:security -->## 安全规范
- 不硬编码密钥、密码、Token- 定期运行依赖安全扫描- 及时更新有漏洞的依赖- 锁定依赖版本- 使用 `.env.example` 记录所需环境变量
```

这就是这个模板生成工具，详细的情况可以看设计文档：docs\superpowers\specs\2026-05-24-cc-project-template-design.md

需要的小伙伴可以下载试用，也可以自己按需修改使用。

最近感觉agent有一个趋势很明显了 -- 多agent协作。以后可能就是一个人管理一群agent协作干活了，有一个词叫agent蜂群，挺形象的。

洗洗睡吧，明天又要开启牛马生活了。