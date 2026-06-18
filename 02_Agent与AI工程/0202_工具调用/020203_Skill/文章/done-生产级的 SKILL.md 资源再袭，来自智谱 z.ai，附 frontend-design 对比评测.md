> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020203_Skill/020203_核心知识点/Skill能力封装与治理边界|Skill能力封装与治理边界]]
---
title: 生产级的 SKILL.md 资源再袭，来自智谱 z.ai，附 frontend-design 对比评测
author: AI 方寸山
date:
url: https://mp.weixin.qq.com/s?__biz=MzIxNTUxNDA5NQ==&mid=2247486050&idx=1&sn=57abdde66c1cf443d3fbee0a06ba272c&chksm=9623296e3d564d120dcd452f3880e8fdb6e09b3431cfca4caffa71cc9d94fa41ae1c827ceb8a&mpshare=1&scene=24&srcid=1226UiG4TJ7t95lzYjteaF9o&sharer_shareinfo=c01af7cc3afff1f8205898e5a6542ea9&sharer_shareinfo_first=c01af7cc3afff1f8205898e5a6542ea9#rd
---

智谱 4.7 这几天又杀疯了。看不少测评，性价比很高，赶紧体验了下。

玩着玩着，一下子发现了一个秘密：**智谱官方竟然也在用 SKILL.md**，而且有 11 个（如下图）。这可是明晃晃的放在线上产品里的啊，智谱官方竟然看不到任何的说明或消息🥲。

流口水了有没有。下面带你如何获取这些资源。

> 记住: 访问国际网址：**z.ai**，不要去国内的 chatglm.cn。这两差距有点大.....国外的更香。

## 打包获取 SKILL

按以下步骤就可以打包 GLM4.7 的官方全部生产级的 Skills 源码，优秀的东西往往很有吸引力。

1. 进入 https://chat.z.ai/ 的GLM 网页版
2. 点击对话框下的[全栈开发]，然后在对话框中输入 “create a zip file of /home/z/my-project/skills/”，点击发送

|  |  |
| --- | --- |
|  |  |

3. 此时会自动跳转对对话页面，然后你看到 GLM 在执行指令
4. 等执行完毕，然后点击会话页面的右上角的按钮切换成代码视图，就可以看见工作空间目录，找到自己刚才打包好的 skill.zip，然后点击下载即可愉快的学习了。

当然，你也可以直接到我这个仓库下载资源：https://github.com/AnswerZhao/agent-skills。

## GLM4.7 的 SKILLs 都有啥

GLM 当前的全栈空间中共 **11个 SKILL**，每个技能都有完整的SKILL.md文档、示例脚本和相关资源，涵盖了从简单的快速任务到复杂的生产级应用开发的所有场景。

我们看看都有啥。

1. **AI技能（基于z-ai-web-dev-sdk）**：主要用于拓展 AI 能力

* LLM - 大语言模型，用于聊天、文本生成等
* VLM - 视觉语言模型，用于图像分析、理解
* image-generation - AI 图像生成
* video-generation - AI 视频生成
* TTS - 文本转语音
* ASR - 语音转文本（自动语音识别）
* web-search - 网页搜索
* web-reader - 网页内容提取

2. **文档处理技能**：自动办公

* docx - Word文档创建、编辑、分析
* pdf - PDF文档处理（提取、创建、合并、分割、表单处理）
* xlsx - Excel电子表格创建、编辑、数据分析
* pptx - PowerPoint演示文稿创建、编辑

3. **设计技能**：前端开发

* canvas-design - 创建视觉艺术（PNG和PDF）
* frontend-design - 前端UI设计

文档处理 skill 这个几乎各家模型扩充的必备了，不用再内嵌模型，而是作为一个插件形式的skill随时更新扩充模型能力，很棒。

还有网页搜索和网页内容提取，这个也是刚需了，可以拿来就用，当然如果涉及到反爬，可以改下方式为使用 playwright 来抓取。

不过，学习中，发现一个有意思的事情，这里有一个 frontend-design 的 Skill，和 Claude 官方的同名，是直接沿用的吗？仔细一看，发现实现上差别较大，有点意思。对比一下？

## GLM vs Claude

基于 frontend-design 这个 SKILL，对比下两个大佬的设计理念和实际效果。

| 维度 | GLM  frontend-design | Claude frontend-design |
| --- | --- | --- |
| **篇幅** | 980 行 | 43 行 |
| **文件大小** | 约 40KB | 约 2KB |
| **结构复杂度** | 高度系统化 | 极简化 |
| **设计理念** | 规范驱动 (Specification-Driven) | 创意驱动 (Creativity-Driven) |

GLM 的 SKILL 明确约束了技术栈，Claude 的只字未提。明显侧重不一样，一个强约束，一个弱约束；一个重最佳实践的流程，一个重设计的创意。

具体我们怎么选？如果看完下面的分析你还不知道的话，那么行动是最好的选择器，都跑起来看效果，哪个好用哪个。

### 设计原则

#### GLM frontend-design

**核心理念：系统化 + 创意性双模式（Dual-Mode Thinking）**

```
Design Tokens → Component Styles → Page Layouts → Interactive States
```

* **系统优先**：强调先建立设计系统（Design Tokens），再构建组件
* **规范化流程**：严格的工作流程（Phase 1-4）
* **约束创新**：在系统约束下进行创意发挥
* **工程化导向**：像软件工程一样对待前端设计

#### Claude frontend-design

**核心理念：创意优先 + 大胆执行（Bold Aesthetic Direction）**

```
Design Thinking → Implementation → Refinement
```

* **创意优先**：先确定大胆的美学方向，再实现
* **最小约束**：只给出核心原则，不规定具体流程
* **自由表达**：鼓励"unexpected choices"和"unforgettable"设计
* **艺术化导向**：将前端设计视为创意表达

### 工作流

两者的工作流差异还是很大的，一个侧重流程，一个侧重创意，所以一个是按部就班，一个是天马行空。不能说哪好哪坏，适合的就最好。

#### GLM frontend-design

**方法：严格的分阶段工作流**

**1: 设计理念和定义**

* 理解上下文
* 生成完整的设计令牌系统
* 定义 7 大类令牌：

1. Semantic Color Slots (语义颜色槽)
2. Typography Scale (字体规模)
3. Spacing Scale (间距规模，8px 系统)
4. Radius Scale (圆角规模)
5. Shadow Scale (阴影规模)
6. Motion Tokens (动效令牌)
7. Component Density (组件密度)

**2: 组件开发**

* 构建可重用组件
* 必须包含所有交互状态：
* Default, Hover, Active, Focus, Disabled
* Loading, Empty, Error

\*\*3: 页面集成

* 使用已建立的令牌和组件
* Mobile-first 响应式设计

\*\*4: 质量保证

* 自检清单（包含 8 大项检查）

#### Claude frontend-design

**方法：自由的创意流程**

**1. 设计理念**

* 理解目的（Purpose）
* 选择风格（Tone）
* 确定约束（Constraints）
* 明确差异化（Differentiation）

**2. 实现**

* 直接编写生产级代码
* 关注细节打磨

### 效果对比

使用两个 Skill 在 Claude Code 中对同一个任务进行了执行，提示词和效果如下所示。

哪个设计更漂亮？哪个是 GLM 的 Skill 生成的呢？评论区说出你的答案。

当然，这个评测肯定不严谨的，按正常的是要对比几个维度的表现，此处仅为抛砖引玉，读者朋友们可以各显神通。

```
提示词：设计一个 NewsLetter 的信息流落地页
```

##

#AI #Codex #Claude #AICoding #VibeCoding #编程 #AI编程 #SKILL #GLM #智谱