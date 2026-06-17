---
title: 实测 4 个爆火 Skill，一句话生成画布/知识库/任务规划/自动发布
author: 彭少
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg3NzU2NjY3OQ==&mid=2247489787&idx=1&sn=dcb1baa3a3af539a26e2730ef776c161&chksm=cedc70a6fc6292fd4b248ff10c5819ab89fa30b2be9c242c641d0cbe858548ce0e3ccdebcfa1&mpshare=1&scene=24&srcid=0123iVw201ZHPuXKudhB2qD2&sharer_shareinfo=9234561edf9a6ea408e500197f2d3982&sharer_shareinfo_first=9234561edf9a6ea408e500197f2d3982#rd
---

**点击上方卡片关注我**

**设置星标 学习更多AI出海知识**

测试了几个最近比较火的skill，分享给大家。之前已经写过几篇 skill 相关的内容，可以看看。

[AI 提效指南：用 Claude Code Skills 自动化内容发布，一个指令同步多平台](https://mp.weixin.qq.com/s?__biz=Mzg3NzU2NjY3OQ==&mid=2247489392&idx=1&sn=e917bb52df2f7bd2ff462b49ae8be5fc&scene=21#wechat_redirect)

[出海建站必备：告别AI味，这两个页面设计 Skills 太牛了！](https://mp.weixin.qq.com/s?__biz=Mzg3NzU2NjY3OQ==&mid=2247489451&idx=1&sn=0447e337cef1e12f577b2fed72048cad&scene=21#wechat_redirect)

## 一句话生成知识画布

这个仓库里其实有3个skill，都是让 Claude Code 更好地处理 Obsidian 文件。

安装方式：给AI说一句 帮我在用户级别下安装这个skill  https://github.com/kepano/obsidian-skills

第一个 json-canvas 可以生成画布/思维导图/流程图

比如用他给的示例

> “
>
> /json-canvas 画一个思维导图，主题是：如何提升工作效率

生成的文件格式是  .canvas ，在Obsidian打开就是下面的效果。

> “
>
> /json-canvas 画一个思维导图，先搜索下程度从商周到明清的历史变迁，重大历史事件，以及重点介绍下三星堆

第二个 obsidian-bases，生成数据库视图（表格、卡片、看板）

让 AI 直接给我生成一个使用示例了。

效果就是这样的，没有仔细去测，感兴趣的可以研究下。

## 一句话把本地的 markdown 文档自动发推

推特现在只要是 Premium 会员就可以发长文，就像公众号这样，放开了之前 Premium+ 会员的限制，对于创作者非常利好了，所以平常输出的内容可以顺便同步到X。

但是X的编辑器不够丝滑，从别的地方粘贴过去的内容，会有一些问题，比如图片不显示，这就很麻烦，尤其是对于教程类型的文章来讲，通常一篇有十几张图，挨个选择上传太麻烦了。

先是看到了@wshuyi 王老师写的如何用 Claude Skill 将 Markdown 图文无痛发布 X Premium 文章。

```
https://github.com/wshuyi/x-article-publisher-skill
```

试了下 X-article-publisher-skill，当时遇到一个卡点图片处理，我们图片都是上传到了图床的，但是图片链接在编辑器里面无法展示，当时cc的解法是先下载到本地，然后再挨个上传，图片多的情况下操作很慢，也非常消耗token，后续就没有用起来。

前几天又看到了 @vista8 向阳乔木更新了一版，图片上传改成占位替换，插入更精准，

```
 https://github.com/joeseesun/qiaomu-x-article-publisher?tab=readme-ov-file
```

然后根据实际执行结果，我让CC接着优化了一下，现在比较丝滑了。

X Article Publisher 优化总结：

1. 图片占位符系统
2. 远程图片自动下载，自动检测 http/https 并下载到 /tmp/article\_images/
3. 扩展 Markdown 语法
4. 先删后贴策略

录了一个视频，可以看下现在的效果，基本达到我理想中的效果了。

## 多步骤任务处理

planning-with-files 这几天传播和讨论比较多。

核心理念参考 Manus Agent 的 Context Engineering，这是多步骤任务的救星，先规划 Todo List、check 进度，解决AI 执行多步骤任务时"跑偏"或"遗忘"的问题。

安装很简单，在Claude Code里直接说

> “
>
> 安装这个Claude Skill：https://github.com/OthmanAdi/planning-with-files

Claude 默认有个 TodoWrite 工具，但那是在脑子里记，上下文一长就容易忘。这个 skill 改成写在纸上，用 markdown 文件当外部记忆。

* task\_plan.md — 记录目标、阶段、进度
* notes.md — 存研究发现、中间结果
* deliverable.md — 最终交付物

触发方式可以是手动触发

**/planning-with-files  然后描述你的任务。**

也可以通过关键词自动触发，比如当你提到下面这些关键词，Claude 应该会自动采用这个模式：

"帮我规划一下..." ，"这是个多步骤任务..." ，"我需要做个研究..." ，"帮我组织一下这个工作..."

再或者直接一点，告诉它**用 planning-with-files 的方式，帮我完成 xxx**

Claude 自带的 plan 模式我觉得也挺好的，shift+tab 键连按两次可以触发，而且也存了文档，如果任务中断了，可以试试找到计划文件，继续执行。

## 命令行操作 NotebookLM

之前写过一篇关于 NotebookLM skill的文章，[出海效率提升：一个 Skill 让 Claude Code 接入你的 NotebookLM 知识库](https://mp.weixin.qq.com/s?__biz=Mzg3NzU2NjY3OQ==&mid=2247489410&idx=1&sn=25390213ebfb7f8e8ee741ba78bf0937&scene=21#wechat_redirect)

我发现这个好像要更高级，可操作性更强。

安装方式，和Claude Code 说：我要安装这个skill，https://github.com/teng-lin/notebooklm-py

注意，安装之后需要打开一个新的终端执行 `notebooklm login` 命令登录，登录成功之后记得在终端里面按回车键，这样才能记住你的登录信息。回到cc的命令行下达指令。

> “
>
> 我想创建一个Claude code 的笔记本，添加源  Claude code 的官方文档，中文版

就是这么丝滑，除了基础的提问，还可以生产播客，生成视频，基本上网页端的核心功能都有了，真的是做到了把 NotebookLM 搬到了命令行。

关于skill的玩法真的太多了，大家一定要用起来，市面上比较火的可以多试试，和自己的使用场景匹配上可以大大提高效率。也可以根据自己的工作流，借助AI来创建skill，总之可玩性很高，空间很大。

欢迎关注，这个账号还会持续分享更多出海工具、实战经验、踩坑记录。

[推荐一个牛逼的中转工具！](https://mp.weixin.qq.com/s?__biz=Mzg3NzU2NjY3OQ==&mid=2247488444&idx=1&sn=64d065f88b535223743cadf14fe447bf&scene=21#wechat_redirect)

---

扫码或微信搜索 257735 添加微信，回复【出海资料】即可免费领取《AI 编程出海资料》，一次读懂普通人也能启动的出海路径。

如果你想进一步系统学习、实操出海项目，👉 点击了解详情：[推荐我的AI编程出海训练营！](https://mp.weixin.qq.com/s?__biz=Mzg3NzU2NjY3OQ==&mid=2247489358&idx=1&sn=a9b44717101426cd166521ef1cfef2ec&scene=21#wechat_redirect)

**[炸裂！Anthropic 又杀疯了！](https://mp.weixin.qq.com/s?__biz=Mzg3NzU2NjY3OQ==&mid=2247489644&idx=1&sn=2c5abdca04f8fec6e05eda9eef4afda7&scene=21#wechat_redirect)**

**[从海外公司注册到 Stripe 收款，跑通了出海收付款全流程（实操分享）](https://mp.weixin.qq.com/s?__biz=Mzg3NzU2NjY3OQ==&mid=2247489551&idx=1&sn=08058b274add835f37b3374fa43b6757&scene=21#wechat_redirect)**

**[出海建站必备：告别AI味，这两个页面设计 Skills 太牛了！](https://mp.weixin.qq.com/s?__biz=Mzg3NzU2NjY3OQ==&mid=2247489451&idx=1&sn=0447e337cef1e12f577b2fed72048cad&scene=21#wechat_redirect)**

**[玩转 Claude Code Hooks：让自动化渗透到每个环节](https://mp.weixin.qq.com/s?__biz=Mzg3NzU2NjY3OQ==&mid=2247489503&idx=1&sn=b81277c62e501d497b7f621e3a726b34&scene=21#wechat_redirect)**

**[出海工具全集：覆盖 9 大类别，收藏这一篇就够了](https://mp.weixin.qq.com/s?__biz=Mzg3NzU2NjY3OQ==&mid=2247488151&idx=1&sn=b94bea4e18d162a0c6e8d542a6941555&scene=21#wechat_redirect)**

**[出海网站实战：Stripe 支付接入，从 0 到收款全流程](https://mp.weixin.qq.com/s?__biz=Mzg3NzU2NjY3OQ==&mid=2247488496&idx=1&sn=ea69777b985c23576fe59777ab1f0da9&scene=21#wechat_redirect)**

**[出海必备：一天搞定5张港卡，我的香港卡办理全攻略](https://mp.weixin.qq.com/s?__biz=Mzg3NzU2NjY3OQ==&mid=2247488697&idx=1&sn=1d04e1a522668308397a823579a68303&scene=21#wechat_redirect)**

**[彭涛：从百万自媒体到 AI 编程出海，我的 2025 破局与重生](https://mp.weixin.qq.com/s?__biz=Mzg3NzU2NjY3OQ==&mid=2247489360&idx=1&sn=ef5a265b751b537df43b60d4e32130aa&scene=21#wechat_redirect)**