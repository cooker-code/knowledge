---
title: 一份超实用的 uv 命令总结
author: 完蛋大王
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzk1NzgwNDYzNQ==&mid=2247484607&idx=1&sn=9c8e091908ce3057c3a0260dcacd1406&chksm=c281ad9684b8f71471fea3c829def3cc2e373880995d021c1e983af266e81955a7474e79efdf&mpshare=1&scene=24&srcid=1011YQ1RNWDDkMyYdebVdH7X&sharer_shareinfo=21b41c31e62c9531ecd62e83363c63b8&sharer_shareinfo_first=21b41c31e62c9531ecd62e83363c63b8#rd
---

### 

大家好，我是完蛋大王。

入职新公司已经两个月了，在多个AI项目中疯狂切换。在深入技术细节的同时，我也深刻体会到，高效的工程实践，尤其是项目管理，对于开发效率和团队协作至关重要。

今天，我想聊聊一个在最近困扰我，但最终被我“驯服”的难题：Python的环境与包管理。

#### 混乱的开端：pyenv, pip, uv 的“三国杀”

在快速切换于不同AI项目的过程中，我遇到了一个不大不小却极其烦人的问题。有的老项目用 pyenv 控制Python版本，用 pip + requirements.txt 管理依赖；而新项目则推荐使用 uv。

结果就是，我的大脑里时刻要绷着一根弦：“现在这个项目，我该用 pyenv local 3.10 还是 uv venv --python 3.10？是该 pip install 还是 uv add？”

一不小心，指令敲错，依赖就可能装到全局环境，或者 pyenv 管理的另一个版本里。这种小混乱累积起来，不仅浪费时间，也为项目埋下了不稳定的种子。

在对各种组合进行充分尝试之后，我下定决心：从今往后，我的所有新项目，一律使用 uv 作为Python版本和包管理的唯一工具。 它就像一把瑞士军刀，将过去需要多个工具才能完成的事情，整合到了一起。

下面，就是我总结的一份 uv 实战笔记，希望能帮助大家少走弯路。

#### 第一步：用 uv 统一管理 Python 版本

告别 pyenv，uv 自身就提供了强大的Python版本管理能力。

·列出已安装和可安装的Python版本

```
uv python list  
```

这个命令会清晰地展示你本地有哪些版本，以及远程有哪些版本可供选择。

·安装你需要的Python版本

```
# 安装 3.11 大版本uv python install 3.11  
# 安装精确的 patch 版本uv python install 3.12.0  
# 甚至可以安装 pypyuv python install pypy3.10
```

  

#### 第二步：初始化项目，奠定规范基础

·创建新项目

```
uv init my-ai-project
```

 

·创建时直接指定Python版本（强烈推荐）  
这是最省心的一步，在项目开始时就锁定Python版本。

```
uv init my-ai-project --python 3.10
```

执行后，uv 会自动在项目根目录下生成 pyproject.toml 和 .python-version 两个文件，将版本信息固化下来。

#### 

#### 第三步：避开那些“坑”

·坑点1：项目创建后，如何修改Python版本？  
如果你创建项目时忘了指定版本，后续想修改，直接用 uv venv --python 3.12 可能会有坑。

注意： 如果你指定的版本（如3.12）和 .python-version 文件里的版本不一致，下次你执行 uv run python 这类命令时，uv 会非常“智能”地帮你删掉现有的虚拟环境，并根据 .python-version 里的版本重新创建一个。

正确姿势：

1.先用 pin 命令修改 .python-version 文件。

2.再用 venv 命令创建对应版本的虚拟环境。

```
# 第1步：锁定版本到 .python-versionuv python pin 3.12  
# 第2步：让虚拟环境与之一致uv venv --python 3.12
```

·坑点2：当 pyenv 和 uv 同时存在  
这是最容易出错的地方！如果你之前安装过 pyenv 并且配置了全局环境变量，那么在你用 uv init 创建的工程里打开终端，python 或 pip 命令可能会被 pyenv 的 shims 劫持。

这种情况下，如果你习惯性地执行 pip install requests，这个包会被安装到 pyenv 管理的Python路径下，而不是你当前项目的 .venv 虚拟环境里！

应对方案（请牢记）：在 uv 项目中，永远使用 uv add 来安装依赖，彻底告别直接使用 pip install 的习惯。

#### 第四步：日常依赖管理

·添加依赖  
uv add 会自动把依赖项更新到 pyproject.toml 文件中，这是我们最期望的行为。

```
uv add pandasuv add "fastapi[all]"
```

·移除依赖  
同样，uv remove 也会同步更新 pyproject.toml。

```
uv remove requests
```

·什么时候才用 uv pip install？

1.临时测试： 当你想临时装个库（比如某个代码分析工具）到虚拟环境里用一下，但不想把它记录到项目的配置文件 pyproject.toml 中时。

2.兼容老项目： 当你接手一个只有 requirements.txt 文件的老项目时，可以用它来快速安装所有依赖。

```
uv pip install -r requirements.txt
```

#### 第五步：锦上添花的技巧

·Tips 1：配置国内镜像源，加速下载  
每次都敲 -i 参数太麻烦了。直接在 pyproject.toml 里一劳永逸地配置好。

```
[[tool.uv.index]]url = "https://pypi.tuna.tsinghua.edu.cn/simple"default = true
```

·Tips 2：用 uv sync 保持环境纯净  
当你从git上克隆一个新项目后，最完美的初始化命令就是 uv sync。

```
git clone ...cd ...uv sync
```

uv sync 有一个杀手级特性：它不仅会安装 uv.lock 文件中记录的所有依赖，还会卸载掉当前虚拟环境中所有不在 lock 文件里的包。这确保了你的本地环境和项目定义的环境是100%一致的，对于复现问题和团队协作来说，这一点至关重要。

#### 

#### 总结

从各自为战的 pyenv 和 pip，到大一统的 uv，改变的不仅仅是一个工具，更是一套工作流。它减少了我在环境问题上消耗的精力，让我能更专注于AI应用本身的逻辑。

如果你也曾被Python环境的混乱所困扰，不妨在下一个新项目中，试试 uv 吧。它也许就是你一直在寻找的那个，能带来秩序和清晰的解决方案。