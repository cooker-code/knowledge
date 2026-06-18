> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020503_Cursor/020503_核心知识点/Cursor工程使用与上下文边界|Cursor工程使用与上下文边界]]
---
title: 告别代码“野路子”：OpenSpec 如何通过规范驱动开发实现1到N，比cursor更加详细
author: AI大模型智能体
date:
url: https://mp.weixin.qq.com/s?__biz=MzkzMzczMjU1Ng==&mid=2247484123&idx=1&sn=90575245050b7f4c251655624a41f898&chksm=c381d21b2e74cb25fd9bdf6e465d6efaa268908db6479d0b7eb2dabc97d5640885a1145b05d7&mpshare=1&scene=24&srcid=1028UdliYYvPbO7TEXxaMV1Y&sharer_shareinfo=67376b851a3e3274d7a9f839a58f959e&sharer_shareinfo_first=67376b851a3e3274d7a9f839a58f959e#rd
---

**为什么需要 OpenSpec？,**当前 AI 编程助手主要面临以下挑战：

1. **上下文缺失：**

   AI 的建议通常基于当前文件或邻近代码，无法理解整个项目的架构、模块依赖关系和设计哲学。
2. **规范不一致：**

   无法强制执行团队的编码规范（如命名法、文件结构、设计模式等），导致 AI 生成的代码风格迥异，破坏了代码库的一致性。
3. **大型项目乏力：**

   在代码库庞大的项目中，AI 更容易“迷路”，生成的建议往往与项目核心逻辑脱节

效果，这是在 [微软发布spec-kit，规格驱动开发，vibe-coding 危机，结合cursor-cli部署实战相册管理](https://mp.weixin.qq.com/s?__biz=MzkzMzczMjU1Ng==&mid=2247484061&idx=1&sn=10c0cbf04db9655d87b43508e49928dd&scene=21#wechat_redirect) 文章中的相册管理，增加一个功能，时间视图

为什么不是spec-kit 因为spec-kit擅长从0到1，有个规范的工程后，大概率也会修改，也就是持续开发(就是日常拿到prd后，在研发的过程中改bug 改需求，很常见)。这个适合 openspec就闪亮登场了。

### 🆚 OpenSpec vs. 其他方案

| 特性 | OpenSpec | spec-kit | Kiro.dev | 无规范 |
| --- | --- | --- | --- | --- |
| **状态管理** | 双文件夹（specs/ + changes/） | 单文件夹 | 多文件夹分散 | 无 |
| **变更跟踪** | 集中式变更文件夹 | 有限 | 分散在各规范中 | 无 |
| **适用阶段** | 0→1 和 1→n 都擅长 | 主要 0→1 | 主要 0→1 | - |
| **跨规范更新** | 优秀（单变更多增量） | 中等 | 困难（分散） | - |
| **归档机制** | 自动合并增量 | 手动 | 手动 | - |
| **AI 工具支持** | 10+ 工具原生支持 | 通用 | 通用 | - |
| **学习曲线** | 低 | 低 | 中 | - |
| **验证工具** | 内置 CLI | 无 | 有限 | - |
| **API 密钥** | 不需要 | 不需要 | 可能需要 | - |

一、安装

前提：Node.js >= 20.19.0 - 使用 `node --version` 检查你的版本

```
安装下 npm install -g @fission-ai/openspec@latest
```

二、使用

在自己的项目工程目录下，输入

```
openspec init
```

这里还是选择自己的编程工具， 我是使用cursor

创建后，也会很贴心的告诉你， 把下面的内容粘贴给cursor

2.1 整体project

Please read openspec/project.md and help me fill it out with details about my project, tech stack, and conventions之后 就把这个工程的目的，架构都写入了 project.md文件里面

3  三步走

### 1：创建变更提案

### /openspec:proposal  目前是以相册为单位进行展示的，可以增加一个切换功能，按照相册排列，也可以按照照片的时间顺序进行排序(颗粒度可以选择天 月 年)

### 运行结束后，可以看到在changes下得到了新的文件，有功能文件  还有 proposal tasks

#### 验证和审查

检查更改是否正确创建并审查提案, 主要是看看...

openspec list

```
Changes:  add-time-sort-view     0/20 tasks
```

 验证规范格式

openspec validate add-time-sort-view

```
Change 'add-time-sort-view' is valid
```

openspec show add-time-sort-view

```
Warning: Ignoring flags not applicable to change: scenarios## 为什么当前应用只支持按相册进行照片展示，用户无法按照时间顺序浏览所有照片。用户希望能够切换视图模式，既可以按相册组 织查看，也可以按照照片的拍摄时间进行时间线浏览，以便更好地回顾和管理照片。
```

#### 完善规格 反复修改规范，直到满足您的需求：

会修改 spec.md and tasks.md.\*  这里我没有太多需要修改规范的，所以略

#### 2. 实施变革

 开始执行  /openspec-apply   add-time-sort-view

这里会比较漫长的执行代码， 可以查看所有变更和进度

openspec view

最后可以看到增加了一个切换的按钮， 可以时间排序，还分不同的维度。

### 3：归档已完成的变更

`/openspec:archive  add-time-sort-view`

### 📚 命令参考

| 命令 | 说明 | 示例 |
| --- | --- | --- |
| `openspec list` | 查看活跃的变更文件夹 | `openspec list` |
| `openspec list --specs` | 列出所有规范 | `openspec list --specs` |
| `openspec view` | 交互式仪表板 | `openspec view` |
| `openspec show` | 显示变更或规范详情 | `openspec show add-2fa` |
| `openspec validate` | 检查规范格式和结构 | `openspec validate add-2fa --strict` |
| `openspec archive` | 归档已完成的变更 | `openspec archive add-2fa --yes` |
| `openspec init` | 初始化 OpenSpec | `openspec init` |
| `openspec update` | 更新指令文件 | `openspec update` |