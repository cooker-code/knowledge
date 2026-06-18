---
title: 现在 uv 可以直接运行远程python脚本了
author: 数据大宇宙
date:
url: https://mp.weixin.qq.com/s?__biz=MzUzNDk1MTc5Mw==&mid=2247488659&idx=1&sn=a0e9afe934fbf947cca009e5fbde3651&chksm=fb90b0a41995286d88c9516f1a34f2e907316c13f36e52401c0cbcbbb11703189ed25557d384&mpshare=1&scene=24&srcid=0415v94lgGDeidR7sMT1WPns&sharer_shareinfo=50fe75bbec11149a1e43f9aeb64a7ae3&sharer_shareinfo_first=50fe75bbec11149a1e43f9aeb64a7ae3#rd
---
> 已吸收至：[[09_电脑工具/0901_开发工具与CLI/090101_uv/090101_核心知识点/uv远程脚本与工具链信任边界|uv远程脚本与工具链信任边界]]


许多人不知道，用 uv 搞 Python 项目压根不需要创建完整项目。比如下面 Python 写的文件分类小工具，往往就只需要一个 `.py` 文件。

这个文件怎么跑起来？ 很简单，只需要执行 `uv run 脚本路径` 命令。

甚至，你也可以直接执行网络上的远程脚本，比如 `uv run https://gitee.com/carson_add/data-u/raw/master/posts/%E9%82%AA%E4%BF%AEuv/uv-scripts/file_sorter.py`。

> 前提是你已经安装了 uv。如果还没安装，可以参考本系列的 `uv 国内逃生指南`，里面有一键绿色安装和国内镜像源加速的方法。

既然通过命令可以直接执行，那我们就可以利用各种方式来间接调用这个命令。比如，在 Windows 上创建一个批处理文件（.bat）的快捷方式，就可以实现双击运行一个 Python 脚本，让它像一个小应用一样弹出一个界面。下面我们一步步来实现这个目标效果。

## 目标效果

* 双击一个快捷方式。
* 屏幕出现一个由 Python 脚本生成的简单应用界面。
* 这个界面由单一的 Python 脚本完成，其大部分功能代码可以借助 AI 生成。

## 1. 添加依赖信息

要让 uv 知道一个孤立的脚本需要哪些外部库，我们需要在脚本文件里声明依赖。

### 操作目的

在 Python 脚本文件的头部，用特定格式的注释声明其依赖的第三方库，这样 uv 在运行时就能自动识别和安装。

### 操作步骤

1. 在电脑上找个地方（比如桌面），新建一个 Python 文件，命名为 `test.py`。
2. 打开文件，输入一行简单的代码，比如 `print("Hello, world!")`。
3. 打开命令行终端（cmd 或 PowerShell），输入以下命令为脚本添加依赖信息：

   ```
   uv add --script "E:\working\code\python\test.py"
   ```

   如果想在添加依赖信息的同时导入包，可以在命令后面加上包名，例如：

   ```
   uv add --script "E:\working\code\python\test.py" "pandas" "numpy"
   ```
4. 命令执行成功后，uv 会自动在 `test.py` 文件的最上方写入类似下面的内容。这个叫“内联依赖元数据”。

   ```
   # /// script
   # requires-python = ">=3.10"
   # dependencies = [
   #     "pandas",
   #     "numpy",
   # ]
   # ///

   print("Hello, world!")
   ```

你也可以手动将这段依赖信息写入脚本文件。本质上，`uv add --script` 命令做的就是这个文本插入工作。

## 2. 脚本执行流程

添加好依赖信息后，就可以用 uv 来运行脚本了。uv 会智能地处理依赖安装和脚本执行。

### 执行命令

在命令行终端中执行：

```
uv run "E:\working\code\python\test.py"
```

### 操作流程图

### 注意： 第一次执行时，uv 会根据脚本头部的依赖信息自动创建环境并安装依赖，之后执行会利用缓存，速度会快很多。

## 3. 优化操作体验

每次都要打开终端输入 `uv run` 命令比较繁琐。我们可以创建一个批处理文件来封装这个命令，实现双击运行。

### 操作目的

创建一个可执行文件，将 `uv run` 命令封装起来，简化运行 Python 脚本的操作步骤。

### 操作步骤

有很多方式可以优化“执行命令”这个操作，这里介绍一种在 Windows 系统上最简单的方式：创建批处理文件（.bat）。

1. 在 Python 脚本（`test.py`）的同级目录下，新建一个文本文档。
2. 将文件名修改为 `运行工具.bat`（注意扩展名要改成 `.bat`），系统提示时点击“确定”。
3. 右键点击 `运行工具.bat`，选择“编辑”，在文件中写入以下内容：

   ```
   @echo off
   uv run "E:\working\code\python\test.py"
   pause
   ```

   **注意：** 请将 `E:\working\code\python\test.py` 替换为你脚本的实际路径。
4. 保存文件。之后，你只需要双击这个 `运行工具.bat`，就会弹出命令行窗口并自动执行你的 Python 脚本。

后续文章会分享更多好用的脚本。

本文相关资源放在仓库上 [https://gitee.com/carson\_add/data-u]
