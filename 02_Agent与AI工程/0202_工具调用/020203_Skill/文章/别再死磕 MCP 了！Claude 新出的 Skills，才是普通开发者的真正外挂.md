---
title: 别再死磕 MCP 了！Claude 新出的 Skills，才是普通开发者的真正外挂
author: 墨痕AI编程
date: 
url: https://mp.weixin.qq.com/s?__biz=MzYzODAyMzU5Nw==&mid=2247483705&idx=1&sn=5131b3b55346b53ee5a1f205130bda49&chksm=f182ee0cef606405e9d70505dccbd825a5383aab9a6df2a820941ec80cd524d8b815e5e88775&mpshare=1&scene=24&srcid=1021qns7BCG663PMBnleVHWB&sharer_shareinfo=a650a786dacc4cb3b869e42d52e10de6&sharer_shareinfo_first=a650a786dacc4cb3b869e42d52e10de6#rd
---

昨天下午，我正被一堆重复性工作折磨：给几十个函数补文档、重命名日志文件、跑单元测试……

突然想起刚看到的 **Claude Skills**，随手试了试

把一个写文档的 Python 脚本封装成 Skill，丢进项目目录。  
结果，只在 Claude Code 里说一句：

> “帮我给这个函数生成 Google 风格的 docstring。”

它就自动调用脚本，输出格式完美、缩进对齐的文档，连参数类型都解析出来了。

再次之前，我经常为配置 **MCP（Model Context Protocol）** 折腾到凌晨，token 爆了三次，最后干脆放弃。

## 一、先说清楚：Claude Skills 到底是什么？

你可以把它想象成 **“给 Claude 装插件”**。

Claude Skills像是直接给AI装上了一个个“专业App”。

需要处理Excel？打开“Excel大师”App，里面全是数据透视表、公式填充的SOP。

需要画图？启动“设计师”App，它会严格按照你的品牌色板、字体规范来输出。

需要做个GIF？加载“Slack GIF Creator”App，它会先检查文件大小，再开始生成。

而且这个“调用”**不需要联网、不用部署服务器、不占对话上下文**

你只需要在项目文件夹里放一个说明文件和一个脚本，Claude 就能“认出”它，并在合适的时候用起来。

## 二、MCP 很酷，但真的不适合我这样的普通开发者

过去几个月，MCP 被吹成“AI Agent 的终极协议”。  
听起来很厉害，但实际用起来**太重了**。

你要写 JSON schema、部署本地服务、处理认证、调试跨域……  
一个简单的“查日志”功能，光配置就要半小时。

更头疼的是，MCP 要求把所有工具描述一次性塞进上下文。  
项目一大，光工具说明就占几万 token，真正写代码的空间没了。

MCP 没错，它面向的是企业级系统集成。

但对我们这些只想**少写点重复代码、多喝杯咖啡**的开发者来说，它就像让你先考驾照、买车、建车库，才能出门买个菜。

## 三、Claude Skills：Anthropic 给开发者的外挂

就在大家被各种协议绕晕时，Anthropic 悄悄上线了 **Claude Skills**。  
虽然是悄咪咪的上线，但迅速引起了大家的重视。

为什么？因为 **它真的太“轻”了**。

一个 Skill，就是一个文件夹，里面放一个SKILL.md 说明文件，再加你的脚本（Python、Bash、JS 都行）。

Claude 只在需要时加载它，用完就走，不占上下文，也不浪费 token。

最让我惊喜的是：**它支持动态组合**。

比如，我可以先用一个 Skill 解析日志，再用另一个 Skill 生成图表，Claude 会自动判断调用顺序。

而 MCP 一旦加载，就固定不变，灵活性差很多。

说白了：MCP 是给平台用的，Skills 是给普通人用的。

## 四、实测：5 分钟，让 AI 帮你自动重命名文件

别被“Skill”这个词吓到。创建它，比写一个 Bash 脚本还简单。

场景：我想把所有 .log 文件批量加上前缀 error\_

#### 第一步：建个文件夹

```
mkdir -p .claude/skills/batch-rename
```

第二步：写个说明文件 SKILL.md

```
# Batch Rename Files  
重命名当前目录下所有指定后缀的文件，加上前缀和后缀。  
例如用户说：“把 .log 文件加上前缀 error_”，Skill 会运行：python rename.py --prefix error_ --ext .log
```

第三步：写个脚本 rename.py

```
import osimport argparse  
parser = argparse.ArgumentParser()parser.add_argument("--prefix", default="")parser.add_argument("--suffix", default="")parser.add_argument("--ext", default=".log")args = parser.parse_args()  
for f in os.listdir("."):    if f.endswith(args.ext):        new_name = args.prefix + f.replace(args.ext, "") + args.suffix + args.ext        os.rename(f, new_name)        print(f"✅ Renamed {f} → {new_name}")
```

#### 第四步：在 Claude Code 里直接说

> “用 batch-rename 技能把所有 .log 文件加上前缀 error\_”

搞定！

**无需部署、无需 API、不用改任何配置**。只要这个文件夹在项目里，Claude 就能用它。

我昨天用这个 Skill 处理了 127 个日志文件，全程只说了两句话。  
省下的时间，够我写完这篇文章了。

官方 GitHub 已开源多个示例（代码生成、数据清洗、测试生成等），你可以直接 fork 改造。

## 五、为什么我觉得新号博主该关注 Skills？

作为刚起步的 AI 编程博主，我发现 **Claude Skills 是个被严重低估的机会**：

* 中文圈深度解读少，内容稀缺；
* 读者看完就能动手，容易收藏转发；
* 它天然适合做系列：模板分享、场景拆解、效率对比……

更重要的是，**它代表了 AI 编程的新方向！**

参考资料：

* Claude Skills 官方文档
* anthropics/skills GitHub 示例库

欢迎讨论关于AI编程的知识，我会经常分享自己的编程心得和机遇风口，扫码！