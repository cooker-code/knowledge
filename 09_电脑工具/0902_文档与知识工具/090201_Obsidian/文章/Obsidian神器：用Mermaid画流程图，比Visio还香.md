---
title: Obsidian神器：用Mermaid画流程图，比Visio还香
author: Ai学习的老章
date: 
url: https://mp.weixin.qq.com/s?__biz=MzA4MjYwMTc5Nw==&mid=2649008213&idx=1&sn=a997db76deb4a703b5cf2735a5a6ab4e&chksm=86e8141b2e40dbc2521a001c819ecdddf399f616f5f3237a16d54b0a64eda756dfff2ca369c3&mpshare=1&scene=24&srcid=0125WEI6oyB6HQXMbnjSFAOy&sharer_shareinfo=aa43a83444dce8463459fe45a195f53d&sharer_shareinfo_first=aa43a83444dce8463459fe45a195f53d#rd
---

大家好，我是 Ai 学习的老章

[Obsidian「颜值天花板」Minimal 主题，让你的笔记回归极简美学](https://mp.weixin.qq.com/s?__biz=MzA4MjYwMTc5Nw==&mid=2649007414&idx=2&sn=8b567ee15d12ecde35253fd95218b4b7&scene=21#wechat_redirect)

[Obsidian 官方出手了！3个 SKILL 让 Claude Code 直接帮你画白板、建数据库、写Markdown笔记](https://mp.weixin.qq.com/s?__biz=MzA4MjYwMTc5Nw==&mid=2649007351&idx=1&sn=6741aa50de60479ad6c6f8e0afbdbd0f&scene=21#wechat_redirect)

[Obsidian AI写作神器：一键配置DeepSeek，写作效率飙升1000%！](https://mp.weixin.qq.com/s?__biz=MzA4MjYwMTc5Nw==&mid=2648990755&idx=1&sn=3721f784a6db1d7fb14df0be6e9df0e1&scene=21#wechat_redirect)

今天继续安利一个让我效率起飞的神器组合：**Mermaid + Obsidian**。

以前我画这些图都用 Visio 或者 Draw.io，画完导出图片插入文档。问题是：**图和文档是分离的**，改一个地方两边都要改，久了就懒得维护了。

Mermaid 彻底解决了这个痛点——**用代码画图，图和文档融为一体**。

### 什么是 Mermaid？

Mermaid 是一个基于 JavaScript 的图表工具，用类似 Markdown 的文本语法来定义图表，然后自动渲染成可视化的图形。

官方介绍说得很清楚：

> Mermaid is a JavaScript-based diagramming and charting tool that uses Markdown-inspired text definitions and a renderer to create and modify complex diagrams. The main purpose of Mermaid is to help documentation catch up with development.

翻译成人话：**让文档跟上代码的节奏**。

这货在 2019 年获得了 JS Open Source Awards 的「最令人兴奋的技术应用」奖，目前 GitHub 上有 91.4k+ 被依赖，652+ 贡献者。可以说是图表界的「当红炸子鸡」。

### Mermaid 支持哪些图表？

这是 Mermaid 最强的地方——支持的图表类型多到离谱：

| 图表类型 | 适用场景 | 复杂度 |
| --- | --- | --- |
| **Flowchart（流程图）** | 业务流程、算法逻辑 | ⭐ |
| **Sequence Diagram（时序图）** | API 交互、系统调用 | ⭐⭐ |
| **Class Diagram（类图）** | 面向对象设计 | ⭐⭐ |
| **State Diagram（状态图）** | 状态机、生命周期 | ⭐⭐ |
| **Entity Relationship（ER 图）** | 数据库设计 | ⭐⭐ |
| **Gantt Chart（甘特图）** | 项目进度管理 | ⭐⭐ |
| **User Journey（用户旅程图）** | 用户体验分析 | ⭐ |
| **Git Graph（Git 分支图）** | 版本控制可视化 | ⭐⭐ |
| **Mindmap（思维导图）** | 头脑风暴、知识整理 | ⭐ |
| **Kanban（看板）** | 任务管理 | ⭐ |
| **XY Chart（XY 图表）** | 简单数据可视化 | ⭐ |
| **C4 Diagram（C4 架构图）** | 系统架构设计 | ⭐⭐⭐ |

我日常用得最多的是前 5 种，基本覆盖了技术文档的所有需求。

### 为什么选 Mermaid？

对比一下常见的画图方案：

| 方案 | 优点 | 缺点 |
| --- | --- | --- |
| Visio | 专业、功能全 | 收费、只能 Windows、图文分离 |
| Draw.io | 免费、跨平台 | 图文分离、协作麻烦 |
| Excalidraw | 手绘风格、好看 | 不够正式、复杂图难画 |
| **Mermaid** | 代码即图、版本可控、嵌入文档 | 需要学语法 |

Mermaid 的核心优势：

1. **代码即图**：图表就是文本，可以 Git 版本控制
2. **与文档融合**：Markdown 直接嵌入，不用维护额外的图片文件
3. **跨平台支持**：GitHub、GitLab、Notion、Obsidian、VS Code 都原生支持
4. **学习曲线平缓**：语法简单，10 分钟上手

### Obsidian 原生支持！

好消息是：**Obsidian 原生支持 Mermaid，不需要装任何插件！**

直接在笔记里写 ```` ```mermaid ```` 代码块就能渲染。这一点比 Vega 还方便。

### 快速上手：画第一张流程图

新建一个笔记，输入以下内容：

```
flowchart TD  
    A[开始] --> B{是否理解?}  
    B -->|是| C[继续学习]  
    B -->|否| D[重新阅读]  
    D --> B  
    C --> E[完成]
```

保存后，你会看到一张漂亮的流程图。

**核心语法解释：**

* `flowchart TD`：声明流程图，TD 表示从上到下（Top-Down）
* `A[文字]`：方括号表示矩形节点
* `A{文字}`：花括号表示菱形（判断）节点
* `-->`：带箭头的连接线
* `-->|标签|`：带标签的连接线

### 方向控制

```
TD / TB — 从上到下（默认）  
LR — 从左到右  
RL — 从右到左  
BT — 从下到上
```

比如横向流程图：

```
flowchart LR  
    输入 --> 处理 --> 输出
```

后面我就不截图了

### 节点形状大全

Mermaid 支持非常丰富的节点形状：

```
flowchart LR  
    A[矩形] --> B(圆角矩形)  
    B --> C([体育场形])  
    C --> D[[子程序]]  
    D --> E[(数据库)]  
    E --> F((圆形))  
    F --> G>标签形]  
    G --> H{菱形}  
    H --> I{{六边形}}
```

### 进阶示例：常用图表模板

#### 1. 时序图（Sequence Diagram）

画 API 调用、系统交互最常用：

```
sequenceDiagram  
    participant 用户  
    participant 前端  
    participant 后端  
    participant 数据库  
  
    用户->>前端: 点击登录  
    前端->>后端: POST /api/login  
    后端->>数据库: 查询用户  
    数据库-->>后端: 返回用户信息  
    后端-->>前端: 返回 Token  
    前端-->>用户: 跳转首页
```

**语法要点：**

* `->>` 实线箭头
* `-->>` 虚线箭头
* `participant` 定义参与者

#### 2. 类图（Class Diagram）

画面向对象设计：

```
classDiagram  
    class Animal {  
        +String name  
        +int age  
        +makeSound() void  
    }  
    class Dog {  
        +bark() void  
    }  
    class Cat {  
        +meow() void  
    }  
    Animal <|-- Dog  
    Animal <|-- Cat
```

**关系符号：**

* `<|--` 继承
* `*--` 组合
* `o--` 聚合
* `-->` 关联

#### 3. 状态图（State Diagram）

画状态机、订单流程：

```
stateDiagram-v2  
    [*] --> 待支付  
    待支付 --> 已支付: 支付成功  
    待支付 --> 已取消: 超时/取消  
    已支付 --> 已发货: 商家发货  
    已发货 --> 已完成: 确认收货  
    已完成 --> [*]  
    已取消 --> [*]
```

#### 4. ER 图（Entity Relationship）

画数据库表关系：

```
erDiagram  
    CUSTOMER ||--o{ ORDER : places  
    ORDER ||--|{ LINE_ITEM : contains  
    PRODUCT }|--o{ LINE_ITEM : "ordered in"  
    CUSTOMER {  
        int id PK  
        string name  
        string email  
    }  
    ORDER {  
        int id PK  
        date created_at  
        int customer_id FK  
    }
```

**基数符号：**

* `||--||` 一对一
* `||--o{` 一对多
* `}o--o{` 多对多

#### 5. 甘特图（Gantt Chart）

画项目进度：

```
gantt  
    title 项目开发计划  
    dateFormat YYYY-MM-DD  
    section 需求阶段  
        需求分析: a1, 2026-01-01, 7d  
        需求评审: a2, after a1, 3d  
    section 开发阶段  
        后端开发: b1, after a2, 14d  
        前端开发: b2, after a2, 14d  
    section 测试阶段  
        联调测试: c1, after b1, 7d  
        上线: milestone, m1, after c1, 0d
```

#### 6. 用户旅程图（User Journey）

画用户体验分析：

```
journey  
    title 用户购物体验  
    section 发现  
        搜索商品: 5: 用户  
        浏览详情: 4: 用户  
    section 决策  
        对比价格: 3: 用户  
        查看评价: 4: 用户  
    section 购买  
        加入购物车: 5: 用户  
        填写地址: 2: 用户  
        支付: 4: 用户
```

分数 1-5 表示体验好坏（1 差 5 好），自动生成情绪曲线。

#### 7. 思维导图（Mindmap）

```
mindmap  
    root((AI技术栈))  
        大模型  
            ChatGPT  
            Claude  
            Gemini  
        框架  
            LangChain  
            LlamaIndex  
        部署  
            vLLM  
            Ollama  
        应用  
            RAG  
            Agent
```

### 常见坑点与解决方案

用了很久，总结几个常见的坑：

| 问题 | 原因 | 解决方案 |
| --- | --- | --- |
| 图表不渲染 | 括号不匹配 | 检查 `[]`、`{}`、`()` 是否配对 |
| 「Unsupported markdown: list」 | 列表语法冲突 | `[1.Item]` 不要 `[1. Item]`（去掉点后空格） |
| Subgraph 引用失败 | 用了显示名而不是 ID | `subgraph id["显示名"]` ，引用时用 id |
| 中文乱码 | 编码问题 | 用 Unicode 或确保 UTF-8 编码 |
| 连线交叉太乱 | 布局不合理 | 换方向（LR→TD）或用隐形边 `~~~` |

**特殊字符处理技巧：**

* 文本有空格：用双引号 `["带 空格 的文字"]`
* 文本有引号：用中文引号 `「」`『』`
* 文本有括号：用全角括号 `（）`

### 样式定制

Mermaid 支持自定义节点样式：

```
flowchart LR  
    A[成功]:::success --> B[警告]:::warning --> C[错误]:::error  
    classDef success fill:#2ECC71,color:white  
    classDef warning fill:#F39C12,color:white  
    classDef error fill:#E74C3C,color:white
```

也可以直接给单个节点加样式：

```
flowchart TD  
    A[节点A]  
    style A fill:#90EE90,stroke:#333,stroke-width:2px
```

### 我的使用场景

分享几个实际用法：

1. **技术方案文档**：用流程图描述系统逻辑
2. **API 文档**：用时序图描述接口调用流程
3. **数据库设计**：用 ER 图描述表关系
4. **项目管理**：用甘特图跟踪进度
5. **代码评审**：用类图解释设计模式

最爽的是，这些图就是文本，可以直接 Git 版本控制，团队协作超方便。

### 推荐学习资源

1. **官方文档**（必看）：https://mermaid.js.org/
2. **在线编辑器**：https://mermaid.live/

* 实时预览，还能导出 PNG/SVG

3. **语法速查表**：https://mermaid.js.org/intro/syntax-reference.html
4. **GitHub 集成**：GitHub 原生支持 Mermaid，README 里可以直接用

### 总结

Mermaid + Obsidian 这个组合，彻底解决了我「画图麻烦、图文分离」的痛点。

**优点：**

* 代码即图，版本可控
* Obsidian 原生支持，无需插件
* 支持 14+ 种图表类型，覆盖面广
* 语法简单，10 分钟上手
* 跨平台通用（GitHub/GitLab/Notion 都支持）

**缺点：**

* 复杂图表布局不如可视化拖拽灵活
* 高度定制样式需要学习 CSS

如果你经常写技术文档、画流程图，强烈建议试试 Mermaid。一旦用习惯了，Visio 再也不想打开了。

**制作不易，如果这篇文章觉得对你有用，可否点个关注。给我个三连击：点赞、转发和在看。若可以再给我加个🌟，谢谢你看我的文章，我们下篇再见！**