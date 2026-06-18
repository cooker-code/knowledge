> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020203_Skill/020203_核心知识点/Skill能力封装与治理边界|Skill能力封装与治理边界]]
---
title: 精通 Claude Skills：从入门到创建属于你自己的 AI 超能力
author: 幻影物语
date:
url: https://mp.weixin.qq.com/s?__biz=Mzk0NzcwNTcxMA==&mid=2247484313&idx=1&sn=7e3ae293c9fa6394c22b6b474412d648&chksm=c2d32f526f117272f4cf293432911c723d8e7a7718152919e0723894fd6ad51dbe2e7394366c&mpshare=1&scene=24&srcid=1030KJKzNFvIz0WExcLVybMM&sharer_shareinfo=6ffcefcbdfd88572c495cb084c877fa4&sharer_shareinfo_first=6ffcefcbdfd88572c495cb084c877fa4#rd
---

你是否曾在使用 Claude 时，反复输入相同的背景信息、指令或代码模板？Anthropic 最新推出的 **Claude Skills** 功能，正是为了解决这一痛点，它能让你将特定的知识、指令和流程打包，把 Claude 训练成处理你个人专属任务的“专家”。

这篇文章将带你深入了解 Claude Skills，不仅会剖析其工作原理，更将通过一个手把手的教程，教你如何创建并使用一个自定义 Skill，真正实现工作流的自动化。

## **第一部分：科普篇——深入理解 Claude Skills**

在动手之前，我们需要先理解三个核心问题：什么是 Skills？它的结构是怎样的？它与我们熟知的 Projects 和 Subagents 有何不同？

**1. 什么是 Claude Skills？**

简单来说，Claude Skills 是一种可重复使用的、定制化的指令集。你可以把它想象成一个“技能包”，里面包含了执行特定任务所需的所有信息：

* 指令 (Instructions): 告诉 Claude 如何完成任务的详细步骤。
* 资源 (Resources): 任务所需的背景文件、数据、参考文档等。
* 脚本 (Scripts): 可执行的代码片段，让 Claude 能完成更复杂的编程或数据处理任务。

根据 Anthropic 的官方定义，你可以将 Skills 视为“自定义的入职培训材料”，它能将你的专业知识打包，让 Claude 成为处理你最关心事务的专家。

**2. 一个 Skill 的内部构造**

一个 Skill 的核心是一个名为 `SKILL.md` 的 Markdown 文件，它由两部分组成：

| 结构 | 描述 | 作用 |
| --- | --- | --- |
| **YAML Frontmatter** | 文件顶部的元数据区域，用 `---` 包裹。通常包含 `name` (技能名称) 和 `description` (技能描述)。 | 这部分内容**始终加载**在 Claude 的上下文中。Claude 通过读取这部分简短的描述，来快速判断在当前任务中是否应该激活这个 Skill。 |
| **Markdown Body** | YAML 部分之后的主要内容。包含了完成任务所需的详细指南、操作步骤、代码示例等。 | 这部分内容只有在 Skill 被**触发后**才会加载到上下文中。这种“按需加载”的机制极大地节省了上下文窗口，提高了效率。 |

此外，你还可以在 Skill 文件夹中包含其他文件（如 `.py` 脚本、`.txt` 文本、`.md` 参考文档），并在 `SKILL.md` 中引用它们。

**3. Skills vs. Projects vs. Subagents：如何选择？**

这三者是 Claude 中用于管理上下文和任务的工具，但侧重点不同。下表清晰地展示了它们的区别：

| 特性 | Skills | Projects | Subagents |
| --- | --- | --- | --- |
| **目的** | 自动化可重复的工作流，定义标准操作程序 (SOP)。 | 提供一个包含聊天历史和知识库的独立工作空间。 | 自主执行子任务，并进行独立的上下文管理。 |
| **范围** | 狭窄，专注于特定任务的指南。 | 广泛，覆盖多轮对话的上下文。 | 专注于工作流内部的特定子任务。 |
| **调用方式** | 由 Claude 根据任务**自动调用**。 | **用户手动选择** 要进入哪个项目。 | 主代理**调用子代理**来处理特定任务。 |
| **上下文管理** | 仅在相关时加载技能指令，非常节省上下文。 | 始终维持项目内的文件和指令可见。 | 创建隔离的上下文，防止主对话混乱。 |
| **复用性** | **始终可用** ，并自动应用于匹配的任务。 | 可以作为模板被复制，用于相似的未来工作。 | 按需创建，实例是临时的。 |

**小结：** 如果你想让 Claude **自动**学会并执行一个**固定流程**的任务，用 **Skills**。如果你想为某个**长期话题**（如“季度财报分析”）创建一个独立的、包含所有相关文件的**工作区**，用 **Projects**。如果你想让主代理在处理复杂任务时，能**分派**一些小任务给“临时工”处理，用 **Subagents**。

## **第二部分：实战篇——手把手创建你的第一个 Skill**

接下来，我们将通过一个完整的示例，创建一个名为 `manim-viz` 的 Skill，它的功能是使用 Manim（一个数学动画引擎）来创建数学公式的可视化动画。

### 步骤 1：创建 Skill 文件夹结构

首先，你需要一个存放所有 Skills 的目录。按照 Claude 的规范，这个结构应该是这样的：

```
```
.claude/
└── skills/
    └── manim-viz/
        ├── references/
        │   └── mobjects.md
        ├── scripts/
        │   └── examples.py
        └── SKILL.md
```
```

* 在你的项目根目录创建一个 .claude 文件夹。
* 在 .claude 文件夹内创建一个 skills 文件夹。
* 在 skills 文件夹内，为你的新技能创建一个专属文件夹，我们这里命名为 manim-viz。
* 在 manim-viz 内部，创建核心文件 SKILL.md，以及用于存放额外资料的 references 和 scripts 文件夹（可选）。

### 步骤 2：编写核心文件 `SKILL.md`

这是最关键的一步。打开 `SKILL.md`，并填入以下内容。

```
```
---
name: Manim Visualizer
description: Generates Python code for Manim animations based on user descriptions.
version: 1.0.0
---

# Overview

This skill generates Python code for creating mathematical animations using the Manim library. It takes a user's description of a desired visualization and translates it into a functional Manim scene.

# Instructions

1.  **Understand the User's Goal**: Carefully analyze the user's request to understand the core mathematical concept they want to visualize (e.g., "show a circle transforming into a square," "animate the sine wave").

2.  **Consult Reference Material**: Before writing any code, review the `references/mobjects.md` file to identify the correct Manim Mobjects (shapes, text, etc.) needed for the scene. This file contains a list of common shapes and their usage.

3.  **Use Code Examples as a Base**: Refer to the `scripts/examples.py` file to see standard structures for Manim scenes. Use these examples as a starting point for the new code to ensure correctness.

4.  **Construct the Python Code**: *   Import necessary classes from `manim` (e.g., `from manim import Scene, Circle, Square, Create, Transform`).
    *   Define a new class that inherits from `Scene` (e.g., `class UserAnimation(Scene):`).
    *   Inside the `construct` method, write the logic for the animation.
    *   Add comments to the code to explain key steps.

5.  **Present the Final Code**: Provide the complete, runnable Python code block to the user. Explain briefly what the code does.

# Important Rules

*   Always generate the full Python script, including imports and the class structure.
*   Do not execute the code. Your task is only to generate it.
*   If the user's request is too complex, simplify it to a core concept that Manim can handle and generate code for that.

# Examples

*   **User Input**: "Animate a circle turning into a square."
*   **Expected Output**: ```python
    from manim import Scene, Circle, Square, Transform

    class CircleToSquare(Scene):
        def construct(self):
            # Create a circle
            circle = Circle()
            circle.set_fill(BLUE, opacity=0.5)

            # Create a square
            square = Square()
            square.set_fill(RED, opacity=0.5)

            # Animate the transformation
            self.play(Transform(circle, square))
            self.wait(1)
    ```
```
```

### 步骤 3：准备并打包 Skill

1. 准备附加文件：在 scripts/examples.py 中放入一些完整的 Manim 代码示例，在 references/mobjects.md 中放入关于 Manim 对象的详细文档。

references/mobjects.md

```
```
# Common Manim Mobjects

This document lists some of the most common Manim Mobjects (Mathematical Objects) that can be used in animations.

## Basic Shapes

*   **Circle**: A circle.
    *   `mobject = Circle(radius=1.0, color=BLUE)`
*   **Square**: A square.
    *   `mobject = Square(side_length=2.0, color=RED)`
*   **Rectangle**: A rectangle.
    *   `mobject = Rectangle(width=4.0, height=2.0, color=GREEN)`
*   **Line**: A simple line.
    *   `mobject = Line(start=LEFT, end=RIGHT, color=WHITE)`
*   **Dot**: A point in space.
    *   `mobject = Dot(point=ORIGIN)`
*   **Polygon**: A shape with any number of vertices.
    *   `mobject = Polygon([-1, -1, 0], [1, -1, 0], [1, 1, 0], [-1, 1, 0], color=YELLOW)`

## Text

*   **Text**: For rendering simple text.
    *   `mobject = Text("Hello, Manim")`
*   **Tex**: For rendering LaTeX mathematical expressions.
    *   `mobject = Tex(r"$\int_a^b f(x) \, dx$")`
*   **MathTex**: A more modern version of `Tex`.
    *   `mobject = MathTex(r"c^2 = a^2 + b^2")`

## Graphs

*   **Axes**: A set of axes for plotting.
    *   `axes = Axes(x_range=[-5, 5, 1], y_range=[-3, 3, 1])`
*   **Graph**: A plot of a function.
    *   `graph = axes.plot(lambda x: x**2, color=BLUE)`
```
```

scripts/examples.py

```
```
# Collection of standard Manim scene examples.
# Use these as a template for generating new animations.

from manim import *

# Example 1: Basic Transformation
# A circle transforms into a square.
class BasicTransformExample(Scene):
    def construct(self):
        # Create objects
        circle = Circle()
        square = Square().shift(RIGHT * 2)

        # Show the initial object
        self.play(Create(circle))
        self.wait(1)

        # Transform it
        self.play(Transform(circle, square))
        self.wait(1)


# Example 2: Animating a Graph
# Drawing the graph of a sine wave.
class SineWaveExample(Scene):
    def construct(self):
        # Create axes
        axes = Axes(
            x_range=[-10, 10, 1],
            y_range=[-1.5, 1.5, 1],
            axis_config={"color": BLUE},
        )

        # Create the graph
        graph = axes.plot(lambda x: np.sin(x), color=WHITE)
        graph_label = axes.get_graph_label(graph, label="sin(x)")

        # Animate the drawing
        self.play(Create(axes))
        self.play(Create(graph))
        self.play(Write(graph_label))
        self.wait(2)


# Example 3: Positioning Text and Shapes
# Demonstrates how to place objects on the screen.
class PositioningExample(Scene):
    def construct(self):
        # Create a circle in the center
        circle = Circle()

        # Create a square and place it to the upper right
        square = Square()
        square.to_corner(UR) # UR = UP + RIGHT

        # Create text and place it below the circle
        text = Text("This is the center")
        text.next_to(circle, DOWN, buff=0.5)

        self.play(Create(circle), Create(square), Write(text))
        self.wait(1)
```
```

2. 打包为 ZIP 文件：将整个 manim-viz 文件夹压缩成一个 ZIP 文件。确保 SKILL.md 文件位于压缩包的根级别。 在 macOS 上，右键点击 manim-viz 文件夹，选择“压缩‘manim-viz’”即可。 在 Windows 上，右键点击文件夹，选择“发送到” -> “压缩(zipped)文件夹”。

### 步骤 4：在 Claude 中上传并使用 Skill

1. 打开 Claude 桌面应用。
2. 进入 Settings -> Capabilities (在某些版本中可能直接显示)。
3. 找到 Skills (Preview) 部分，点击 Upload skill 按钮。
4. 将你刚刚创建的 manim-viz.zip 文件拖拽进去或选择上传。
5. 上传成功后，确保该 Skill 旁边的开关是打开状态。

### 步骤 5：见证奇迹！

现在，你的 Claude 已经学会了 Manim！回到主聊天窗口，输入一个相关的指令：

**"请使用 manim-viz 技能为我创建一个欧拉公式的可视化动画。"**

Claude 会：

1. 检测到任务：通过你的指令和它已知的 manim-viz 技能描述，它会自动激活该技能。
2. 读取技能文档：它会加载 SKILL.md 的主体内容，学习如何使用 Manim。
3. 安装依赖：它会执行 pip install manim 命令。
4. 编写和执行代码：它会根据技能中的指南和示例，编写出生成欧拉公式动画的 Python 代码，并在其虚拟环境中执行。
5. 交付成果：最终，它会生成一个 MP4 视频文件，展示一个精美的欧拉公式动画，

你已经成功地将一项复杂的专业技能传授给了 Claude，并且未来可以随时调用，无需再重复解释任何关于 Manim 的细节！

## 结论

Claude Skills 是一个极其强大的功能，它将 AI 从一个通用的对话工具，转变为可以被深度定制的、高效的个人助理。通过将你的重复性工作流程、专业知识和常用资源打包成一个个 Skill，你可以极大地提升工作效率，让 Claude 真正为你所用。

现在，就开始思考你的日常工作中有哪些可以被自动化的流程，然后动手创建你的第一个 Skill 吧！