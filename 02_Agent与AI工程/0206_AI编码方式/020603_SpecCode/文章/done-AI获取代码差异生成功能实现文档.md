> 已吸收至：[[02_Agent与AI工程/0206_AI编码方式/020603_SpecCode/020603_核心知识点/SpecCode规格驱动与验收边界|SpecCode规格驱动与验收边界]]
---
title: AI获取代码差异生成功能实现文档
author: 三七互娱技术团队
date:
url: https://mp.weixin.qq.com/s?__biz=MzUzNzAzMzQwNg==&mid=2247491821&idx=1&sn=7386e9767c994e2d9fe00933b3d83adb&chksm=fbf99e83dfcd344267c727bd54ae56c25fc27fe60827f09ee74cec929fe475dc27f491b4f935&mpshare=1&scene=24&srcid=1212XnJdnP0rkTxSde7DRNuy&sharer_shareinfo=7491ad4a7f12b276131dc3fe6a6b0bbe&sharer_shareinfo_first=7491ad4a7f12b276131dc3fe6a6b0bbe#rd
---

01

# 背景

在SDK日常功能迭代中，完成一个较大功能的开发后，通常需要编写文档来帮助团队成员理解功能逻辑并方便后续维护。然而，整理技术实现细节、绘制流程图、架构图等往往需要耗费大量时间。此时，借助AI技术对比代码差异，自动生成功能实现文档，将有助于提升工作效率。

02

# cursor生成

## 代码差异获取

在cursor聊天框中有，Git有以下对比功能：

* Commit(Diff of  Working State： “审查工作状态” 对比未提交更改
* Branch(Diff of Main Branch：“审查与主分支的差异”这将审查当前分支与主分支之间的差异
* commit记录：审核做的提交记录

使用GIT=>Branch(Diff with Main Branch)

对比主分支，可以获取功能实现的代码差异

## 提示词生成

提示词 Prompt ,在.cursor/rule下生成tech-doc-generation.mdc文件

## 文档生成

在cursor聊天框中，@tech-doc-generation提示词和@Branch(Diff of Main Branch 本地生成markdown格式的文档

03

# claude code生成

使用tech-doc-generation.mdc提示词

对比功能分支与master的差异,执行Todos列表

生成的markdown文档

04

# 自动创建wiki文章

## token获取

在 wiki个人主页下 获取Token

## 配置MCP服务

###### cursor配置

在cursor中配置mcp-atlassian服务（Open Settings → MCP → + Add new global MCP server）

服务运行在docker中，需要先安装docker环境

###### claude code配置

参数的含义：

* CONFLUENCE\_URL：wiki的域名
* CONFLUENCE\_PERSONAL\_TOKEN：个人访问wiki权限的令牌
* READ\_ONLY\_MODE：读取权限，设置为“true”以禁用写入操作
* MCP\_VERBOSE：设置为“true”以获得更详细的日志记录
* ENABLED\_TOOLS：要启用的功能逗号分隔列表

启用功能的定义：

* confluence\_search：使用 CQL 搜索 Confluence 内容
* confluence\_get\_page：获取特定页面的内容
* confluence\_create\_page：创建新页面
* confluence\_update\_page：更新现有页面

## MCP服务运行

拉取docker镜像，运行docker容器

## 优化内容显示

Confluence wiki加载markdown格式文档 需要增加自定义markdown宏

wiki文章需要先插入markdown宏格式，再加载markdown内容，提示词如下：

自动生成、并创建wiki文章

05

# 总结

在以上技术文档生成方面，Cursor 与 Claude Code 展现出不同的侧重点：Cursor 更聚焦于代码层面的细节分析，产出内容技术颗粒度细，但略显零散；而 Claude Code 则更侧重于功能实现与架构的整体阐述，其产出逻辑连贯，更符合开发人员的文档阅读习惯。AI生成文档的意义在于，将开发者从繁琐的文档工作中解放出来，使其能更专注于核心开发任务。此外，在代码审查环节，AI生成的摘要能为审查者提供即时、精准的上下文，加速审查过程并提高代码质量。

# 相关资源

* mcp-atlassian服务
* claude-code文档

###### 三七互娱技术团队

扫码关注 了解更多