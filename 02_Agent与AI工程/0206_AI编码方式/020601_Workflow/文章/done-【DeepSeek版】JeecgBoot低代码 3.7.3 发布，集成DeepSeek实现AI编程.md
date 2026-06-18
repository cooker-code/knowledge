> 已吸收至：[[02_Agent与AI工程/0206_AI编码方式/020601_Workflow/020601_核心知识点/Workflow编排与验证闭环|Workflow编排与验证闭环]]
---
title: 【DeepSeek版】JeecgBoot低代码 3.7.3 发布，集成DeepSeek实现AI编程
author: JEECG
date:
url: http://mp.weixin.qq.com/s?__biz=MzIwMDE0OTgzNA==&mid=2650413227&idx=1&sn=6aea64acf0bc941f2c810b10ce09815b&chksm=8f62264cad9bf40bf7d50d87b4c9ba266c7c1932a99fedbb1f440e858fc3fcfe47c9a10871e3&mpshare=1&scene=24&srcid=02104HEKxUjXz3hnRZIPPrHR&sharer_shareinfo=3fb7764c0e375239b80636e2bf368952&sharer_shareinfo_first=3fb7764c0e375239b80636e2bf368952#rd
---

### **项目介绍**

> JeecgBoot是一款企业级的AI低代码平台！前后端分离架构 SpringBoot2.x/3.x，SpringCloud Alibaba，Ant Design&Vue3，Mybatis-plus，Shiro;支持AI大模型DeepSeek和ChatGPT、Ollama本地模型;强大的代码生成器让前后端代码一键生成，无需写任何代码! 引领AI低代码新开发模式：AI生成-> OnlineCoding-> 代码生成-> 手工MERGE， 帮助Java项目解决80%重复工作，让开发更多关注业务。既能快速提高效率，节省成本，同时又不失灵活性！AIGC能力：AI对话助手、AI建表、AI写文章、AI流程编排、AI知识库等。

**当前版本**：v3.7.3 | 2025-02-10

#### **源码下载**

* <https://github.com/jeecgboot/JeecgBoot>

####

#### **升级日志**

> 春节被DeepSeek刷屏了，这个火出圈的产品 JeecgBoot 速速跟上，最新版已经适配三个AI大模型： [DeepSeek v3版](https://help.jeecg.com/java/DeepSeekzhichi.html)、[ChatGPT](https://help.jeecg.com/java/chatgpt.html) 和 [DeepSeek-R1本地大模型](https://help.jeecg.com/java/ai/DeepSeekR1.html)，推荐使用 DeepSeek 速度更快、质量更高。

##### **issue处理**

####

* JeecgBoot 低代码 AI 大模型支持DeepSeek和ChatGPT切换
* JeecgBoot 支持对接Ollama安装的本地大模型DeepSeek-R1
* 升级前端依赖vite6、antd3.4.19、antd4.2.6
* JimuReport升级到最新版1.9.3
* JimuBI大屏升级到最新版1.9.3
* 租户套餐管理优化体验
* AutoPoi Excel表格导入有问题，还会报个错。#7703
* 首页AI助手不明显优化
* 【issues/7709】当dataSource是响应式时，单元格编辑输入会自动关闭
* 【issues/7549】Online 表单开发 页面属性 查询选择模糊查询 结果生成的代码是 JRangeNumber 而且页面中不显示：父子表
* jvxetable 字典问题 · [Issue #7497](https://github.com/jeecgboot/JeecgBoot/issues/7497)
* Redis 锁无法释放，造成redis死锁造成大量的redis exists redis命令引起redis QPS异常飙升 · [Issue #6876](https://github.com/jeecgboot/JeecgBoot/issues/6876)
* 操作失败，Error in execution; nested exception is io.lettuce.core.RedisCommandExecutionException: ERR unknown command"keys" with args beginning with: `sys:cache:online:list..\*, \* · [Issue #6918](https://github.com/jeecgboot/JeecgBoot/issues/6918)
* 显示左侧logo时会导致下面菜单滚动显示不全 · [Issue #7548](https://github.com/jeecgboot/JeecgBoot/issues/7548)
* 主题切换为顶部混合模式时，页面顶部内容显示不出来，被遮盖 · [Issue #7561](https://github.com/jeecgboot/JeecgBoot/issues/7561)
* 账户设置->修改手机号：获取验证码接口 404 错误 · [Issue #7587](https://github.com/jeecgboot/JeecgBoot/issues/7587)
* 最新版样式错乱 · [Issue #7605](https://github.com/jeecgboot/JeecgBoot/issues/7605)
* Online 表单开发 页面属性 查询选择模糊查询 结果生成的代码是 JRangeNumber 而且页面中不显示 · [Issue #7549](https://github.com/jeecgboot/JeecgBoot/issues/7549)
* 3.7.1 bug：JVxeTable 单选删除不生效 · [Issue #7624](https://github.com/jeecgboot/JeecgBoot/issues/7624)
* v3.7.2弹窗全屏底部有空隙 · [Issue #7601](https://github.com/jeecgboot/JeecgBoot/issues/7601)
* JVxeTable组件代码与JeecgBoot前端文档内容不符（getValue方法） · [Issue #7631](https://github.com/jeecgboot/JeecgBoot/issues/7631)
* 修改手机号报 404 错误 · [Issue #7681](https://github.com/jeecgboot/JeecgBoot/issues/7681)
* JvxeUserSelectCell 组件，希望能把maxTagCount 参数改成props而不是写死为1 · [Issue #7661](https://github.com/jeecgboot/JeecgBoot/issues/7661)
* JVxeTable表格@blur监听textarea组件会重复触发事件 · [Issue #7664](https://github.com/jeecgboot/JeecgBoot/issues/7664)
* 官网演示版本中“我的部门”功能数据展示异常 · [Issue #7658](https://github.com/jeecgboot/JeecgBoot/issues/7658)
* js增强onlchange事件 · [Issue #7642](https://github.com/jeecgboot/JeecgBoot/issues/7642)
* 3.7.2前端install后dev启动后报错 · [Issue #7644](https://github.com/jeecgboot/JeecgBoot/issues/7644)
* 升级3.7.2 flyway自动升级失败 · [Issue #7650](https://github.com/jeecgboot/JeecgBoot/issues/7650)
* JVxeTypes.image组件action字段只能定义第1张图片的上传接口，后面图片的接口还是使用公共上传接口 #7750
* sys\_announcement\_send表的sql文件没有设置id为主键 #7725
* 升级AI助手，deepseek 每次发送新的消息提问，会把之前的提问消息历史重复发送 #7754

#### **AIGC功能清单**

* AI对聊天助手
* AI建表（Online表单）
* AI流程编排（研发中）
* AI知识库问答系统（研发中）
* AI应用开发平台（研发中）
* AI聊天窗口支持嵌入第三方（研发中）

#### **技术交流**

* 开发文档：[https://help.jeecg.com‍](https://help.jeecg.com/)
* 在线演示：[http://boot3.je‍ecg.com](http://boot3.jeecg.com/)
* 快速入门：[新手指南](https://jeecg.com/doc/quickstart) | [代码生成](https://help.jeecg.com/java/codegen/online.html)
* AI能力： [AI对话助手](https://help.jeecg.com/java/ai/aichat.html) | [AI大模型支持](https://help.jeecg.com/java/AImoxingzhichi.html)
* 快速体验： [一分钟体验低代码](https://jeecg.blog.csdn.net/article/details/106079007?spm=1001.2014.3001.5502) | [在线体验零代码](https://app.qiaoqiaoyun.com/myapps/index)
* 视频教程: [http://‍jeecg.com/doc/video](http://jeecg.com/doc/video)

#### **为什么选择 JeecgBoot?**

> 开源界"小普元"超越传统商业平台。引领低代码开发模式(OnlineCoding-> 代码生成器 -> 手工MERGE)，低代码开发同时又支持灵活编码， 可以帮助解决Java项目70%的重复工作，让开发更多关注业务。既能快速提高开发效率，节省成本，同时又不失灵活性。

* 采用最新主流前后分离框架(SpringBoot+Mybatis-plus+Ant-Design+Vue)，容易上手; 代码生成器依赖性低,灵活的扩展能力，可灵活实现二次开发;
* 开发效率很高,采用代码生成器，单表数据模型和一对多(父子表)、树列表等数据模型，增删改查功能自动生成，菜单配置直接使用（前端代码和后端代码都一键生成）；
* 支持主流的AI大模型：支持 ChatGPT、DeepSeek、Ollama本地搭建大模型等
* 提供AI对话助手、AI建表、AI写文章等AIGC功能
* 代码生成器提供强大模板机制，支持自定义模板风格。目前提供四套风格模板（单表两套、一对多两套）
* 封装完善的用户、角色、菜单、组织机构、数据字典、在线定时任务等基础功能。强大的权限机制，支持访问授权、按钮权限、数据权限、表单权限等
* 零代码在线开发能力，在线配置表单、在线配置报表、在线配置图表、在线设计表单
* 常用共通封装，各种工具类(定时任务,短信接口,邮件发送,Excel导入导出等),基本满足80%项目需求
* 简易Excel导入导出，支持单表导出和一对多表模式导出，生成的代码自带导入导出功能
* 集成简易报表工具，图像报表和数据导出非常方便，可极其方便的生成图形报表、pdf、excel、word等报表；
* 采用前后分离技术，页面UI精美，针对常用组件做了封装：时间、行表格控件、截取显示控件、报表组件，编辑器等等
* 查询过滤器：查询功能自动生成，后台动态拼SQL追加查询条件；支持多种匹配方式（全匹配/模糊查询/包含查询/不匹配查询）；
* 数据权限（精细化数据权限控制，控制到行级，列表级，表单字段级，实现不同人看不同数据，不同人对同一个页面操作不同字段
* 在线配置报表（无需编码，通过在线配置方式，实现曲线图，柱状图，数据等报表）
* 页面校验自动生成(必须输入、数字校验、金额校验、时间空间等);
* 提供单点登录CAS集成方案，项目中已经提供完善的对接代码
* 表单设计器，支持用户自定义表单布局，支持单表，一对多表单、支持select、radio、checkbox、textarea、date、popup、列表、宏等控件
* 专业接口对接机制，统一采用restful接口方式，集成swagger-ui在线接口文档，Jwt token安全验证，方便客户端对接
* 接口安全机制，可细化控制接口授权，非常简便实现不同客户端只看自己数据等控制
* 高级组合查询功能，在线配置支持主子表关联查询，可保存查询历史
* 提供各种系统监控，实时跟踪系统运行情况（监控 Redis、Tomcat、jvm、服务器信息、请求追踪、SQL监控）
* 消息中心（支持短信、邮件、微信推送等等）
* 集成Websocket消息通知机制
* 提供APP发布方案：
* 支持多语言，提供国际化方案；
* 数据变更记录日志，可记录数据每次变更内容，通过版本对比功能查看历史变化
* 平台UI强大，实现了移动自适应
* 平台首页风格，提供多种组合模式，支持自定义风格
* 提供简单易用的打印插件，支持谷歌、IE浏览器等各种浏览器
* 示例代码丰富，提供很多案例参考
* 采用maven分模块开发方式
* 支持菜单动态路由
* 权限控制采用 RBAC(Role-Based Access Control，基于角色的访问控制)

#### **系统效果预览**

##### **AI功能**

**AI聊天助手**

**AI建表**

**AI写文章**

##### **积木BI大屏**

##### **PC端**

##### **在线聊天&通知**

##### **图表示例**

##### **APP效果**

##### **PAD端**

##### **在线接口文档**

##### **积木报表**

欢迎吐槽，欢迎star~