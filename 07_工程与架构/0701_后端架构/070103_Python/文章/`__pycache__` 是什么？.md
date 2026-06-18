---
title: `__pycache__` 是什么？
author: 代码麻辣烫
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzk0NjY0MTc0NA==&mid=2247496298&idx=2&sn=4889de162643b389dc5052b0c334d3cc&chksm=c2501fce8502549c250d664bb85c622ae3baf6ceedbf7871225a43c7b64c17b8b0893401cbc4&mpshare=1&scene=24&srcid=1030pgY0vS25VGei76D4Esbg&sharer_shareinfo=c1b2d1bad8a44818fc4dc268b003e04b&sharer_shareinfo_first=c1b2d1bad8a44818fc4dc268b003e04b#rd
---

你是否曾经打开过项目文件夹，发现了一个神秘的文件夹名为 `__pycache__`？乍一看，它可能像是多余的文件，但实际上这是 Python 内置的一个智能功能！

## 一言蔽之：Python 为何使用 `pycache`？🤔

Python 是一种解释型语言，这意味着你的代码会在运行前被即时翻译。但每次都这样做会很慢。因此，Python 将你的代码转换成一种中间形式，称为 **字节码**，并将其存储在 `__pycache__` 文件夹中。这样一来，下次你运行程序时，Python 可以直接加载预先编译好的字节码，从而使程序启动得更快。

> ❝
>
> **注意：**加载字节码比每次都解析和编译源代码要快得多。这就是为什么第一次运行可能稍慢一些，但后续运行会非常快！ ⚡

## `pycache` 文件夹里有什么？🔍

打开任何一个 `__pycache__` 文件夹，你会看到以 `.pyc` 结尾的文件。它们的名称包含以下信息：

* **模块名称**：例如 `math`。
* **Python 版本**：如 `cpython-311` 或 `pypy310`。
* **优化级别**：如果使用了优化标志，有时会编码在其中。

例如，一个名为 `math.cpython-312.pyc` 的文件告诉你，它是用 CPython 3.12 编译的。

## Python 何时创建 `pycache`？📅

* **在导入模块时：**每次你导入一个模块（即使是间接导入），Python 会检查 `__pycache__` 中是否存在编译版本。如果不存在，或者源代码已更改，它会编译该模块并将字节码存储在那里。
* **在包中：**对于包含多个模块或子包的项目，你可能会在不同目录层级看到多个 `__pycache__` 文件夹。
* **并非总是为脚本生成：**如果你运行一个没有导入任何模块的独立脚本，可能根本不会生成 `__pycache__`。

## `pycache` 如何加速代码运行？🚀

以下是背后的原理：

1. **首次运行：**Python 解析并编译你的 `.py` 文件为字节码，然后将其保存在 `__pycache__` 中。
2. **后续运行：**Python 检查时间戳或文件哈希值，以确定你的源代码是否已更改。如果未更改，Python 将直接加载预先编译好的字节码，而不是重新编译。

例如，如果我有一个名为 `data_processor.py` 的脚本：

1. 读取我的 `data_processor.py` 文件
2. 将其编译为字节码
3. 将字节码保存在 `__pycache__` 中

在后续运行时，Python：

1. 检查 `data_processor.py` 是否已更改
2. 如果未更改，它将直接加载预先编译好的字节码

这个过程显著减少了模块导入时的开销，尽管它可能不会影响程序的整体执行速度。

## 管理 `pycache`：清理、禁用或重定向？🗑️

### 是否需要删除它？

* **通常不需要：**在我的经验中，最好不要动这些文件。它们有助于代码更快地运行！

### 何时清理：

* **调试时：**如果你怀疑旧的字节码导致了问题，删除 `__pycache__` 会强制 Python 重新编译所有内容。
* **部署时：**许多开发人员会将 `__pycache__/` 和 `*.pyc` 添加到他们的 `.gitignore` 文件中，以保持仓库的整洁。

要递归删除所有 `__pycache__` 文件夹，可以使用（Linux/macOS）：

```
find . -type d -name __pycache__ -exec rm -rf {} +
```

Windows：

```
PS> $dirs = Get-ChildItem -Path . -Filter __pycache__ -Recurse -Directory  
PS> $dirs | Remove-Item -Recurse -Force
```

## 如何禁用字节码生成

* **命令行选项：**使用 `-B` 标志运行脚本：

```
python -B your_script.py
```

* **环境变量：**在你的 shell 中设置 `PYTHONDONTWRITEBYTECODE` 变量：

```
export PYTHONDONTWRITEBYTECODE=1
```

（将此行添加到你的 shell 启动文件中，以实现永久解决方案。）

* **在代码中：**在主脚本的顶部（在任何其他导入之前）放置以下代码：

```
import sys  
sys.dont_write_bytecode = True
```

## 将 `pycache` 重定向到中央位置

如果你希望保留字节码缓存的好处，但又不想让项目目录变得杂乱，你可以从 Python 3.8 开始，设置 `PYTHONPYCACHEPREFIX` 环境变量：

```
export PYTHONPYCACHEPREFIX="$HOME/.cache/cpython/"
```

现在，Python 不会在你的项目中创建多个 `__pycache__` 文件夹，而是会在一个单一目录下镜像你的项目结构。这使得清理或管理缓存文件变得更加容易。

## 深入了解：`.pyc` 文件里有什么？🔍

一个 `.pyc` 文件包含：

* **头部：**包括一个魔术数字（标识 Python 版本）、一个位字段，以及用于时间戳失效检查的时间戳（或用于哈希失效检查的哈希值）。
* **代码对象：**由 Python 虚拟机执行的编译字节码（使用 Python 的 `marshal` 模块进行序列化）。

虽然你不需要了解字节码格式的每一个细节，但了解这些文件是如何使 Python 导入变得高效的，这本身就很有趣。

## 阅读和执行缓存的字节码

对于那些好奇的头脑，想要“透视” `.pyc` 文件，你可以使用 Python 的 `marshal` 模块来读取字节码，甚至执行它。以下是一个简单的脚本示例：

```
import marshal  
from pathlib import Path  
  
def load_pyc(file_path):  
    with Path(file_path).open("rb") as f:  
        header = f.read(16)    
        code = marshal.loads(f.read())  
    return code  
  
bytecode = load_pyc("__pycache__/math.cpython-312.pyc")  
exec(bytecode)
```

这个脚本读取编译后的代码并运行它——即使没有原始源文件，Python 也可以执行字节码。

## 总结：为什么 `pycache` 很重要

`__pycache__` 文件夹是 Python 设计中不可或缺的一部分，它提高了模块导入的速度和整体效率。以下是快速回顾：

* **目的：** 存储编译后的字节码，以节省后续运行的时间。
* **创建：** 在你导入模块时自动生成。
* **管理：** 你可以根据需要安全地删除、禁用或重定向它。
* **版本控制：** 最好将 `__pycache__/` 及相关模式添加到你的 `.gitignore` 文件中。

了解 `__pycache__` 不仅可以解开 Python 开发中一个常被忽视的方面，还能帮助你更好地管理项目的环境，无论你是调试、部署，还是仅仅想保持工作空间的整洁。