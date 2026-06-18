---
title: nndeploy: 一款AI 落地利器
author: 嵌入式视觉
date:
url: https://mp.weixin.qq.com/s?__biz=MzUzNTg4Nzk3Ng==&mid=2247488339&idx=1&sn=11b7dd34d3cc09470c26c3d856cd8ab0&chksm=fb6144a68f44a412152361d3b559316a62f6d57328f5a4eb33c6d6f6b417c6c5c3955d2a337d&mpshare=1&scene=24&srcid=1121ENaiPDZbLMGOAfqDOII1&sharer_shareinfo=3993253981286d4ff09b526d9ab6af59&sharer_shareinfo_first=3993253981286d4ff09b526d9ab6af59#rd
---
> 已吸收至：[[01_LLM与大模型/0105_模型服务/0105_核心知识点/模型服务网关与部署边界准则|模型服务网关与部署边界准则]]

# nndeploy：一款简单易用和高性能的AI部署框架

## 一、AI落地的"三座大山"，你踩过几座？

做过 AI 落地的开发者都懂：从算法跑通到产品上线，中间隔着的远不止"一行代码"。无论算法、部署还是应用团队，都在被重复且复杂的问题消耗精力，算法/模型部署麻烦一直是工业界想要解决的问题。

### 1. 算法工程师："搞定了SOTA，却卡在了交付"

* 训练完模型不算结束，交付给推理部署团队后还有一堆工作：从训练代码里梳理完整的推理部署流程、写交付文档、和多团队拉通对齐等；
* 辛苦调优出的SOTA算法，想展示惊艳效果让大家体验，结果只能打开命令行敲`python main.py`启动，其他人看着日志一脸茫然。

### 2. 部署工程师："面对的从来不是'单一模型推理'"

* 推理框架碎片化：NVIDIA显卡要用TensorRT、Intel CPU要用OpenVINO、手机端要用MNN/ncnn、华为昇腾要用AscendCL，每个框架用法都不同，光学习接口就耗费大量时间，后续开发和维护更麻烦；
* 模型部署不只是"跑模型"：还得做前处理、后处理，部署工程师需要基于原始算法逻辑，用C++/SIMD/CUDA重新开发这些模块；
* 多模型组合场景复杂：现代AI应用常需多个模型协同，比如换脸、老照片修复、文生图，都要多模型串联。没有框架支持时，得写大量胶水代码衔接，不仅耦合度高、难维护，还没法做有效并行优化；
* 性能优化门槛高：高性能算子、内存优化、并行调度、图优化等工作难度大，即便经验丰富的工程师，也得花几周才能把模型从"能跑"优化到"跑得快"。

### 3. 非AI开发者："有百万级创意，却卡在了技术门槛"

* 脑子里有很多AI产品创意：想给图片编辑工具加"老照片修复"、给短视频APP做"AI换脸"、给电商平台搭"商品图美化"工具；业务逻辑、前后端开发都能搞定，唯独卡在AI功能上。作为非AI领域开发者，只想快速给产品加AI能力，却因不懂AI、不会推理部署，让创意只能停留在想象阶段。

而 nndeploy，就是为解决这些痛点而生的开源AI部署框架——用可视化工作流降低门槛，用统一接口搞定多端推理，用深度优化满足生产需求，让AI落地从"耗时数周"变成"几天搞定"。

> **GitHub 地址**：https://github.com/nndeploy/nndeploy

下图展示了基于 nndeploy 的完整落地流程：拖**拽节点搭建工作流→调参预览效果→导出 JSON 配置→API 调用部署**。全面支持 Linux/Windows/macOS/Android/iOS 平台，下图呈现的是 Android 端 APP 的落地效果：

|  |  |  |
| --- | --- | --- |
| 算法类型 | 桌面端搭建AI工作流 | 移动端部署效果 |
| LLM Qwen 2 |  |  |
| Segment RMBG（图像分割） |  |  |

## 二、nndeploy的核心能力：简单、高效、能落地

nndeploy是一款简单易用且高性能的AI部署框架。基于"可视化工作流+多端推理"，开发者能快速从算法仓库中，开发出指定平台与硬件所需的SDK，大幅节省开发时间。此外，框架已预置LLM、AIGC生成、换脸、目标检测、图像分割等众多AI模型，开箱即用。

### 1. 简单易用：拖拉拽搞定AI流程，不用写复杂代码

* **可视化工作流**：鼠标拖拽节点就能搭出完整AI流程，参数在界面上调整，效果实时预览，无需修改一行代码；
* **自定义节点无门槛**：算法团队用Python写预处理节点，部署团队用C++/CUDA写前后处理节点，不用懂前端技术，就能直接集成到可视化工作流中；
* **一键跨平台部署**：工作流导出为JSON文件后，通过Python/C++ API调用，可自动适配目标平台（比如导出到Android只需修改配置），无需重新编写业务逻辑；
* **灵活算法组合**：能自由组合不同算法，快速构建创新型AI应用。

### 2. 高性能：不只是"能跑"，更能"跑得快"

* **多并行模式支持**：覆盖串行、流水线并行（提升数据处理吞吐量）、任务并行（降低单次流程耗时）；
* **深度内存优化**：内置零拷贝、内存池、内存复用等优化策略，减少内存占用；
* **高效算子优化**：预置C++/CUDA/Ascend C/SIMD等优化实现的节点，提升计算效率；
* **13种推理框架适配**：一套代码可对接ONNXRuntime、TensorRT、MNN、OpenVINO、AscendCL等13种推理框架，底层框架选择由配置决定，无需修改业务代码。

### 3. 开箱即用：100+节点，覆盖高频AI场景

无需自行寻找模型、编写节点，nndeploy已预置100+常用节点，覆盖大语言模型、AIGC、检测分割等高频场景，拿过来就能用：

| 应用场景 | 可用模型 |
| --- | --- |
| **大语言模型** | **QWen-2** 、QWen-3 |
| **图片生成** | Stable Diffusion 1.5、Stable Diffusion XL、Stable Diffusion 3、HunyuanDiT 等模型 |
| **换脸** | **deep-live-cam** |
| **OCR** | **Paddle OCR** |
| **目标检测** | **YOLOv5、YOLOv6、YOLOv7、YOLOv8、YOLOv11、YOLOx** |
| **目标追踪** | FairMot |
| **图像分割** | RBMGv1.4、PPMatting、**Segment Anything** |
| **分类** | ResNet、MobileNet、EfficientNet、PPLcNet、GhostNet、ShuffleNet、SqueezeNet |
| **API 服务** | OPENAI、DeepSeek、Moonshot |

## 三、技术底层：为什么能做到"简单又高性能"？

**nndeploy 通过将 AI 算法拆分为"工作流节点"实现部署，采用"三层分离架构"设计**，整体架构如下图所示：

三层架构的详细说明如下表：

| 层级 | 核心组件 | 主要职责 |
| --- | --- | --- |
| 用户界面层 | 可视化编辑器（React）、命令行工具 | 拖拉拽搭建流程、实时调参、预览效果 |
| 后端服务层 | FastAPI、工作进程、任务队列、WebSocket | 管理工作流、处理异步任务 |
| 计算执行层 | 图执行引擎、多推理后端 | 高性能执行流程、适配不同硬件 |

实际使用中，推荐通过可视化工作流完成设计与调试，验证效果后，再用Python/C++ API在生产环境部署。无论前端界面操作还是API调用，最终都通过高性能C++计算引擎执行，确保开发与部署环境的一致性，实现"一次开发，处处运行"。

计算执行层是框架核心，由"图执行引擎"和"多端推理"两大组件构成：图执行引擎基于有向无环图（DAG）实现工作流调度，多端推理则统一封装十多种推理后端，提供跨平台推理能力。整体架构如下图所示：

### 图执行引擎

有向无环图执行引擎包含三层抽象：**Graph（图）作为工作流容器，负责节点管理、拓扑排序与执行调度**；Node（节点）作为独立的计算单元；Edge（边）作为节点间的数据传输通道。值得注意的是，Graph 继承自 Node，这意味着"图本身也是一个节点"，可作为子图嵌入到更大的 Graph中，实现任意层级的嵌套结构。通过这种设计，用户能像搭积木一样组合各类算法节点，构建复杂AI工作流。

图执行引擎支持以下执行模式：

* **串行执行**：按工作流拓扑排序后的顺序，逐个执行节点；
* **流水线并行**：像工厂流水线一样，前处理、推理、后处理同时进行，**提升数据处理吞吐量**；
* **任务并行**：多个无数据依赖的节点同时运行，**缩短单次算法全流程耗时**；
* **组合并行**：支持嵌套工作流，可灵活组合各类并行模式，充分发挥硬件性能。

### 多端推理

nndeploy 的多端推理模块，为不同推理后端提供了统一的模型推理接口，让开发者无需关注底层推理框架的差异。以下以"新增 MNN 推理后端"为例，讲解其实现原理：

1. 深入理解 MNN 的 API 设计与推理流程；
2. 掌握 nndeploy 的"推理超参数配置类"与"推理基类"设计逻辑；
3. 编写推理适配器：继承推理基类实现特定推理后端适配器、继承推理超参数配置类实现参数类、编写数据结构转换工具类；
4. 基于新后端运行模型，验证功能完整性。

由于不同推理框架采用各自的数据交互方式（如 TensorRT 的`io_binding`绑定机制、OpenVINO 的 `ov::Tensor` 数据容器、TNN 的 `TNN::Blob`数据结构），nndeploy 设计了统一的数据容器 `Tensor` 和 `Buffer`，屏蔽这些底层差异。

同时，框架提供"异构设备抽象层"，通过对硬件设备的抽象，屏蔽不同硬件编程模型的差异。目前已支持CPU（X86、ARM架构）、GPU（CUDA）、NPU（如华为昇腾AscendCL）等设备，让开发者能用相同代码在不同硬件平台上运行，实现"一次编写，多端推理部署"。

此外，框架内部还开发了"推理子模块"作为缺省推理框架（与TensorRT\MNN等推理框架同等地位，但是没有人力做充分的性能优化，故不推荐用户在生产环境下使用），当用户环境未编译链接其他推理框架时可使用。模型支持方面，已适配图像分类（ResNet50）、目标检测（YOLOv11）、图像分割（RMBG1.4）等，目前正拓展大语言模型（LLM）推理支持。

## 四、3 步上手：从安装到跑通AI流程

## 如何使用 nndeploy？

1，**安装**

```
pip install --upgrade nndeploy
```

2， **启动可视化界面**

```
nndeploy-app --port 8000
```

启动成功后，打开 http://localhost:8000 即可访问工作流界面

3，**导出工作流并命令行执行**

完成工作流搭建后，保存为 JSON 文件并通过命令行执行：

```
# Python CLI
nndeploy-run-json --json_file path/to/workflow.json
# C++ CLI
nndeploy_demo_run_json --json_file path/to/workflow.json
```

4，**导出工作流并 API 加载运行**

在可视化界面中完成工作流搭建后，可保存为 JSON 文件，然后通过 Python/C++ API 加载执行。

如 Python API 加载运行 LLM 工作流代码：

```
graph = nndeploy.dag.Graph("")
graph.remove_in_out_node()
graph.load_file("path/to/llm_workflow.json")
graph.init()
input = graph.get_input(0)
text = nndeploy.tokenizer.TokenizerText()
text.texts_ = [ "<|im_start|>user\nPlease introduce NBA superstar Michael Jordan<|im_end|>\n<|im_start|>assistant\n" ]
input.set(text)
status = graph.run()
output = graph.get_output(0)
result = output.get_graph_output()
graph.deinit()
```

如 C++ API 加载运行 LLM 工作流代码：

```
std::shared_ptr<dag::Graph> graph = std::make_shared<dag::Graph>("");
base::Status status = graph->loadFile("path/to/llm_workflow.json");
graph->removeInOutNode();
status = graph->init();
dag::Edge* input = graph->getInput(0);
tokenizer::TokenizerText* text = new tokenizer::TokenizerText();
text->texts_ = {
    "<|im_start|>user\nPlease introduce NBA superstar Michael Jordan<|im_end|>\n<|im_start|>assistant\n"};
input->set(text, false);
status = graph->run();
dag::Edge* output = graph->getOutput(0);
tokenizer::TokenizerText* result =
    output->getGraphOutput<tokenizer::TokenizerText>();
status = graph->deinit()
```

## 五、从“一个人的想法”到“一群人的坚持”

2022 年底的一个深夜，A神（项目发起人-Always）对着电脑屏幕发呆。作为一名在 AI 推理框架与算法部署领域深耕多年的程序员，刚完成第 N 个“从算法到上线”的项目。经过几十个部署项目的历练，他发现所有 AI落地项目，其实都在重复同一套流程——只是模型和业务逻辑不同而已。

“能不能把这套流程抽象成通用框架？让程序员只需专注核心逻辑，其他工作交给框架自动完成？”

**“拖拉拽工作流 + 有向无环图 + 多端推理”**——这个技术方案在我脑海里逐渐清晰。为了实现这个想法，我做了一个在很多人看来不理智的决定：降薪加入一家 955 公司，给自己更多业余时间投入项目开发。接下来的大半年里，几乎所有业余时间都用在了这个项目上。

2023年8月28日晚上11点，A神把 nndeploy 的第一个版本推到了 GitHub。那一刻他心情心情很复杂：有兴奋，有紧张，也有忐忑——这个项目真的能帮到其他开发者吗？会有人理解我想解决的问题吗？

没想到当天就收到了 CGraph 作者 Chunel 的私信。他不仅认可项目思路，还在自己的技术社区推广，让项目快速突破了 100 个Star。

### 小伙伴们的加入

随着项目被更多人发现，一些技术开发者开始主动了解 nndeploy。每当有人想深入参与，我都会花时间和他们聊项目背景、技术细节，帮他们解决入门时的问题。

渐渐地，一批认同项目理念的开发者留了下来，成为核心成员：csrdxka、youxiudeshouyeren、zhangzhaosen、acsars520、JoDio-zd、Arpan、Leonisux、Realtyxxx、raymond、02200059Z……我们一起攻克了一个又一个技术难题。

### 解决技术难题

项目发展中，最让我印象深刻的是“解决 DAG 图表达能力瓶颈”的经历。当时我们遇到一个问题：现有图的表达能力不够强，用户用起来不够方便。

Always 和 02200059Z讨论了很多次，都没找到完美方案。于是，他们约了 youxiudeshouyeren、Chunel、JoDio-zd 几位开发者，在一个周末开线上会议专门讨论。

会议刚开始，想新增“子图”概念，但 Chunel 基于他在CGraph项目的经验提醒：这样做需要引入 Group、Cluster、Region 等多个概念，会增加用户理解成本，开发工作量也会大幅增加。

讨论陷入僵局时，有人突然提出：“能不能让图继承节点？这样图本身就能当节点，嵌入到另一个图里。”

这个想法让所有人眼前一亮！深入分析后发现，这个方案有三个明显优势：代码实现简单、表达能力强、能灵活实现“图中嵌图”。

确定方案后，大家分工协作：Chunel 负责线程池模块，JoDio-zd 和我一起开发流水线并行功能，youxiudeshouyeren 承担任务并行的开发。

这次协作让我深刻体会到团队协作的力量——每个人的专业背景和经验，都为项目贡献了不可替代的价值。最终我们不仅解决了技术难题，还为nndeploy奠定了坚实的架构基础。

随着项目发展，这样的故事还有很多：和 Arpan、csrdxka、youxiudeshouyeren 等跨时区讨论LLM部署方案；和csrdxka、zhangzhaosen一起打磨工作流架构……

现在，csrdxka 和 youxiudeshouyeren 已经从开发者成长为项目维护者——他们不仅在技术上贡献突出，还在社区建设中发挥了重要作用。他们也期待有更多开发者加入 nndeploy，感兴趣的可以给项目提 issue 和 PR.。

### 企业落地实践

随着项目成熟，nndeploy 已在多个行业的生产环境中落地：

* 文档处理公司用它搭建 OCR 完整流水线；
* 智能汽车企业在座舱项目中部署 NLP 算法；
* AI Box 公司利用它的多端推理能力做算法部署；
* 算法研发团队通过可视化工作流，缩短了“模型到部署测试”的周期。

这些实际应用不仅验证了nndeploy的价值，用户反馈更直接推动了项目迭代——让框架在实用性和稳定性上不断完善，能为更多开发者提供更好的AI落地解决方案。

## 六、写在最后

我与 Always（nndeploy 发起人）的相识源于 CGRAPH 技术群。作为同样关注 AI 基础设施的开发者，我们在群里交流量化技术的细节，慢慢熟悉起来。后来我们经常通过电话分享对这个行业的一些观察和想法。

记得有个周末的上午，我们聊到 AI 推理技术的发展趋势。我**们都觉得，Python + PyTorch 的组合可能会是一个重要的方向**。那次交流中，Always 分享了他对 nndeploy 的一个技术想法：希望能做成 Python 与 C++ 的混合框架，既能用上 Python + PyTorch 生态的丰富资源和开发便利性，又能在需要高性能的地方发挥 C++ 的优势。

现在回头看 nndeploy 的发展，这个想法确实在逐步实现。C++ 核心功能已经在 Python 侧完成了封装，并且支持了基于 Python 的 DiT（Diffusion Transformer）等新模型的部署。这样的架构设计，为 AI 算法的部署提供了完整的解决方案。

写到这里，我非常感慨，A 神完成了我非常认可又想去做的事情，作为一个AI 方向的程序员，我们都想在开源界留下点有趣、有用、有意义的项目，我们都有着纯粹的技术热爱，但是我的专注和执行力还需要提高。而 A 神有想法有执行力，所以做出了有用有意义的项目-nndeploy，而且也在工作、兴趣和生活上找到了平衡。

最后，希望大家能从这个项目从学到东西、并帮助大家落地 AI 项目、同时认识到一群有意思、开放包容、真诚的技术人。

> **GitHub 地址**：https://github.com/nndeploy/nndeploy
