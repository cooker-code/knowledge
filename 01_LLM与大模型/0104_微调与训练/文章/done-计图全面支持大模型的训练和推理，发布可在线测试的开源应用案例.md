---
title: 计图全面支持大模型的训练和推理，发布可在线测试的开源应用案例
author: 图形学与人工智能
date:
url: http://mp.weixin.qq.com/s?__biz=MzA3OTE4OTkxMw==&mid=2247487644&idx=1&sn=9ad6e116d658b6b587286564a4c435b0&chksm=9fb6126aa8c19b7cab9ed929ac5edc42051c5f4e9988cd08121582e12e0c656186d23fefd320&mpshare=1&scene=24&srcid=0613OAETnlulx0uSCG9Xbu5V&sharer_sharetime=1686659335478&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---
> 已吸收至：[[01_LLM与大模型/0104_微调与训练/0104_核心知识点/训练基础与模型压缩来源校准|训练基础与模型压缩来源校准]]

随着ChatGPT的推出，大模型正在快速地发展，国内同样涌现出一大批优秀的大模型研究。大模型的推理和训练需要深度学习框架的支持。计图(Jittor)深度学习框架基于元算子融合的创新技术，在大模型的推理和训练上体现出巨大的技术优势。

4月3日，非十科技与清华大学可视媒体研究中心合作，发布了大模型推理库JittorLLMs，大幅降低了硬件配置要求，支持了国内外优秀的大模型，帮助用户轻松部署大模型；复旦MOSS团队、清华&智谱华章GLM团队均表示：JittorLLMs推理库，可以帮助到了更多的用户部署大模型，降低了硬件配置要求。

Jittor团队进一步推出的大模型支持，目标在于帮助更多的用户实现大模型训练、推理和部署。就在今天，非十科技提供全流程AI大模型服务，支持模型训练、微调、推理，并提供从算力、算法、性能调优一站式解决方案，让人人皆可无忧训练大模型。

**Jittor支持大模型训练与部署**

此次Jittor团队发布几项新研发的技术，在分布式训练、分布式微调、模型量化与调优等方面，为大模型训练和部署提供更友好的支持！

1. JittorLLM-Train：基于该分布式训练库，可进行大模型训练，最大可支持万亿级别大模型参数训练。
2. JittorLLM-Tune：基于该分布式微调库，可支持多种主流多模态图文大模型的微调，如Transformer based模型，如OpenLLaMA、Falcon、Alpaca、vicuna等等，支持Conv based模型，支持扩散类模型，支持LoRA等主流微调方法。
3. JittorLLM-Deploy：基于该推理库，可轻松实现模型量化、模型加速、模型调优等功能，大模型推理速度相比原生模型，性能提升先比国际主流框架最大可达8倍，让用户用最低的成本和最少的算力，完成模型部署。
4. OpenLLaMA-Chinese：这是一个由非十科技实现的基于计图的全流程开源大模型示范案例，包括模型训练、微调、部署等，用户可参考案例，快速实现自己的业务场景。该案例旨在给用户提供一个训练、微调以及部署的指南。

   *注: 本开源案例的基础模型来自Berkeley AI Research提出的OpenLLaMA、英文数据来自Stanford提出的Alpaca项目、中文数据来自哈工大&科大讯飞Chinese-LLaMA-Alpaca项目，特别感谢以上单位，以及LLama-X、transformers等开源项目。*

下面，介绍Jittor在大模型支持方面的几个特点。

## **精度完全对齐**

模型精度、训练曲线与国际主流解决方案完全对齐， 如图1所示，JittorLLM-Train在训练OpenLLaMA-Chinese应用案例中，与DeepSpeed+PyTorch的精度保持完全一致。

## 图1 精度对齐的曲线

## **降低训练成本、增大模型规模**

应用计图原生支持的元算子融合机制，可以在大模型训练上提升20%的性能。大模型在训练过程中，常常碰到参数过大，显存不够，无法训练。

Jittor框架还通过高效的动态显存卸载与磁盘交换技术，减少显存压力，大大提升模型训练规模，最大可支持万亿级别的模型训练。

*图2 Jittor框架大模型训练内存优化机制*

## **可自由商用的、可完整复现的中文大模型应用案例**

目前，LLaMA是目前研究人员使用最多的大模型，然而LLaMA存在几个问题 ：

1. 数据不开源：因为数据不开源，LLaMA无法完整复现，生成内容可能存在不可控的问题。
2. 参数不完全开放：LLaMA的参数只能通过申请获得，无法商用。

OpenLLaMA是 Berkeley AI Research的博士生Hao Liu发起的一个开源LLaMA复刻项目，目的是从头开始训练一个类似LLaMA模型，使用的模型架构、context长度、训练步骤、学习速率等，完全按照原始的LLaMA论文设置。唯一的区别是OpenLLaMA使用RedPajama数据进行训练。使用Google的TPU-v4s。

然而，OpenLLaMA和LLaMA一样，仍然不具备良好的中文对话能力；计图团队在OpenLLaMA的基础之上，进行了50k中文指令微调，增强中文对话能力。该案例完全开源可复现，目的在于为计图用户，提供一个基于计图的训练、推理、部署全流程开源示范案例。

下面的视频展示了，通过计图微调OpenLLaMA，逐渐学会清华校歌的过程：

*图3 Jittor训练微调的演示*

## **支持国产芯片DCU推理盘古大模型**

下面的视频展示了如何在国产芯片曙光DCU上，基于计图推理盘古大模型的视频演示：

图4 基于计图在DCU上的盘古大模型推理演示

## **总结**

计图此次全面支持大模型相关的推理和训练任务，包含JittorLLM-Train训练库、JittorLLM-Tune微调库、JittorLLM-Deploy推理库，以及一个完全开源的OpenLLaMA-Chinese应用案例，同时，也实现了对国产芯片的支持。

Jittor团队希望和国内科研院所和企业合作，支持基于国产软硬件的大模型技术的演进和应用的深入，共同推动我国人工智能生态的发展。

**致谢**

感谢非十科技对Jittor框架的代码开源、以及为计图框架支持大模型推理和训练做出的贡献。非十科技作为人工智能服务的企业，致力于加速人工智能算法从硬件到软件全流程的落地应用、提供各类计算加速硬件的适配、定制深度学习框架以及大模型推理和训练等方面的服务。非十大模型服务的官网：

https://llm.fittentech.com/

**GGC往期回顾**

1. [Scopus发布2022年度影响因子, CVMJ从5.9升至11.1, 排名8/102](http://mp.weixin.qq.com/s?__biz=MzA3OTE4OTkxMw==&mid=2247487597&idx=1&sn=af1a394c0ddf461ad7f884c24322b2b7&chksm=9fb6129ba8c19b8d45f8425a4e7413edfd26649f697f1b4560d8dec3e24350580a36cb8c94eb&scene=21#wechat_redirect)

2. [Computational Visual Media第9卷第3期导读](http://mp.weixin.qq.com/s?__biz=MzA3OTE4OTkxMw==&mid=2247487549&idx=1&sn=c56a1a60df92c94feceaf95e058b5f1f&chksm=9fb612cba8c19bddb1252de2e596c3fc09aa7277958e581391df8591cf9761af8ae7cd89a168&scene=21#wechat_redirect)

3. [第三届“计图”人工智能算法挑战赛启动](http://mp.weixin.qq.com/s?__biz=MzA3OTE4OTkxMw==&mid=2247487474&idx=1&sn=e2371b8f79bfbf7cf85a9ac38f22599c&chksm=9fb60d04a8c18412a43d8dfd0c3fae0aee4a069a1137815c8109c815a2895869d49cce0a3e92&scene=21#wechat_redirect)

4. [计图助力AIGC｜非十科技发布首款可编辑的3D内容生成APP -Fitten3D](http://mp.weixin.qq.com/s?__biz=MzA3OTE4OTkxMw==&mid=2247487461&idx=1&sn=0c40df0b4ccc229c045f6fe52bceca3d&chksm=9fb60d13a8c18405a29324a01092f8ebf063c796059bfe8a945f9e9618d2b07fc762eaed392a&scene=21#wechat_redirect)

5[.](http://mp.weixin.qq.com/s?__biz=MzA3OTE4OTkxMw==&mid=2247485917&idx=1&sn=2c919384eb1e9b8e9c092844b010724d&chksm=9fb60b2ba8c1823d72a07024058ddb5fb646d3bddc14e51389ec0785866a1926dfedf1a39d03&scene=21#wechat_redirect)[计图团队与头歌平台合作发布“计图深度学习框架实践课程”](http://mp.weixin.qq.com/s?__biz=MzA3OTE4OTkxMw==&mid=2247487443&idx=1&sn=9a525ee56ae8745cbc089ba18c80fd01&chksm=9fb60d25a8c184338ede8495f94b5f5d44405873fdd68a839c587ccd0addebc7b3d75bab3deb&scene=21#wechat_redirect)

您可通过下方二维码，关注清华大学计算机系图形学实验室，了解计算机图形学、Jittor框架、CVMJ期刊及会议的相关资讯。
