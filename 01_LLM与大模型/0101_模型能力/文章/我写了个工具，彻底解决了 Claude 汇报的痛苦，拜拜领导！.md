---
title: 我写了个工具，彻底解决了 Claude 汇报的痛苦，拜拜领导！
author: AI开源前哨
date: 前哨君前哨君
url: https://mp.weixin.qq.com/s?__biz=MzkxNjQzNTc4NA==&mid=2247489241&idx=1&sn=5eeb63c28ced97b263495bbdda6df733&chksm=c04074f560b6165eaf422a6c56eadeb82ea3f75e9309f026b47805c708ddc906095770568414&mpshare=1&scene=24&srcid=0519sWNHpGm0TPnXCEqhFYFU&sharer_shareinfo=a072bc180e6f78c32d6b91bcb32f55d4&sharer_shareinfo_first=a072bc180e6f78c32d6b91bcb32f55d4#rd
---

最近领导开始要求汇报AI工具的使用情况了。

每周要统计用了多少次会话、消耗了多少Token、在哪些项目上用得最多、主要用来做什么工作，还要结合代码提交量来证明AI确实提升了效率。

这些数据散落在Claude的本地日志和各个Git仓库里，手动整理一次至少要花半小时，还容易出错。

虽然有很多的工具可以进行使用统计，但是再去要结合自己的项目工作去统计然后直接呈现出来的工具我还没找到合适的。

而这个功能看下来也很简单，为了省事，我于是花了一个下午撸了这个小工具，ccusage-report。

它借助开源工具 ccusage，自动从 Claude Code 的 JSONL 日志和本地 Git 仓库提取所有需要的数据，生成可视化效率报告，还能一键导出可直接提交的工作汇报。后续还会整合支持opencode和codex等。

### 这个工具能帮你做什么

ccusage-report不是一个简单的使用统计工具，它真正做到了把AI使用数据和实际开发产出结合起来，给你一个完整的效率画像。

**第一，数据整合能力强**。它读取 Claude Code 本地存储的所有会话记录、请求次数和 Token 消耗，还能关联你指定的 Git 仓库，自动统计对应时间段内代码提交次数、新增删除行数和变更文件数。

这样你就能看到，哪些项目投入了更多 AI 资源，这些资源又转化成了多少实际代码产出。

**第二，报告体系覆盖三种时间范围**。日报、周报、月报，正好对应日常工作汇报场景。日报帮你快速回顾当天工作，周报提交给团队，月报用作个人月度效率复盘。每份报告包含会话数、请求数、Token 总消耗、场景分布、模型使用排名、项目分布和工具调用统计，数据一目了然。

最实用的是**一键生成工作汇报**。点击 Web 界面的“生成工作汇报”按钮，或者命令行加上 work 参数（也就是 --work），工具自动把所有分析数据整理成标准 Markdown 格式汇报文档。

文档内容有使用概览表格、场景分布饼图、项目明细和 Git 指标，复制一下就能发到飞书、钉钉或邮件，省去手动排版的麻烦。

最后，**配置非常灵活**。通过 Web 界面右上角的设置按钮在线修改，或者直接编辑 config.json 文件都行。Claude 数据目录、关联的 Git 仓库路径、要排除的测试项目，全都能自定义。场景分类关键词也可以按自己的工作习惯去添加或修改，让工具更准确地识别工作类型。

### 5分钟快速上手

这个工具基于Node.js开发，没有任何额外依赖，下载就能运行。

#### 环境要求

只需要你的电脑上安装了Node.js 18.0.0或更高版本。

#### 基本使用

```
# 克隆项目到本地  
git clone https://github.com/yaowen51888-rich/ccusage-report.git  
cd ccusage-report  
  
# 启动Web服务（会自动打开浏览器）  
node index.js serve
```

Web服务默认运行在4567端口，如果你想修改端口，可以通过环境变量指定：

```
set CCUSAGE_PORT=8080 && node index.js serve
```

如果你更喜欢用命令行，也可以直接生成报告：

```
# 生成今日日报  
node index.js report daily  
  
# 生成指定日期的日报  
node index.js report daily 2026-05-15  
  
# 生成本周周报  
node index.js report weekly  
  
# 生成本月月报  
node index.js report monthly 2026-05-01
```

#### 常用命令示例

```
# 只统计指定项目的数据  
node index.js report daily --projects D://fzwork,E://play/idea  
  
# 直接生成Markdown格式的工作汇报  
node index.js report daily --work  
node index.js report weekly --work  
  
# 初始化配置文件  
node index.js init
```

#### 配置文件详解

工具的所有配置都保存在根目录的config.json文件中，第一次运行时会自动生成默认配置，你只需要根据自己的情况修改即可。

```
{  
  "claudeDir":"C:\\Users\\<用户名>\\.claude",  
"repos":["D:\\work\\project1","E:\\dev\\project2"],  
"excludeProjects":[],  
"scenarioKeywords":{  
    "coding":["实现","功能","开发","implement","feature","refactor","重构"],  
    "testing":["测试","test","覆盖率","coverage","jest","vitest"],  
    "debugging":["修复","bug","debug","fix","报错","错误","异常","error"],  
    "documentation":["文档","doc","readme","注释","说明","指南"],  
    "review":["review","审查","代码审查","/review"],  
    "planning":["计划","plan","设计","架构","方案","design","architect"]  
}  
}
```

* • `claudeDir`：Claude Code的数据目录，一般在用户目录下的.claude文件夹
* • `repos`：需要关联的Git仓库路径数组，可以添加多个
* • `excludeProjects`：要排除统计的项目名称数组
* • `scenarioKeywords`：场景分类关键词，工具会根据你和Claude的对话内容匹配对应的工作类型

对于那些需要向团队或上级汇报AI工具使用效果的开发者来说，这个工具能帮你节省大量整理数据的时间，让你把精力集中在真正的开发工作上。

如果你也在使用Claude Code，并且被每周的工作汇报困扰，不妨试试这个工具。项目已经开源在GitHub上，欢迎大家Star和提交Issue。

项目地址

https://github.com/yaowen51888-rich/ccusage-report

"前哨君"精选AI开源项目网站

https://www.yaowendeep.cn

欢迎 置顶（标星）关注本公众号「AI开源前哨」获取有趣AI技术/工具分享,这样就第一时间获取推送啦~

[我在项目根目录放了一个文件，AI再也没把UI做丑过！](https://mp.weixin.qq.com/s?__biz=MzkxNjQzNTc4NA==&mid=2247489225&idx=1&sn=1766d31203455387c34f6a36b4ef9b69&scene=21#wechat_redirect)

  

[Git 之后，AI 编程终于迎来了自己的“版本控制系统”](https://mp.weixin.qq.com/s?__biz=MzkxNjQzNTc4NA==&mid=2247489212&idx=1&sn=7438b24d4fd2e29df33e0924179e8498&scene=21#wechat_redirect)

  

[开源仅 10 天，狂揽 30K Star！Claude Design 的终极开源平替杀疯了](https://mp.weixin.qq.com/s?__biz=MzkxNjQzNTc4NA==&mid=2247489200&idx=1&sn=d8e664bc9a7a02b20fcf6ed997640c04&scene=21#wechat_redirect)