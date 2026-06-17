---
title: Python 生态的 Rust 革命：UV + Ruff + Ty 极速工具链实战
author: 青之物语
date: 
url: https://mp.weixin.qq.com/s?__biz=MzU5MjcxNjU5Mg==&mid=2247483914&idx=1&sn=7ac1ce3db7654525b704868a687658de&chksm=ffbb22acda67ca32d6565ad2eca02b522722c388a72e5ca5bf933b213528457b60f7b48dd6d6&mpshare=1&scene=24&srcid=0326nZhwNqJNZ28vx3UPUl0T&sharer_shareinfo=43a04f37d2b8c06b897d14be164db13f&sharer_shareinfo_first=43a04f37d2b8c06b897d14be164db13f#rd
---

> 当 Python 遇见 Rust，一场关于速度的范式转移正在发生。

## 一、为什么 Python 开发者需要 Rust 工具？

传统 Python 开发工具链长期被**慢**字困扰：

* • `pip install` 解析依赖耗时数分钟
* • `black` + `flake8` + `isort` 串行运行，单文件检查超过秒级
* • `pyright` / `mypy` 启动缓慢，类型检查耗时

**Rust** 的出现改变了这一切——它带来的不仅是内存安全，还有**编译型语言的极致性能**。今天介绍的三款工具，正是 Rust 在 Python 生态的代表作。

---

## 二、三剑客实战指南

### 1. UV：全能包管理与项目工具

**定位**：pip + poetry + rye 的**终极替代品**

**核心优势**：

* • 依赖解析速度**比 pip 快 10-100 倍**
* • 内置虚拟环境管理
* • 支持 lockfile 锁定版本
* • 同时接管项目初始化 + 包管理

**对比实测**（安装 50 个包的 Django 项目）：

| 工具 | 耗时 |
| --- | --- |
| pip | 42 秒 |
| poetry | 38 秒 |
| **uv** | **3.5 秒** |

**实战命令**：

```
# 安装 (一行命令)  
curl -LsSf https://astral.sh/uv/install.sh | sh  
# 或  
pip install uv  
  
# 项目初始化 + 自动创建虚拟环境  
uv init my-project  
cd my-project  
  
# 添加依赖（自动匹配版本）  
uv add fastapi uvicorn sqlalchemy  
uv add --dev pytest httpx ruff  
  
# 安装所有依赖  
uv sync  
  
# 运行脚本  
uv run python main.py  
uv run pytest
```

**配置文件** (`pyproject.toml`)：

```
[project]  
name = "my-project"  
version = "0.1.0"  
requires-python = ">=3.11"  
dependencies = [  
    "fastapi>=0.100",  
    "uvicorn>=0.23",  
]  
  
[tool.uv]  
dev-dependencies = [  
    "pytest>=7.4",  
    "ruff>=0.1",  
]
```

---

### 2. Ruff：史上最快的 Python Linter & Formatter

**定位**：代码检查 + 格式化 + 自动修复

**核心优势**：

* • **比 black 快 10-100 倍**（Rust 编译型）
* • 兼容 PEP8、isort、flake8、pylint 等规则
* • 内置自动修复功能，一键修复 200+ 问题

**实测性能**：

| 工具 | 1000 文件检查时间 |
| --- | --- |
| flake8 + isort + black | ~45 秒 |
| **Ruff** | **0.3 秒** |

**实战命令**：

```
# 安装  
pip install ruff  
# 或  
uv add --dev ruff  
  
# 检查 + 自动修复  
ruff check . --fix  
  
# 格式化  
ruff format .  
  
# 仅检查特定规则  
ruff check . --select E,F,W --ignore E501
```

**配置文件** (`pyproject.toml`)：

```
[tool.ruff]  
line-length = 88  
target-version = "py311"  
  
[tool.ruff.lint]  
select = ["E", "F", "W", "I", "N", "UP", "B", "C4", "ASYNC"]  
ignore = ["E501"]  
  
[tool.ruff.format]  
quote-style = "double"  
indent-style = "space"
```

---

### 3. Ty：下一代 Python 类型检查器

**定位**：Python 类型检查的**极速替代品**

**核心优势**：

* • 由 Ruff 团队开发，共享 Rust 基础设施
* • 比 Pyright 快 **5-10 倍**
* • 与 Ruff 深度集成，共享配置
* • 更友好的错误提示

**实战命令**：

```
# 安装  
pip install ty  
# 或  
uv add --dev ty  
  
# 类型检查  
ty check .  
  
# 与 ruff 配合（ruff check 会自动运行 ty）  
ruff check .
```

**配置文件** (`pyproject.toml`)：

```
[tool.ty]  
# 与 ruff 共用配置  
files = ["."]  
exclude = ["*.pyc", ".git", "venv"]  
  
[tool.ruff.lint]  
select = ["E", "F", "W", "I", "N", "UP", "B", "C4", "ASYNC", "TCH", "TYD"]  
# TYD = ty 规则
```

---

## 三、组合技：一套配置走天下

完整的 `pyproject.toml`：

```
[project]  
name = "my-api"  
version = "0.1.0"  
requires-python = ">=3.11"  
dependencies = [  
    "fastapi>=0.100",  
    "uvicorn>=0.23",  
    "sqlalchemy>=2.0",  
    "pydantic>=2.0",  
]  
  
[tool.uv]  
dev-dependencies = [  
    "pytest>=7.4",  
    "httpx>=0.24",  
    "ruff>=0.1",  
    "ty>=0.1",  
]  
  
[tool.ruff]  
line-length = 88  
target-version = "py311"  
  
[tool.ruff.lint]  
select = ["E", "F", "W", "I", "N", "UP", "B", "C4", "ASYNC", "TCH", "TYD"]  
  
[tool.ruff.format]  
quote-style = "double"
```

**日常开发流程**：

```
# 克隆项目后一键环境搭建  
uv sync  
  
# 开发时并行运行检查（快到无感）  
ruff check . && ruff format . && ty check .  
  
# 安装新依赖  
uv add requests  
uv add --dev black
```

---

## 四、Astral：幕后推手是谁？

### 核心团队

三款工具均由 **Astral** 公司开发，创始人 **Charlie Marsh** 曾任职于 Spotify、Datadog，是 Python 工具链领域的领军人物。

Astral 致力于构建**下一代 Python 开发者工具**，愿景是让 Python 开发拥有与 Go、Rust 同等的高速体验。

### 工具演进说明

* • **Rye**：Astral 早期推出的项目工具，已停止维护，功能已合并到 **UV**
* • **UV**：现在Astral 主推的全能工具，同时支持项目管理和包安装
* • **Ruff**：代码检查与格式化事实标准
* • **Ty**：最新推出的类型检查器，与 Ruff 一脉相承

### OpenAI 收购传闻

> ⚠️ **截至本文发稿时**：疑似 OpenAI 计划收购 Astral 。

---

## 五、总结：为什么你应该切换？

| 场景 | 传统方案 | Rust 方案 | 收益 |
| --- | --- | --- | --- |
| 包安装 | pip / poetry | uv | 快 10-50 倍 |
| 代码检查 | black+flake8+isort | ruff | 快 10-100 倍 |
| 类型检查 | pyright / mypy | ty | 快 5-10 倍 |
| 项目管理 | poetry / conda | uv | 统一入口 |

**一句话推荐**：

> 用 **uv** 管项目 + 装包，用 **ruff** 写码，用 **ty** 检查类型——Python 开发体验焕然一新。