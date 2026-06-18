---
title: 【python】pyproject.toml详解&常用UV命令大全
author: 梦无矶测开实录
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg2NDA0NTQyMw==&mid=2247489862&idx=1&sn=58565914c8e1d736eb6d69570527dbb1&chksm=cf9c7045cbcf300f916840f2d52dd1e04cc9240848932b21ecfb373aed632eceaf4aa9b080f4&mpshare=1&scene=24&srcid=1230ydrqy0bvB0h3SAli5oMZ&sharer_shareinfo=45967e4d2237739dc964606ca2e33416&sharer_shareinfo_first=45967e4d2237739dc964606ca2e33416#rd
---
> 已吸收至：[[09_电脑工具/0901_开发工具与CLI/090101_uv/090101_核心知识点/uv项目入口与发布边界|uv项目入口与发布边界]]


"UV 是近十年来 Python 生态系统发生的最好的事情。"

—— 社区开发者评价

# 一、pyproject.toml详解

紧接上篇：[万字长文解析Python环境管理：我从pip换到uv的迁移实操](https://mp.weixin.qq.com/s?__biz=Mzg2NDA0NTQyMw==&mid=2247489822&idx=1&sn=c29e25460e482139c46b26f6a59b85ee&scene=21#wechat_redirect)

上篇因为篇幅原因，没有单独去讲解pyproject.toml里面的参数配置，`pyproject.toml` 是Python项目的现代核心配置文件，如果只是简单的个人项目，只需要着重看[project]下的dependencies就行。

## 📝 综合示例

下面是一个使用 **Poetry** 和 **uv** 工具链的 `pyproject.toml` 完整示例，每个部分我都加上了详细的注释，后面还梳理了一份表格方便查阅。

```
# pyproject.toml
# 这是现代Python项目的核心配置文件，遵循TOML格式。
# 它统一了项目元数据、依赖声明、构建配置和各类开发工具的设置。

### 第一部分：项目元数据与依赖声明 (PEP 621标准) ###
# 此部分由 `[project]` 开头，是Python打包生态系统（如pip, build, uv）识别的标准部分。
[project]
# 项目名称，在PyPI（Python包索引）上必须是唯一的。这是安装时使用的名字。
name = "fastapi-data-service"
# 项目当前版本，应遵循语义化版本规范（主版本.次版本.修订号）。
version = "0.1.0"
# 项目的简短描述，通常会显示在PyPI列表和`pip show`命令的输出中。
description = "一个基于FastAPI的数据查询服务"
# 指定项目的详细说明文档文件，构建包时会将其包含进去。
readme = "README.md"
# 项目作者列表，支持多个作者。格式为“名字 <邮箱>”。
authors = [
    "梦无矶 <mengwuji@163.com>",
    "Awesome Team <team@163.com>"
]
# 项目使用的开源许可证。这里使用内联表格式指定了MIT许可证的文本。
license = { text = "MIT" }
# **项目运行时依赖**：这是一个列表，定义了项目运行所必需的第三方库及其版本约束。
# 当用户安装此包时，这些依赖会被自动安装。
dependencies = [
    "fastapi>=0.100.0",      # Web框架，要求至少0.100.0版本
    "sqlalchemy>=2.0.0",     # ORM工具，要求至少2.0.0版本
    "pydantic>=2.0.0",       # 数据验证与设置管理，要求至少2.0.0版本
    "uvicorn[standard]>=0.24.0", # ASGI服务器，并安装其“standard”额外依赖（如日志、压缩）
]
# **可选依赖组**：用于定义按功能分组的、非必须的依赖。
# 用户可以按需安装，例如：`pip install “fastapi-data-service[mysql,dev]”`。
optional-dependencies = {
    # “mysql”组：如果用户需要使用MySQL数据库，则安装此组
    "mysql" = ["mysqlclient"],
    # “postgres”组：如果用户需要使用PostgreSQL数据库，则安装此组
    "postgres" = ["psycopg2-binary"],
    # “dev”组：开发过程中需要的工具（如测试、代码检查、格式化）
    "dev" = ["pytest", "mypy", "black"],
}

### 第二部分：构建系统定义 (必填) ###
# 此部分告诉Python的构建工具（如`pip`、`build`）应该用什么工具来“打包”这个项目。
[build-system]
# 构建本项目的包所需要的最小依赖集合。这里指定了setuptools和wheel。
requires = ["setuptools>=61.0", "wheel"]
# 指定实际执行构建操作的后端工具。`setuptools.build_meta`是setuptools提供的标准后端。
build-backend = "setuptools.build_meta"

### 第三部分：Poetry工具专用配置 (如果使用Poetry) ###
# 以下配置仅在使用Poetry作为依赖管理器时才需要。Poetry会优先读取此部分。
[tool.poetry]
name = "fastapi-data-service"  # 项目名（通常与[project]部分一致）
version = "0.1.0"               # 版本（通常与[project]部分一致）

# Poetry管理的项目主要依赖，作用类似于 `[project].dependencies`。
[tool.poetry.dependencies]
python = ">=3.9,<3.13"  # 指定项目兼容的Python解释器版本范围
fastapi = ">=0.100.0"
sqlalchemy = ">=2.0.0"

# Poetry管理的“开发依赖组”。这些是仅在开发时需要的包，不会打包进最终发布的项目中。
[tool.poetry.group.dev.dependencies]
pytest = "^7.4.0"    # 测试框架，“^7.4.0”表示允许7.4.0及以上但不包括8.0.0的版本
black = "^23.0"      # 代码格式化工具
mypy = "^1.8.0"      # 静态类型检查工具
ipython = "^8.18.0"  # 增强的交互式Python shell，用于调试和探索

### 第四部分：开发工具配置 (消灭散落的配置文件) ###
# 以下配置允许将各种开发工具的设置统一集中在此文件中。

# Pytest（测试框架）配置
[tool.pytest.ini_options]
testpaths = ["tests"]          # 告诉pytest在“tests”目录下寻找测试文件
addopts = "-v --tb=short"      # 默认命令行选项：`-v`详细输出，`--tb=short`简短的错误回溯

# Mypy（静态类型检查器）配置
[tool.mypy]
python_version = "3.9"           # 按哪个Python版本的语法进行类型检查
warn_return_any = true           # 对返回类型为`Any`的函数发出警告
warn_unused_configs = true       # 对配置文件中未使用的配置节发出警告
ignore_missing_imports = true    # 忽略无法解析的导入（常用于第三方库或无类型存根的情况）

# Black（代码格式化工具）配置
[tool.black]
line-length = 88                 # 每行最大字符数（Black的默认值）
target-version = ['py39']        # 目标Python版本，决定使用哪些语法特性
include = '\.pyi?$'              # 一个正则表达式，匹配需要格式化的文件（.py和.pyi文件）

# isort（导入语句排序工具）配置
[tool.isort]
profile = "black"                # 使用与Black兼容的配置预设，避免冲突
line_length = 88                 # 每行长度，与Black保持一致
```

下表详细解析了它的主要部分、配置项及作用，并附有示例说明。

| **配置区块** | **核心配置项** | **详细解析与作用** | **典型示例** |
| --- | --- | --- | --- |
| **项目元信息**`[project]` | `name` | **项目名称** ，用于包索引和安装。**必填**，必须是唯一的。 | `name = "my-awesome-lib"` |
|  | `version` | **项目版本** ，遵循语义化版本。**强烈建议**。 | `version = "0.1.0"` |
|  | `description` | 项目简短描述，会显示在PyPI上。 | `description = "一个用魔法解决一切问题的库"` |
|  | `authors` | 作者列表，列表项为 `name <email>` 格式的字符串。 | `authors = [ "张三 <zhangsan@email.com>" ]` |
|  | `dependencies` | **项目运行时依赖列表** 。这是定义依赖的**标准位置**。 | `dependencies = [ "requests>=2.25", "pandas~=2.0" ]` |
|  | `optional-dependencies` | **可选依赖组** ，用于按功能分组。 | `optional-dependencies = { gui = ["pyqt5"], cli = ["rich"] }` |
| **构建系统**`[build-system]` | `requires` | **构建项目所需的包列表** ，如 `setuptools`, `hatchling`。 | `requires = ["setuptools>=61.0", "wheel"]` |
|  | `build-backend` | **构建后端的入口点** ，决定如何打包。 | `build-backend = "setuptools.build_meta"` |
| **工具特定配置** | `[tool.poetry]` | **Poetry工具** 的项目配置，包含与 `[project]` 类似但更丰富的元信息。 | ``` [tool.poetry]``name = "my-project"``version = "1.0.0" ``` |
|  | `[tool.poetry.dependencies]` | Poetry管理的**项目主要依赖**。 | ``` [tool.poetry.dependencies]``python = "^3.8"``flask = "^2.0.1" ``` |
|  | `[tool.poetry.group.dev.dependencies]` | Poetry管理的**开发依赖组**。 | ``` [tool.poetry.group.dev.dependencies]``pytest = "^7.0"``black = "^23.0" ``` |
|  | `[tool.pytest.ini_options]` | **Pytest测试框架** 的配置。 | ``` [tool.pytest.ini_options]``testpaths = ["tests"]``python_files = "test_*.py" ``` |
|  | `[tool.mypy]` | **Mypy静态类型检查器** 的配置。 | ``` [tool.mypy]``python_version = "3.9"``ignore_missing_imports = true ``` |
|  | `[tool.black]` | **Black代码格式化工具** 的配置。 | ``` [tool.black]``line-length = 88``target-version = ['py39'] ``` |
|  | `[tool.isort]` | **isort导入排序工具** 的配置。 | ``` [tool.isort]``profile = "black"``line_length = 88 ``` |

# 二、纯 `uv` 版本 `pyproject.toml`

只需要保留 `pyproject.toml` 中的 `[project]`**和**`[build-system]`**这两个核心部分**，以及所有 `[tool.*]` 的开发工具配置（如pytest, black）。

**删除**`[tool.poetry]`**整个区块**，这个是poetry才会用到的，咱们用uv不需要。

因为 `uv` 和 `pip` 等现代Python工具都直接遵循 **PEP 621** 标准，它们只从 `[project]` 部分读取项目信息和依赖。

| 配置区块 | 对 `uv` 是否有用？ | 作用解析 | 建议 |
| --- | --- | --- | --- |
| `[project]` | **✅****核心有用** | `uv` 从此处读取**项目名、版本、运行时依赖、可选依赖组**。 | **必须保留** ，这是依赖管理的源头。 |
| `[build-system]` | **✅****核心有用** | `uv` 和 `pip` 打包、安装时，据此知道如何构建你的项目。 | **必须保留** 。 |
| `[tool.poetry]` | **❌****完全无用** | 这是Poetry工具的**私有配置**，`uv` 会直接忽略。 | **可以安全地全部删除** ，以保持文件简洁。 |
| `[tool.pytest]`**等** | **✅****非常有用** | 这是**开发工具**（如pytest, black, mypy）的配置，与依赖管理器无关。 | **强烈建议保留** ，统一管理工具配置。 |

### 🗑️ 优化后的纯 `uv` 版本 `pyproject.toml`

删除 `[tool.poetry]` 部分后，配置文件瞬间简单了很多，什么™的叫简洁？！（我加了详细注释，看起来东西很多，但是是为了大家方便理解，不信你把注释删掉，你就发现根本没有几行）

```
# pyproject.toml
# 这是为纯 uv 工具链优化的现代Python项目配置文件。
# uv、pip、build 等工具将直接读取此文件中的标准部分。

### 1. 项目元数据与依赖声明 (PEP 621 标准) ###
# 这是最核心的部分，定义了项目的身份和所有依赖。
# uv add, uv sync, uv pip install 等命令都基于此部分工作。
[project]
# 项目在PyPI上的唯一标识名称，也是 `pip install` 时使用的名字。
name = "fastapi-data-service"
# 当前版本号，应遵循“主版本.次版本.修订号”的语义化版本规则。
version = "0.1.0"
# 项目的简短描述，会显示在包索引和 `pip show` 中。
description = "一个基于FastAPI的数据查询服务"
# 指定项目的长描述文件，构建分发包时会包含此文件内容。
readme = "README.md"
# 项目作者/维护者列表，支持多人协作。
authors = [
    "梦无矶 <mengwuji@163.com>",
    "Awesome Team <team@163.com>"
]
# 项目采用的许可证。这里使用内联表格式指定了MIT许可证的文本。
license = { text = "MIT" }
# 【核心】项目运行时必需依赖列表。
# 当用户 `pip install fastapi-data-service` 时，这些包会被自动安装。
# 使用 `uv add <包名>` 命令会自动在此列表中添加条目。
dependencies = [
    "fastapi>=0.100.0",       # Web框架，约束最低版本为0.100.0
    "sqlalchemy>=2.0.0",      # 数据库ORM工具，约束最低版本为2.0.0
    "pydantic>=2.0.0",        # 数据验证与序列化库，约束最低版本为2.0.0
    "uvicorn[standard]>=0.24.0", # ASGI服务器，`[standard]`表示安装额外功能（如日志）
]
# 【核心】可选依赖组（也称为“extras”）。
# 用户可按需安装，例如：`pip install “fastapi-data-service[mysql,dev]”`。
# 使用 `uv add “fastapi-data-service[mysql]”` 可为当前项目安装该组依赖。
optional-dependencies = {
    # “mysql”功能组：为项目添加MySQL数据库驱动支持。
    "mysql" = ["mysqlclient"],
    # “postgres”功能组：为项目添加PostgreSQL数据库驱动支持。
    "postgres" = ["psycopg2-binary"],
    # “dev”开发组：仅开发时需要的工具（测试、检查、格式化等）。
    # 安装命令示例：`uv sync --all-extras` 或 `uv add -e .[dev]`
    "dev" = ["pytest", "mypy", "black", "ipython"],
}

### 2. 构建系统定义 (打包必备) ###
# 这部分告诉Python的打包工具链（如pip、build）如何“构建”你的项目。
[build-system]
# 构建本项目所需的最小依赖集合。通常包括setuptools和wheel。
requires = ["setuptools>=61.0", "wheel"]
# 指定执行打包操作的后端。`setuptools.build_meta` 是使用最广泛的后端。
build-backend = "setuptools.build_meta"

### 3. 开发工具配置 (统一配置，最佳实践) ###
# 以下配置块与包管理无关，但将项目所有工具的设置集中于此，消灭了散落的配置文件。
# 各工具在运行时会自动查找并应用本文件中属于自己的配置。

# Pytest（测试框架）配置
[tool.pytest.ini_options]
# 指定pytest自动发现测试文件的目录。
testpaths = ["tests"]
# 定义运行pytest时自动添加的默认命令行参数。
# `-v`: 详细输出模式，显示每个测试用例的结果。
# `--tb=short`: 当测试失败时，只显示简短的回溯信息。
addopts = "-v --tb=short"

# Mypy（静态类型检查器）配置
[tool.mypy]
# 指定用于类型检查的Python语法版本。
python_version = "3.9"
# 当函数返回类型为模糊的 `Any` 时发出警告，鼓励使用具体类型。
warn_return_any = true
# 对配置文件中可能拼写错误或未使用的配置项发出警告。
warn_unused_configs = true
# 忽略无法找到类型提示（存根文件）的第三方库导入，避免报错。
ignore_missing_imports = true

# Black（代码格式化工具）配置
[tool.black]
# 每行代码允许的最大字符数，超过此长度会自动换行。88是Black的默认值。
line-length = 88
# Black应遵循的Python语法版本，这会影响其格式化决策（例如是否在f-string中使用=号）。
target-version = ['py39']
# 一个正则表达式，指定Black需要格式化的文件类型。
# `\.pyi?$` 匹配以 `.py` 或 `.pyi` 结尾的文件。
include = '\.pyi?$'

# isort（导入语句排序工具）配置
[tool.isort]
# 使用与Black代码格式化工具兼容的预设配置，确保两者排序和格式化规则不冲突。
profile = "black"
# 每行最大长度，与Black的设置保持一致。
line_length = 88
```

### 🚀 如何使用这个配置文件与 `uv`

1. **初始化/安装**：在项目根目录，一个命令即可创建虚拟环境并安装所有依赖（含`dev`组）：

```
uv sync --all-extras
```

1. **添加依赖**：使用 `uv add` 会自动更新 `[project].dependencies`：

```
uv add httpx
```

1. **安装可选功能**：如需安装 `mysql` 支持：

```
uv add "fastapi-data-service[mysql]"
```

正常我们直接使用uv sync下载toml里面的依赖就可以，新增的时候使用uv add就行，本文后面会汇总一个uv常用命令大全

# 三、如何使用 UV 发布到 PyPI？

之前写过一篇：[【python系列】手把手教你在pypi发布自己的包-他人可pip下载](https://mp.weixin.qq.com/s?__biz=Mzg2NDA0NTQyMw==&mid=2247485308&idx=1&sn=302db8160554957d97139f0cb44fd20b&scene=21#wechat_redirect)

但这个方法很容易出现依赖冲突，而且配置也比较繁琐，很容易出错

所以我推荐使用uv管理三方库，发布pypi也使用uv

**整体步骤梳理：**

- 步骤 1：构建分发包

    在项目根目录下执行 `uv build`，UV 会生成 `.whl` 和 `.tar.gz` 文件到 `dist` 目    录。

- 步骤 2：配置 PyPI 凭证

    创建 `.pypirc` 文件配置你的 PyPI 账户信息。

- 步骤 3：发布到 PyPI

    执行 `uv publish` 即可将包上传到 PyPI。

使用 UV来创建一个新项目、管理依赖、构建包，并最终发布到 PyPI，可能会有些步骤和之前的文章有内容重复。

但想要学好学精一样东西，靠的就是极致的重复。

俗话说得好，复杂的事情简单化，简单的事情重复化，重复的事情坚持化，五年10000小时，你就是——专业的熟练工，点到极致就是自动化！

### 分步详细操作指南

接下来，我们详细说明流程中的每一步具体该如何操作。

#### **第一步：配置项目**

确保你的 `pyproject.toml` 文件（现代 Python 项目的核心配置文件）包含以下必要部分：

```
# 1. 定义构建系统 (必填)
[build-system]
requires = ["hatchling"]  # 或 "setuptools>=61.0"
build-backend = "hatchling.build"  # 或 "setuptools.build_meta"

# 2. 定义项目元数据 (必填)
[project]
name = "your-unique-package-name"  # PyPI上必须唯一
version = "0.1.0"                  # 遵循语义化版本
authors = [
    {name = "Your Name", email = "you@example.com"}
]
description = "A brief description of your package"
readme = "README.md"
requires-python = ">=3.8"
license = {text = "MIT"}           # 指定许可证
dependencies = [
    "requests>=2.25",              # 声明依赖
]

# 3. (可选) 发布到测试环境的配置[citation:6]
# 如果你想先发布到 TestPyPI，可以添加：
[[tool.uv.index]]
name = "testpypi"
url = "https://test.pypi.org/simple/"
publish-url = "https://test.pypi.org/legacy/"
```

**注**：项目应具有标准的包结构。对于库项目，建议使用 `src/` 目录布局。

#### **第二步：构建包**

在项目根目录执行构建命令：

```
uv build
```

此命令会读取 `pyproject.toml`，在 `dist/` 目录下生成分发包文件（通常是 `.whl` 轮子和 `.tar.gz` 源码包）。

#### **第三步：上传发布**

发布前，你需要先在 PyPI 官网（或 TestPyPI）注册账户，并在账户设置中生成一个 **API Token** 用于认证。

**发布到生产环境 PyPI：**

```
uv publish --token <你的PyPI-API令牌>
```

**发布到测试环境 TestPyPI：**如果你配置了 `tool.uv.index`，可以直接使用：

```
uv publish --index testpypi
# 系统会提示你输入用户名和密码/令牌
# 用户名填 `__token__`
# 密码填你在 TestPyPI 上生成的令牌
```

或者通过参数指定：

```
uv publish --token <你的TestPyPI令牌> --publish-url https://test.pypi.org/legacy/
```

### UV 发布包到 PyPI 核心命令总结

#### **发布三连招**

发布一个包到 PyPI，最核心的其实就是**三个命令**，它们构成了一个标准工作流：

(我会再写一篇uv发布pypi实操的文章)

| 步骤 | 命令 | 作用与关键点 |
| --- | --- | --- |
| **1. 项目初始化** | `uv init --lib` | 创建符合发布标准的**库项目结构**（含`pyproject.toml`），普通项目用 `uv init`。 |
| **2. 打包构建** | `uv build` | 读取 `pyproject.toml`，在 `dist/` 目录生成**分发包文件**（`.whl`和`.tar.gz`）。**发布前必做**。 |
| **3. 上传发布** | `uv publish --token <API_TOKEN>` | 将 `dist/` 下的包**上传到PyPI**。核心是需要PyPI账户的**API Token**进行认证。 |

**一键式发布流程示例：**

```
# 1. 初始化一个库项目
uv init --lib my-package
cd my-package

# 2. 添加依赖（如果需要）并编辑代码和 pyproject.toml
uv add requests

# 3. 构建包
uv build

# 4. 发布到PyPI（使用你的真实Token）
uv publish --token pypi-你的长长长长令牌字符串
```

# 四、 UV 常用命令大全（核心功能分类）

除了发布，UV 几乎覆盖了 Python 项目管理的所有方面。

以下是按功能分类的核心命令速查表：

| 功能类别 | 常用命令 | 等效传统命令 | 说明 |
| --- | --- | --- | --- |
| **🚀****项目与依赖管理** | `uv init` | `cookiecutter` | **创建新项目** ，生成标准结构。 |
|  | `uv add <package>` | `poetry add` / `pip install` | **添加依赖** 并更新`pyproject.toml`。 |
|  | `uv remove <package>` | `poetry remove` / `pip uninstall` | 移除依赖。 |
|  | `uv sync` | `poetry install` / `pip install -r` | **安装所有依赖** ，同步环境。 |
|  | `uv lock` | `poetry lock` | 生成或更新锁文件 (`uv.lock`)。 |
|  | `uv tree` | `poetry show --tree` | **可视化依赖树** 。 |
| **🐍****Python与环境管理** | `uv python install 3.11` | `pyenv install` | **下载并安装** 指定Python解释器。 |
|  | `uv python list` | `pyenv versions` | 列出所有已安装的Python版本。 |
|  | `uv venv .venv` | `python -m venv .venv` | **创建虚拟环境** ，速度极快。 |
| **📦****PIP兼容接口** | `uv pip install <package>` | `pip install` | **以极速替代pip** 安装包。 |
|  | `uv pip compile requirements.in` | `pip-compile` | 生成锁定的`requirements.txt`。 |
|  | `uv pip sync requirements.txt` | `pip-sync` | 根据锁定文件精确同步环境。 |
|  | `uv pip list` / `uv pip show` | `pip list` / `pip show` | 列出/查看包信息。 |
| **▶️****运行与工具** | `uv run script.py` | `python script.py` (需激活环境) | **在项目环境中运行** Python脚本。 |
|  | `uvx <command>` (如 `uvx ruff`) | `pipx run` | **无需安装，直接运行** PyPI上的任何工具。 |
|  | `uv tool install ruff` | `pipx install` | 全局安装命令行工具。 |
| **⚙️****构建与发布** | `uv build` | `python -m build` | **构建分发包** ，生成`dist/`目录。 |
|  | `uv publish` | `twine upload` | **上传包到PyPI** 。 |

**下期更新：uv发布pypi实操**
