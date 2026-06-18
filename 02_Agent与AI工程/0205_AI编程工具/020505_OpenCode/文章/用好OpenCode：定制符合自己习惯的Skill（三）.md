---
title: 用好OpenCode：定制符合自己习惯的Skill（三）
author: 码力欧实验室
date: 
url: https://mp.weixin.qq.com/s?chksm=fc8b4e7bcbfcc76d2609b95e80b028f1aa3520a54f5089853b30457d8593c4d44c6cafd9175e&exptype=unsubscribed_card_recommend_article_u2i_mainprocess_coarse_sort_tlfeeds&ranksessionid=1770031933_3&req_id=1769995221964840&mid=2247484215&sn=021738f11373c3f3880a2808d3f80a7d&idx=1&__biz=MzU2ODY5NTg0Mw%3D%3D&scene=169&subscene=200&sessionid=1770032020&flutter_pos=29&clicktime=1770032057824&enterid=1770032057824&finder_biz_enter_id=5&key=daf9bdc5abc4e8d045763914fdbbb92dc400268a5b4ab5c2ebc66b0e2bddbf312ea9574c0ee6577a72733854a4c0a4ee17f2897f0e43d3d6777689d8b45dae3c83ee6924819c838c93d820e1cd6442fa835e18c8f5defc088b8eb3cd9df1514876ec72b1524c2fd16c196fb377ad28e0cfc8bd6bb9476b7448b4c7d3946f9cdb&ascene=0&uin=MjAwNTI2NjEyMw%3D%3D&devicetype=OHOS-22&version=f3800e2a&abtest_cookie=AAACAA%3D%3D&lang=zh_CN&countrycode=CN&exportkey=n_ChQIAhIQDplFOqqmL3nahKZ8HtjUyhLfAQIE97dBBAEAAAAAAAjBLYguavUAAAAOpnltbLcz9gKNyK89dVj0MyRq1lvcoEd51VivoLC89%2BuUvebu6jPTneNtSVBt89UzDNOO6zXI0b2JOGxwbR7PVzkIkGJ4Ct%2FlM1h60Nb%2BN2H0%2FzEg5a5exItOzdV7uGNUV3cTyXUH2D2hT2%2B7rGmIVeyKI9z9IozAdOoNJiqW%2F%2BvBrZ7QAEPh%2Fr9xyVkwU3vup6yP%2F%2Bl2cl%2Bxx6u%2BMC5twnNjva1tAVurQt6Ny%2FF5xbkeiJpC5il7MRNVFaLyP%2BDHuJbNRWiCVIM%3D&pass_ticket=nDDS2JbuiyOXb%2F4UB%2FB17ygiChXwJJ0fjOQBSLFeKTwhEYL9Bz6YLBT1ILs5jM5P&wx_header=3
---

最近每天都在用 **OpenCode**，这东西是个AI CLI编程工具，类似 Cursor。上一篇介绍了superpowers[用好OpenCode：深入superpowers（二）](https://mp.weixin.qq.com/s?__biz=MzU2ODY5NTg0Mw==&mid=2247484209&idx=1&sn=a9f5af50db4d31a0280a4e5ee0c71d64&scene=21#wechat_redirect)，本篇继续深入。

OpenCode必选的SKILL开源库 **superpowers**，里面有个叫 `executing-plans` 的技能，用来帮AI按计划执行任务。老实说，原版能用，但在我看来，它有点缺心眼了。

比如它不按规矩写代码，不管测试效率，也不管业务逻辑到底通不通。为了能让自己用着顺手，我决定改造它，做一个符合我开发习惯的 Skill。

---

## 一、 为什么要改？

我看了一下 Github 上的 `superpowers` 仓库，原版的 `executing-plans` 有几个让我很难受的问题：

1. 1. **不守规矩**：我项目里有 `AGENTS.md` 专门定开发规范（日志怎么打、错误怎么处理），它经常视而不见，自己想怎么写就怎么写。
2. 2. **浪费时间**：制定个小计划，它都要每个任务都跑一遍全量测试。开发过程中频繁跑测试太慢了，我觉得应该按需跑或者最后统一跑。
3. 3. **没大局观**：它只盯着单元测试通过。有时候测试全绿，但几个功能串起来一跑，主流程是断的。
4. 4. **验收草率**：写完代码就完事了，不回头看看“代码”和“计划文档”、“设计文档”是不是一回事。
5. 5. **算力浪费**：现在的AI大多支持多线程，明明可以多开几个 Subagent 并行写代码，它偏偏要一个接一个地来，干看着着急。

我需要的不是一个只会执行命令的脚本，而是一个懂业务、守纪律、还能高效利用算力的“搭档”。

---

## 二、 我的改造思路

为了解决上面的问题，我基于 `superpowers:writing-skills` 创建了一个新技能，叫 `executing-plans-with-standards`。

我的核心思路就三点：

1. 1. **业务主流程优先**：别纠结细节，先保证 API -> 校验 -> 逻辑 -> 数据库 -> 响应 这条路是通的。
2. 2. **三方对齐**：写完代码后，必须评估 **代码实现、计划文档、设计文档** 这三者是不是对得上，防止写着写着需求就变了。
3. 3. **灵活的质量门禁**：

* • 写代码时：只检查语法和类型，不强制跑测试。
* • 写完后：统一跑全量测试。
* • 特殊情况：如果计划里明确写了“这个任务要带测试”，那就在任务级跑。

---

## 三、 Skill 具体怎么做的

为了不把一个文件写得太臃肿，我把这个技能拆成了三块，就像一个开发小组一样分工。

### 1. 主 Skill：`executing-plans-with-standards`

这是组长，负责指挥。流程简化为 7 步：

1. 1. **看计划**：这是第一步。如果计划里没有“更新设计文档”和“清理废弃代码”这两项，**强制给它加上**。这是为了防止代码写了一堆，文档没更新，垃圾代码也没删。
2. 2. **拆任务**：看看哪些活可以一起干，哪些必须等前面的干完。
3. 3. **并行干活**：识别独立的任务，同时启动几个 Subagent 一起写代码。这就是充分利用 AI 算力。
4. 4. **汇报进度**：干完一批，说一下情况。
5. 5. **评估对齐度**：调用专门的评估技能，看看代码有没有写偏。
6. 6. **统一测试**：所有代码写完了，一次性把测试跑完，确保集成没问题。
7. 7. **收尾**：调用收尾技能。

### 2. 辅助 Skill A：`triparty-alignment-assessment`

这个负责“质检”。我用了一个**清单**的形式，让 AI 逐项核对：

* • **业务主流程完整性**：核心链路通了吗？
* • **计划-代码对齐**：计划里要的功能都写了吗？
* • **设计-代码对齐**：代码符合架构设计吗？
* • **标准合规性**：日志、错误处理、响应格式符合 `AGENTS.md` 吗？  
  如果发现没对齐，主 Skill 会自动生成修复任务，直到检查通过。

### 3. 辅助 Skill B：`finish-implementation`

负责收尾。原版的总想着推送到远程仓库或者创建 PR。但我个人的习惯是：**本地先稳住，不轻易推，也不轻易删分支**。  
所以我改了它的行为：

* • 默认保留分支（防止万一要回滚）。
* • 只做本地合并，不主动推送。
* • 选项精简为：1. 本地合并并删分支，2. 保留分支（推荐）。

---

## 四、 怎么装上用？

写好了文件，还得告诉 OpenCode 去哪里找它们。  
在 OpenCode 里，Skill 和 Command 是分开放的。你需要把文件移动到对应的目录。  
假设你的配置目录在 `~/.config/opencode/`，操作如下：

1. 1. **放 Skill 文件**：  
   把写好的三个 Skill 目录放到：  
   `~/.config/opencode/skills/`  
   目录结构长这样：

   ```
   1

   2

   3

   4

   5

   6

     

   ~/.config/opencode/  
   ├── skills/  
   │   ├── executing-plans-with-standards/  
   │   ├── finish-implementation/  
   │   ├── triparty-alignment-assessment/  
   │   └── ...
   ```
2. 2. **配置命令**：  
   为了方便调用，我们在 `~/.config/opencode/command/` 下建个文件叫 `execute-plan-with-standards.md`：

   ```
   1

   2

   3

   4

   5

     

   ---  
   description: 按标准执行计划（支持并行与三方对齐评估）  
   disable-model-invocation: true  
   ---  
   Invoke the executing-plans-with-standards skill...
   ```

弄完这两步，重启 OpenCode。以后你在命令面板里输入 `execute-plan-with-standards`，或者用 `/` 命令，就能直接用它了。

---

## 提示词

别看着上面的内容这么多，其实这个SKILL完全是AI帮我完成的。只要你的指令给得准，AI 完全能理解什么叫“工程化”。下面我把这次用到的关键 Prompt 拆解出来，你可以直接拿去复用。

第一轮、第三轮中使用到的writing-skills、brainstorming SKILL都是superpowers自带的，可以直接使用。

**第一轮**：

```
1

2

3

4

5

  

使用 superpowers:writing-skills 创建一个执行计划的 skill，要求如下：  
   
参照 superpowers:executing-plans 的设计  
执行编码任务时，严格遵循 @AGENTS.md 中的指导原则、日志、错误级别、错误码、统一响应、测试用例等规范  
执行完成后，要评估代码实现与计划文档的对齐度
```

**第二轮**：

```
1

2

3

  

- 我希望更多关注“业务主流程”完整性  
- 评估代码实现、计划文档、设计文档三者的对齐程度  
- 任务执行时，如果计划文档中未要求明确要求该任务要执行测试用例，则该任务的测试用例执行只在计划完成时统一执行
```

**第三轮**：

脑暴一下这个SKILL，看还有什么没有考虑到，需要调整的。

```
1

  

/brainstorming execute-plan-with-standards.md
```

**第四轮**：

```
1

2

  

- 将三方对齐，拆成多个skill，在executing-plans-with-standards 使用  
- superpowers:finishing-a-development-branch，参照一下这个skill，生成一个自定义的skill，在executing-plans-with-standards中使用
```

**第五轮**:

将这三个SKILL调整为opencode全局SKILL

## 六、 总结

这就是我改造 Skill 的过程。原版的工具虽然能用，但它不懂我的工程习惯。工具始终是工具，让它真正好用的，是你对技术的理解。希望这篇教程能帮你把AI工具改成自己趁手的兵器。