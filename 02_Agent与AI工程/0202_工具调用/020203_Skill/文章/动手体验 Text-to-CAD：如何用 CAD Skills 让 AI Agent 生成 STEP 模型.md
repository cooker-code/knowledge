---
title: 动手体验 Text-to-CAD：如何用 CAD Skills 让 AI Agent 生成 STEP 模型
author: 先登AI
date: 乐进乐进
url: https://mp.weixin.qq.com/s?__biz=MzYzMTgxMDM2Mg==&mid=2247484573&idx=1&sn=20e7e3927afa5ac33c8ee2ff4962433e&chksm=f19aff012c536688d60cf7be6edff70c693afa1ed186ecd4c24897c9550af1dd0d7ebcff02fd&mpshare=1&scene=24&srcid=0530bFqKojds0Ds2FnZ7kQIp&sharer_shareinfo=ea04357c6f5c995029a8fa74c15b4a31&sharer_shareinfo_first=ea04357c6f5c995029a8fa74c15b4a31#rd
---

CAD SKILLS PRACTICE

# 动手体验 Text-to-CAD：如何用 CAD Skills 让 AI Agent 生成 STEP 模型

GitHub 上有一个很有意思的开源项目：**text-to-cad**。

一个**CAD Skills**：一套面向 AI Agent 的 CAD、机器人和硬件设计技能库。它能让 Codex、Claude Code 这类 Agent 学会调用 CAD 工具链，完成建模、导出 STEP、几何检查、截图预览、本地 Viewer 查看等一系列工程动作。

这个 Skill 怎么装？怎么用？怎么跑官方给出的 Benchmark 例子？生成的 STEP 模型又该怎么检查？

## 一、先搞清楚：它不是一个单一模型，而是一组 Agent Skills

text-to-cad 仓库里包含多种 Skill，包括：

* CAD：根据自然语言或图片需求生成、修改、检查 CAD 模型；
* CAD Viewer：在本地浏览器里查看 STEP、STL、GLB、URDF、G-code 等文件；
* step.parts：搜索螺钉、轴承、电机、连接器等现成 STEP 标准件；
* URDF / SRDF / SDF：生成机器人结构、MoveIt 规划配置和仿真描述文件；
* G-code：把模型切片成 3D 打印 G-code；
* Bambu Labs：和 Bambu 打印机相关的本地打印流程；
* SendCutSend：面向外协加工平台的文件检查。

所以，这个项目的重点不是“训练了一个 CAD 大模型”，而是“把 Agent 接入 CAD 工程工作流”。

在 CAD 这个 Skill 里，它默认采用 **STEP-first** 的方式：  
优先生成 STEP/STP 作为主要 CAD 产物，STL、3MF、GLB、DXF 等作为辅助输出。

这点对机械设计很重要。因为 STEP 比单纯的 Mesh 更适合进入工程 CAD 流程，后续也更容易被 NX、SolidWorks、Creo、FreeCAD、Onshape 等工具读取和交换。

## 二、最简单的安装方式：用 Skills CLI 安装

如果你使用的是支持 Skills 的 Agent，官方推荐的安装方式是：

bash

```
npx skills install earthtojake/text-to-cad
```

这个命令会把仓库里的各个 Skill 安装到支持的 Agent 环境中。

安装完成后，最好重启一下 Agent，让它重新扫描可用 Skill。

安装之后，用户不需要直接调用底层脚本。正常使用方式是直接向 Agent 提需求，例如：

text

```
Create a single solid STEP model in millimeters.  
The part is a rectangular block, 100 mm long in X, 60 mm wide in Y, and 20 mm tall in Z.  
Center the block on the XY origin, with the bottom face at Z = 0.  
Add four vertical through-holes, each 8 mm in diameter, located at X = +/-35 mm and Y = +/-20 mm.  
Add a 2 mm chamfer to the top perimeter edges only. Do not chamfer the holes.  
Export as a STEP file.
```

如果 CAD Skill 正常生效，Agent 应该会把这段自然语言转成内部 CAD brief，然后编写参数化 Python/build123d 建模代码，生成 STEP 文件，并进行检查和预览。

这和普通 ChatGPT 生成一段代码不一样。

在这个项目里，理想流程是：

text

```
自然语言需求  
→ CAD brief  
→ 参数化建模代码  
→ STEP 文件  
→ 几何检查  
→ 快照预览  
→ CAD Viewer 查看  
→ 根据检查结果修正
```

这才是它真正有价值的地方：不是只“生成”，而是把生成结果放进工程验证链路里。

## 三、如果你用 Codex 或 Claude Code，也可以按插件方式安装

仓库也提供了面向 Codex 和 Claude Code 的插件安装方式。

Codex 的安装命令是：

bash

```
codex plugin marketplace add earthtojake/text-to-cad  
codex plugin add cad@text-to-cad
```

Claude Code 的安装命令是：

bash

```
claude plugin marketplace add earthtojake/text-to-cad  
claude plugin install cad@text-to-cad
```

安装完成后，同样建议重启或重新加载 Agent。  
安装插件并不意味着你只安装了一个“CAD 按钮”。更准确地说，是让 Agent 获得一套可调用的 CAD 工作流说明、脚本入口和验证规范。

Agent 会根据用户的需求判断应该触发哪个 Skill。

例如你说：

text

```
帮我生成一个 80mm 直径、10mm 厚的圆法兰，中间 30mm 通孔，60mm 分度圆上有 6 个 6mm 螺栓孔，导出 STEP。
```

它就应该触发 CAD Skill。

如果你说：

text

```
打开刚生成的 STEP 文件让我检查一下。
```

它就应该触发 CAD Viewer Skill。

如果你说：

text

```
给这个机器人手臂写一个 URDF，并把 STL/STEP 网格挂到对应 link 上。
```

它就可能触发 URDF Skill。

这就是 Skills 的好处：  
不是让一个 Agent 靠“常识”瞎猜，而是给它一套工程任务的执行规范。

## 四、把本地 Skill 链接到你的 Agent

如果你 clone 了仓库，并希望 Agent 使用这个本地版本，而不是远程安装版本，可以用项目提供的开发安装脚本。

在仓库根目录执行：

bash

```
scripts/dev/install-skills-dev.sh --agent codex
```

如果你想查看支持哪些 Agent，可以运行：

bash

```
scripts/dev/install-skills-dev.sh --list-agents
```

项目支持的本地开发目标包括：

text

```
codex  
claude  
gemini  
universal  
project
```

如果你同时想链接到 Codex 和 Claude，可以执行：

bash

```
scripts/dev/install-skills-dev.sh --agent codex --agent claude
```

Winodws系统推荐大家直接用git bash运行sh命令。

如果你想全部安装：

bash

```
scripts/dev/install-skills-dev.sh --all
```

这个脚本的作用不是复制文件，而是创建 symlink。  
这样做的好处是：你修改本地仓库里的 Skill 文件后，Agent 立刻能看到变化，不需要反复复制目录。

安装完成后，记得重启或 reload Agent。

## 五、Benchmark

仓库里有一个 `benchmarks/` 目录，里面放了 10 个典型 CAD 生成任务。

这些例子非常适合用来测试 CAD Skill 的能力边界，这里简单测试5个例子：

1. 矩形校准块，四个通孔；

   ```
   Create a centered 100 x 60 x 20 mm block with four 8 mm vertical through-holes. Add only a 2 mm chamfer on the top outer perimeter.
   ```

   代码写好了现在正在安装脚本依赖。等依赖安装好以后就会运行脚本生成step文件，现在我们来检查一下这个模型。

   codex可以利用已有的技能生成预览step的网页。我们也可以用CAD软件打开这个step文件。

2. 圆法兰，中心孔和环形螺栓孔阵列；

```
Create an 80 mm diameter, 10 mm thick circular flange with a 30 mm central through-bore. Add six 6 mm through-holes on a 60 mm bolt circle and fillet the outside circular edges. 
```

3. L 型支架，带加强筋和两个方向的孔；

```
Create an L-bracket from a base plate and rear vertical plate. Add vertical base holes, horizontal back-plate holes, two triangular gussets, and a filleted base/back transition. 
```

接下来用Claude Code进行尝试。

4. 阶梯轴，带键槽；

```
Create a 120 mm shaft along X with 20/30/20 mm diameter stepped sections. Add end chamfers and a shallow rectangular keyway on top of the middle section. 
```

5. 开口电子外壳，带螺柱；

## 六、用 CAD Viewer 查看模型

生成 STEP 后，如果需要看模型。

CAD Viewer Skill 的作用就是启动或复用本地 Viewer，并返回可以在浏览器中打开的模型链接。

底层命令大致是：

bash

```
npm --prefix scripts/viewer run serve:ensure -- \  
  --root-dir /path/to/root \  
  --file path/to/model.step
```

这里有两个点要注意。

第一，`--root-dir` 很重要。  
它应该指向拥有模型文件的项目根目录，而不是随便指向某个子文件夹。否则 Viewer 可能会认为这是另一个独立项目，从而新开端口或找不到文件。

第二，不要假设固定端口。  
Viewer 会自己探测和复用可用端口，所以应该使用命令实际打印出来的 URL。

如果你通过 Agent 使用，一般只需要说：

text

```
Open the generated STEP file with CAD Viewer and return the review link.
```

Agent 会负责调用 CAD Viewer Skill，并把结果链接返回给你。

## 七、这个项目适合用来做什么？

我认为它目前最适合三类场景。

第一类是 **CAD Agent 工作流研究**。  
如果你关注工业软件 AI、Agent、自动建模、工程设计自动化，这个项目非常值得研究。它展示了一个比较完整的思路：让 Agent 通过技能说明和脚本工具进入 CAD 流程。

第二类是 **参数化零件快速生成**。  
比如支架、法兰、轴、壳体、安装板、夹具、简单装配体，这些规则几何非常适合用它来测试。

第三类是 **机器人和硬件设计文件自动化**。  
它不只支持 CAD，还覆盖 URDF、SRDF、SDF 等机器人描述文件，这让它比单纯的 text-to-3D 项目更贴近机器人和硬件工程。

## 八、它暂时不适合什么？

它不适合直接替代成熟 CAD 软件。  
NX、SolidWorks、Creo、CATIA 这些工业软件的复杂特征建模、装配管理、工程图、公差标注、企业数据管理，不是一个 Skill 项目能马上替代的。

它也不适合做严格工程认证。  
比如承载结构强度、材料失效、疲劳寿命、工艺可制造性、法规合规，这些仍然需要专业工程分析。

它更适合作为“AI Agent 调用 CAD 工具链”的实验平台。  
换句话说，它不是终点，而是一个很好的起点。

## 九、我对这个项目的看法

text-to-cad 最值得关注的，不是它能生成多少漂亮模型，而是它把 CAD AI 带到了另一个方向：

不是让大模型直接替代 CAD 软件，而是让大模型学会使用 CAD 工具链。

这和工业软件 AI 的真实落地路径很接近。

未来工程师可能不会只是在 CAD 软件里点菜单、拖特征、改参数，而是把很多重复流程交给 Agent：

text

```
根据需求生成零件  
导出 STEP  
检查尺寸  
生成快照  
打开 Viewer  
发现问题后自动修正  
搜索标准件  
生成机器人描述文件  
继续进入仿真或制造流程
```

这不是一句话“AI 画图”能概括的。

真正有价值的 CAD AI，一定不是只看起来像，而是要能检查、能修改、能交付、能进入后续工程流程。

text-to-cad 这个项目的意义就在这里：  
它让我们看到了一种更现实的 CAD Agent 形态。

不是炫技式的“文本生成三维模型”，而是面向工程任务的“生成 + 验证 + 查看 + 迭代”。

对工业软件从业者来说，这条路线值得认真跟踪。