---
title: 把 Git 提交变成“可执行规范”：Commit 规范体系与 Husky/Commitlint/Commitizen/Lint-staged 全链路介绍
author: 编程演绎
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzNjUwNDA4Ng==&mid=2247484503&idx=1&sn=b3cf39b7bbf89c3290ce1c5e410c48ee&chksm=c36658f2aca8e38395bd65a5a7e9f6e4acff413cd8c03f6cd79596ce7d807ccaae03f241f68b&mpshare=1&scene=24&srcid=0210LtHYrRuxfVaRoIA5FVzT&sharer_shareinfo=342220506e7f36da6da9f8eeeef1984f&sharer_shareinfo_first=342220506e7f36da6da9f8eeeef1984f#rd
---

本规范基于 `Conventional Commits` 思想，配合 `husky + commitlint + commitizen/cz-* + lint-staged` 工具链，在团队内统一提交历史的可读性与可维护性。

* 规范定义：提交信息怎么写（所有人都遵守）
* 工具保障：本地提交时自动校验（尽量少靠“记住规则”）

---

## 0. 概念与工具简介

这一节用于快速理解“为什么要做这些约束”以及“每个工具各自负责什么”。

### 0.1 Conventional Commits（提交规范思想）

`Conventional Commits` 是一套约定式的提交信息格式，用结构化的 `<type>(<scope>): <subject>` 来表达变更意图，带来的收益包括：

* 提交历史更可读，代码审查更高效
* 便于按类型生成变更日志（Changelog）
* 便于自动化流程（发布、版本号、回滚定位）

### 0.2 Git Hook（Git 钩子）

Git Hook 是 Git 在关键动作发生时触发脚本的机制。这里重点关注客户端钩子：

* `pre-commit`：在写入 commit 之前执行，常用于代码检查/格式化/测试
* `commit-msg`：在写入 commit message 之后、最终落库之前执行，常用于校验提交信息格式

### 0.3 Husky（Hook 管理）

`husky` 用来把 hook 以可维护的方式放进仓库（通常在 `.husky/` 目录），让团队成员在本地 `git commit` 时自动触发同一套脚本。

### 0.4 Commitlint（提交信息校验）

`commitlint` 用来校验 commit message 是否满足规则（例如 type 是否在允许列表、header 长度限制等）。它通常挂在 `commit-msg` hook 上，不通过就中止提交。

### 0.5 Commitizen 与适配器（交互式生成）

`commitizen` 提供交互式问答来生成 commit message，降低“靠记忆手写规范”的成本。

* `commitizen` 本身是平台，需要配置一个适配器（adapter）
* 常见适配器：`cz-conventional-changelog`（经典、最小依赖）、`cz-git`（更灵活，可扩展中文/emoji 等）
* 命令：`git-cz`（以及等价的 `cz`）

### 0.6 Lint-staged（只检查暂存区文件）

`lint-staged` 会自动获取 Git 暂存区（staged）的文件列表，并只对这些文件运行指定命令，避免每次提交都全量 lint。

### 0.7 它们如何协作（一次提交的执行顺序）

当你执行 `git commit` 时，通常会按以下顺序发生：

0. `pre-commit`：运行 `lint-staged`（对暂存区文件做校验/格式化）
1. 编辑/生成 commit message：手写或使用 `commitizen`
2. `commit-msg`：运行 `commitlint`（校验 commit message）
3. 通过后提交落库

---

## 1. Commit Message 统一格式（必须）

```
<type>(<scope>)!: <subject>  
  
<body>  
  
<footer>
```

* `<type>`：变更类型（必填，小写）
* `(<scope>)`：影响范围（可选，建议填）
* `!`：破坏性变更（Breaking Change，可选）
* `<subject>`：一句话概括变更（必填）
* `<body>`：为什么改、怎么改、影响点（可选，但推荐在复杂提交中填写）
* `<footer>`：关联 Issue/PR、Breaking Change 说明（可选）

### 1.1 subject（必须）

* 用祈使句/动词开头（中文也等价）：例如“新增…/修复…/调整…”
* 简短、具体、可读：优先说明“做了什么”，而不是“改了什么文件”
* 不要以句号结尾；不要写无意义描述（如“update”、“fix bug”）
* 建议控制在 100 字符内（与 `commitlint` 的 `header-max-length` 保持一致）

### 1.2 body（推荐）

当变更不止一处、存在权衡或影响面较大时，建议写 body：

* 背景/原因：为什么要改
* 方案/实现：怎么改的
* 影响/风险：有哪些兼容性或上线注意点

### 1.3 footer（按需）

* 关联 Issue/任务：`Closes #123`、`Refs #123`
* 破坏性变更（两种写法二选一）：

+ 在 header 里使用 `!`
+ 或在 footer 写：`BREAKING CHANGE: xxx`

---

## 2. type 与 scope 约定（必须统一）

### 2.1 推荐 type 列表（建议与 commitlint 配置保持一致）

| type | 用途 | 例子 |
| --- | --- | --- |
| feat | 新增功能 | `feat(auth): 支持短信登录` |
| fix | 修复缺陷 | `fix(router): 修复重定向死循环` |
| docs | 文档变更 | `docs: 更新安装说明` |
| style | 仅格式/样式（不影响逻辑） | `style: 格式化代码` |
| refactor | 重构（非 feat/fix） | `refactor(core): 拆分校验逻辑` |
| perf | 性能优化 | `perf(list): 优化首屏渲染` |
| test | 测试相关 | `test: 补充登录单测` |
| build | 构建/依赖/打包 | `build: 升级 pnpm 版本` |
| ci | CI 配置与脚本 | `ci: 添加发布流水线` |
| chore | 杂项（不影响业务/功能） | `chore: 更新依赖与脚手架配置` |
| revert | 回滚提交 | `revert: feat(auth): 支持短信登录` |
| wip | 开发中临时提交（慎用） | `wip: 调试登录态` |

建议：`wip` 仅用于个人分支；合并到主分支前应通过 `rebase/squash` 清理成有意义的提交。

### 2.2 scope 取值建议（可选但推荐）

scope 用来说明影响范围，建议使用“模块/领域/目录名”，保持短且稳定：

* 业务域：`auth`、`user`、`order`
* 基础设施：`router`、`store`、`api`
* 工程化：`build`、`ci`、`deps`
* 文档/配置：`readme`、`eslint`、`prettier`

---

## 3. 示例（直接照抄即可）

* 普通提交：`feat: 新增导出功能`
* 带 scope：`fix(auth): 修复 token 刷新失败`
* 破坏性变更：`feat(api)!: 调整用户接口返回结构`

带 body/footer：

```
feat(user): 新增个人资料编辑  
  
- 支持头像上传与裁剪  
- 增加输入校验与错误提示  
  
Closes #123
```

---

## 4. 工具链落地（一）：提交信息规范（commitlint + commitizen）

目标：在本地拦截不合规的 commit message，并提供低成本的规范化提交方式。

### 4.1 安装依赖

```
pnpm add -D husky @commitlint/cli @commitlint/config-conventional commitizen
```

### 4.2 初始化 Husky

```
pnpm exec husky init
```

初始化后通常会发生这些变化（以 Husky 9+ 为例）：

* 新增 `.husky/` 目录（用于存放 hooks）
* 生成 `.husky/pre-commit`（Husky 初始化会放一个默认示例，常见默认内容是 `pnpm test`，可按项目需要改成 `lint-staged`）
* 在 `package.json` 写入/更新 `prepare` 脚本，确保安装依赖后会自动启用 hooks

确保 `package.json` 至少包含：

```
{ "scripts": { "prepare": "husky" } }
```

### 4.3 第一步：只配置 commitlint（先校验，再引导）

0. 创建 `commitlint.config.ts`：

```
export default {  
  extends: ['@commitlint/config-conventional'],  
}
```

> ❝
>
> 后续根据需求可丰富配置

2. 配置 `.husky/commit-msg`：

```
pnpm exec commitlint --edit $1
```

关于 `pnpm exec` 与 `pnpm dlx`：

* `pnpm exec`：执行**当前项目已安装**的依赖（从 `node_modules/.bin` 解析可执行文件）；团队协作下更稳定、版本可控（推荐）
* `pnpm dlx`：临时下载并执行某个包（一次性运行工具时方便），但版本可能随时间变化，不适合作为仓库长期 hooks 依赖

`pnpm exec commitlint --edit $1` 的工作流程（简述）：

0. `git commit` 触发 `commit-msg` hook
1. Husky 执行 `.husky/commit-msg`
2. `pnpm exec` 在当前项目中解析到 `commitlint` 可执行文件
3. `commitlint` 读取 `commitlint.config.*` 等配置
4. `commitlint` 读取 `$1` 指向的 commit message 临时文件并校验
5. 校验失败返回非 0，Git 中止提交；校验通过则继续提交

#### 阶段总结

* 现在使用 `git commit -m "..."` 提交时，如果 message 不符合规范，会被直接拦截
* 但这一步仍依赖“手写 message”，对新人或高频提交场景不够友好

### 4.4 第二步：配置 commitizen（交互式生成 commit message）

当你安装 `commitizen` 后，会获得两个等价命令：

* `git-cz`：完整命令名，更直观（推荐）
* `cz`：简写别名，输入更短

它们都用于启动交互式提交流程，差别仅在命令名。

0. 选择一个适配器（二选一安装）：

```
pnpm add -D cz-conventional-changelog  
# 或  
pnpm add -D cz-git
```

2. 在 `package.json` 中配置适配器：

* 使用 `cz-conventional-changelog`：

```
{ "config": { "commitizen": { "path": "cz-conventional-changelog" } } }
```

* 使用 `cz-git`：

```
{ "config": { "commitizen": { "path": "cz-git" } } }
```

3. 增加统一入口脚本（推荐）：

```
{ "scripts": { "commit": "git-cz" } }
```

提交方式：

```
git add -A  
pnpm commit
```

#### 阶段总结

* 现在团队可以通过交互式方式生成规范 message，出错率显著降低
* `commitlint` 仍然保留：即使有人绕过交互手写 message，也会被校验兜底

---

## 5. 工具链落地（二）：pre-commit 校验（lint-staged）

目标：在提交前只对“暂存区文件”执行校验/格式化，避免把不合规代码带进仓库。

### 5.1 安装依赖

```
pnpm add -D lint-staged
```

说明：`lint-staged` 只负责“挑选暂存区文件并运行命令”，并不自带规则；你需要自行选择并配置 `eslint/prettier/stylelint` 等工具。

### 5.2 配置 lint-staged（示例）

创建 `.lintstagedrc.json`：

```
{  
  "*.{js,jsx,ts,tsx,vue}": ["eslint --fix --cache"],  
  "*.{css,scss,sass,less,vue}": ["stylelint --fix"],  
  "*.{js,jsx,ts,tsx,vue,css,scss,sass,less,json,md,html}": ["prettier --write"]  
}
```

### 5.3 配置 pre-commit hook

编辑 `.husky/pre-commit`：

```
pnpm exec lint-staged
```

#### 阶段总结

* 现在每次提交都会自动对暂存区文件做校验/格式化
* 不通过则提交失败，从源头减少“提交后才发现 lint 报错/格式不统一”的成本

---

## 6. 跨平台一致性（强烈建议）

目标：避免 Windows/macOS/Linux 换行符差异导致的“无意义 diff”。

### 6.1 `.editorconfig`（编辑器层）

```
root = true  
  
[*]  
charset = utf-8  
indent_style = space  
indent_size = 2  
end_of_line = lf  
insert_final_newline = true  
trim_trailing_whitespace = true  
  
[*.md]  
trim_trailing_whitespace = false
```

### 6.2 `.gitattributes`（Git 层）

```
* text=auto eol=lf  
  
*.png  binary  
*.jpg  binary  
*.jpeg binary  
*.gif  binary  
*.pdf  binary
```

### 6.3 `.editorconfig` vs `.gitattributes`（区别与配合）

两者经常一起用，但关注点不同：

* 生效层级不同：`.editorconfig` 影响“编辑器/格式化器在保存时怎么写文件”，`.gitattributes` 影响“Git 在检出/提交时怎么规范化文件”
* 解决问题不同：`.editorconfig` 统一缩进、编码、行尾空白、是否保留末尾换行；`.gitattributes` 统一换行符、标记二进制文件（避免 diff 乱码）、以及 diff/merge 策略等
* 最佳实践：两者配合使用，既约束开发时的写入行为，也保证仓库层面的跨平台一致性

---

## 7. 常见问题（排障速查）

* Husky 不生效：确认执行过 `pnpm install` 且 `prepare` 脚本已运行；必要时检查 `git config core.hooksPath` 是否指向 `.husky`
* 紧急绕过校验：`git commit --no-verify`（仅限紧急情况，后续需补规范提交）
* Windows 下交互提交体验不佳：优先使用 `pnpm commit`，不要强制在 hook 中进入交互式界面