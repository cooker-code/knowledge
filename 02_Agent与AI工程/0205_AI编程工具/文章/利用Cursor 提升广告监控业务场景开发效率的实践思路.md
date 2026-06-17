---
title: 利用Cursor 提升广告监控业务场景开发效率的实践思路
author: 三七互娱技术团队
date: 
url: https://mp.weixin.qq.com/s?__biz=MzUzNzAzMzQwNg==&mid=2247491370&idx=1&sn=82fae5d5ed0f4a4f3808de7090155f2a&chksm=fbae86281bdd3653c6675861c256dc298ef529c9f19de9b5ec9d36108a6e72332fd06e232d2b&mpshare=1&scene=24&srcid=0806fYl38lBxA6JUr7HTkaHe&sharer_shareinfo=44dad6d45611a7099d2b0da544df82e7&sharer_shareinfo_first=44dad6d45611a7099d2b0da544df82e7#rd
---

01

# 背景

### Cursor简介

Cursor 是一款集成了 AI 技术的代码编辑器，它继承了 VS Code 的强大功能，并融入了 AI 功能，简化了开发工作流。Cursor 的能力主要包括：自然语言编写代码、智能代码补全、与项目代码对话以获取优化建议等。

### 业务背景

在我们的广告监控业务开发中，主要需要处理以下几类场景：

1. 多媒体平台数据查询和处理（头条、快手、腾讯等）
2. 多媒体平台数据指标管理和计算
3. 定时任务、常驻任务处理

这些场景具有以下特点：

* 代码模式相似，多为重复性工作
* 需要严格遵循既定的开发规范
* 容易出现人为疏忽

02

# 实现

根据业务背景，我们可以利用cursor来提升开发效率。

### 全局规则

主要是明确下项目上下文信息和工作路径，使得后面场景生成的代码更符合整体项目要求。

项目的上下文信息主要是对整体项目框架和目录进行说明。

```
# 项目上下文 本文档为 AI 工具提供项目的上下文信息，以帮助生成相关且准确的代码建议。## 项目概览- **框架** ：基于goframe 二开的叫tcf的框架## 目录结构```plaintext- api # 对外提供服务的输入/输出数据结构定义。考虑到版本管理需要，往往以api/v1…存在。- v1- cmd # web ，daemon ，job 三种服务启动入口  - daemon # 常驻进程程序入口 go run cmd/daemon/main.go  - job # job定时任务程序入口 go run cmd/job/main.go  - web # web服务程序入口 go run cmd/web/main.go- data  - kuaishou # 快手相关测试数据文件  - tengxun # 腾讯相关测试数据文件  - toutiao # 头条相关测试数据文件- deployments_sygn # tke配置文件- hack # 存放项目开发工具、脚本等内容。例如，CLI工具的配置，各种shell/bat脚本等文件。- internal # 业务逻辑存放目录。通过Golang internal特性对外部隐藏可见性。  - cmd # 命令行管理目录。可以管理维护多个命令行。    - daemon    - job    - web  - consts # 项目所有常量定义。  - controller # 项目所有常量定义。  - daemon # 执行常驻进程的命令和方法  - dao # 数据访问对象，这是一层抽象对象，用于和底层数据库交互，仅包含最基础的 CURD 方法  - job # 执行定时任务的命令和方法  - logic # 业务逻辑封装管理，特定的业务逻辑实现和封装。往往是项目中最复杂的部分。  - model # 数据结构管理模块，管理数据实体对象，以及输入与输出数据结构定义。  - service # 用于业务模块解耦的接口定义层。具体的接口实现在logic中进行注入。- manifest # 包含程序编译、部署、运行、配置的文件。常见内容如下  - config # 配置文件存放目录。  - docker # Docker镜像相关依赖文件，脚本文件等等。- resource # 静态资源文件。这些文件往往可以通过 资源打包/镜像编译 的形式注入到发布文件中。- go.mod # 使用Go Module包管理的依赖描述文件。```
```

### 定时任务规则

定时任务相对比较独立，所以这里我是单独写了一个规则，让其更契合整体项目，更加通用，更加契合整体项目。

规则如下：

```
 # 开发规范指南 ## 1. 定时任务开发规范 ### 目录结构 - 代码存放目录：`internal/job/` - 参考示例：`job_target_holo_import.go`  ### 开发流程 1.需求分析  - 明确任务目标和执行周期- 确定输入参数和数据源- 定义输出结果格式  2.代码实现   - 使用 `parser.GetOpt` 方法获取输入参数 - 遵循现有代码结构和命名规范    - 实现核心业务逻辑    - 添加必要的日志记录    - 完善错误处理机制   3.测试验证 - 执行脚本验证功能是否生效 - 确保与现有功能兼容  - 验证错误处理机制  - 检查日志输出是否完整  ## 2.代码质量要求 ### 代码结构 - 模块化设计，合理拆分功能 - 复用现有代码，参考同类媒体实现 - 保持代码结构清晰，易于维护 - 规范db操作类和方法，参考同类数据表实现  ### 错误处理 - 全面的错误处理机制 - 详细的错误日志记录 - 合适的错误返回方式  
 ### 数据处理 - 数值型指标：注意精度控制和格式化 - 字符串型指标：处理空值和多值聚合 - 时间类型：统一时区和格式处理  ### 文档和注释 - 添加必要的代码注释 - 说明关键参数和返回值 - 记录特殊处理逻辑 - 更新相关文档  ## 3.开发注意事项 ### 开发前 - 仔细阅读需求文档 - 理解现有代码实现 - 制定开发计划  ### 开发中 - 遵循代码规范 - 注意性能优化 - 保持代码简洁  ### 开发后 - 完整功能测试 - 代码审查 - 文档更新
```

示例：帮我生成一个job，输入参数有media\_id（int类型），功能是找到该媒体的指标excel文件并转成json。文件目录：data/{媒体拼音}（这里需要通过media\_id转化)，指标excel文件名：target.xlsx，json文件名称：target.xlsx。

生成结果基本满足我的要求

03

# 场景规则

根据业务背景，场景这里我分为三种类型：

### 场景1：已支持媒体新增属性指标

这里说明下，我们的广告监控主要分了这几个类型：账户监控、广告第一层级监控、广告第二层级监控、创意层级监控，这些监控类型基本上都包含广告属性的筛选。因此，项目设计之初属性指标都是写在代码里面的。

基于这种场景，当我们媒体有新增的属性指标，我们就可以利用这个场景巧妙地在对应媒体位置加上对应的代码。

示例：腾讯媒体，新增属性指标一键起量状态（auto\_acquisition\_statustext）、一键起量预算（auto\_acquisition\_budget）。

结果：

### 场景2：已支持媒体导入数据指标

项目设计之初，除了广告的属性指标，还有各种各样的数据指标（媒体拉取的数据指标和我们的统计指标）。因为监控需要支持不同时间范围，每个媒体综合起来会有几百上千个指标，所以设计上是加了一个指标查询方式的配置表，所以该场景很大的工作量是梳理这份配置表。

输入：业务和技术梳理的数据指标，一份excel表

输出：将excel表的内容导入到配置表中（输入的excel表包含一些自然语言，需要做好转化）

因为cursor不好读取excel的内容，我这里换了一个实现思路：

1、将输入excel转成输入json文件（这里可以利用定时任务规则生成excel转json的定时脚本，来快速实现）

2、将输入json文件根据场景要求转成目标json文件

3、将目标json文件转成目标excel文件（这里为什么又转成excel是因为之前已有一个excel导入指标配置表的脚本，刚好衔接起来）

源文件：

目标文件：这里会把自然语言的计算公式拆解成分子分母的英文key

### 场景3：新媒体接入

示例：新接入媒体，如b站

结果：代码可以正常生成，但是查询中表名有时候对不上，可以在提示词上加上表名说明。

场景的规则如下：

```
# 开发场景 ## 场景1：已支持媒体新增属性指标 - 要求说明什么媒体，目前已支持头条(toutiao，media_id=1)、快手(kuaishou，media_id=5)、腾讯(tengxun，media_id=-4) - holo数据查询相关代码目录：internal/logic/mq_sm_data_holo，如头条数据查询对应该目录下的mq_sm_data_holo_toutiao.go文件 - 有说明指定层级，则在指定层级添加属性指标（方法名buildPropertyDimension），否则在通用方法加上属性指标（方法名getPropertyCommonColumn） - 有说明指标key，则直接使用，没有说明具体指标key，通过Doc联网查询对应的指标信息 - 不改变代码的已有整体逻辑 - 新增指标没有做特殊函数转化处理的话，不需要定义常量 - 不改动job文件job_target_holo_import.go - 添加指标时注意使用COALESCE函数处理空值，并使用string_agg函数聚合多个值 - 对于数值型指标，考虑使用round函数保留适当的小数位数## 场景2：已支持媒体指标导入### 前置条件 - 确认媒体类型：支持头条(toutiao，media_id=1)、快手(kuaishou，media_id=5)、腾讯(tengxun，media_id=-4) - 确认工作目录下存在必要文件： - 源文件：`target.xlsx` - 目标文件：`target_holo_import.xlsx`### 处理步骤#### 数据预处理 - 检查工作目录下是否存在中间文件`target.json`、`target_holo_import.json`，存在则需要将其删除 - 将源文件`target.xlsx` 转换为 json格式文件`target.json`，可执行脚本go run cmd/job/main.go jobTargetExcelToJson --media_id={媒体id} --file_name=target - 全文件分析 `target.json`，请确保读取完整文件内容#### 数据转化处理 - 内容转化：将`target.json`转化成`target_holo_import.json`，允许多次执行，最终要确保转化后的指标数正确，转化要求如下：1. **指标筛选规则** - 包含条件：字段“归属第一层级”为“媒体数据指标” - 分类处理：将指标分为“求和”、“比率”、“成本”三类2. **字段完整性校验** - 求和类指标：必须包含 [指标名称、指标key、数据表key、是否金额类], `是否金额类`字段枚举值为0、1 - 比率类指标：必须包含 [指标名称、指标key、分子key、分母key、依赖key] - 成本类指标：必须包含 [指标名称、指标key、分子key、分母key、依赖key]3. **计算公式处理** - 解析“计算公式”字段，拆分为分子key和分母key - 中文指标映射：根据求和类指标或“参考指标”将中文指标名转换为英文key(参考`target.json`来补全)4. **指标数校验** - 全文件分析 `target_holo_import.json`，请确保读取完整文件内容 - 比较`target.json`，请保证`target_holo_import.json`的指标数与`target.json`筛选后的指标数一致5. **转化结果json格式校验** - json格式参考：```{"求和": [  {    "指标名称": "账户消费",    "指标key": "COST",    "数据表key": "COST",    "是否金额类": "是"  },],"比率": [  {    "指标名称": "点击率",    "指标key": "CLICK_RATE",    "分子key": "CLICK",    "分母key": "SHOW",    "依赖key": "CLICK,SHOW"  },],"成本": [  {    "指标名称": "点击均价",    "指标key": "CPC",    "分子key": "COST",    "分母key": "CLICK",    "依赖key": "COST,CLICK"  }]}```- 格式转化：将json格式文件`target_holo_import.json`转成目标文件`target_holo_import.xlsx`，可执行脚本go run cmd/job/main.go jobTargetJsonToExcel --media_id={媒体id} --file_name=target_holo_import- 指标导入：将目标的excel执行导入，可执行脚本go run cmd/job/main.go jobTargetHoloImport --media_id={媒体id}### 注意事项1. **数据处理**- 检查指标key的唯一性- 验证计算公式的正确性2. **格式要求**- 保持与参考文件相同的数据结构- 确保生成文件的编码格式正确3. **错误处理**- 记录无法处理的指标- 保存处理过程的日志4. **结果验证**- 检查生成文件的完整性- 验证关键指标的计算结果## 场景3：接入新媒体- 要求说明接入新媒体名称、id、拼音- 有说明新媒体指标key，则直接使用，没有说明具体指标key，通过Doc查询对应的所有指标信息- 定义新媒体常量，文件为internal/consts/mq_monitor.go- 添加媒体ID常量（如MqMediaIdWithXxx）- 如有必要，添加媒体特有的筛选条件转换映射（MqMonitorFilterPropertyKeyMap）- holo数据查询代码目录：internal/logic/mq_sm_data_holo- 创建新媒体数据类文件（参考快手：mq_sm_data_holo_kuaishou.go）- 实现接口方法：GetData、buildSql、buildWhere、buildPropertyDimension等- 添加适当的属性指标和媒体指标- deployments_sygn/app.ini- cronjobs下生成新媒体的定时任务，包含smDataPageWith（媒体层级）Job、smKpiWith（媒体层级）Job等- 参考现有媒体的定时任务配置，设置合适的执行频率和参数- 确保在mq_sm_data_holo.go中注册新媒体的处理器## 场景4：生成sm_target指标导入脚本 - 确认媒体类型：支持头条(toutiao，media_id=1)、快手(kuaishou，media_id=5)、腾讯(tengxun，media_id=-4) - 源文件：`target.xlsx`， 表头格式为[指标名称、指标key、归属第一层级、归属第二层级、指标类型] - 目标表：sm_target，表结构为："`sm_target` (`ID` int(10) unsigned NOT NULL AUTO_INCREMENT, `NAME` varchar(50) NOT NULL COMMENT '指标名称', `TARGET_KEY` varchar(200) NOT NULL COMMENT '指标key', `MEDIA_ID` smallint(6) NOT NULL DEFAULT '0' COMMENT '指标归属媒体', `MONITOR_TYPES` varchar(64) NOT NULL COMMENT '指标归属监控类型', `TYPE` varchar(32) NOT NULL COMMENT '指标类型', `UNIT` varchar(32) NOT NULL DEFAULT '' COMMENT '指标单位', `INDEX_SHOW` smallint(6) NOT NULL DEFAULT '0' COMMENT '排序', `PARENT_ID` int(10) DEFAULT NULL COMMENT 'Target父ID', `FORMULA` varchar(255) DEFAULT NULL COMMENT 'TARGET_KEY计算公式', `STATUS` tinyint(4) NOT NULL DEFAULT '1' COMMENT '是否可用', `TOP_PARENT_ID` int(11) NOT NULL DEFAULT '0' COMMENT '顶级指标ID', `TIPS` varchar(500) NOT NULL DEFAULT '' COMMENT '指标说明', `IS_DIR` tinyint(4) NOT NULL DEFAULT '0' COMMENT '是否为目录结构', PRIMARY KEY (`ID`) USING BTREE, UNIQUE KEY `uniq_sm_target_media_key_monitor_type` (`MEDIA_ID`,`MONITOR_TYPES`,`TARGET_KEY`) ) ENGINE=InnoDB AUTO_INCREMENT=1323 DEFAULT CHARSET=utf8 COMMENT='新监控通用指标表'" - 说明： - 源文件字段[归属第一层级]枚举值为：5 => 属性指标，6 => 媒体数据指标，7 => 统计数据指标 - 源文件字段[归属第二层级]枚举值从目标表获取，参考sql："select id, name from sm_target where parent_id = '{归属第一层级}枚举值key'" - 指标类型：数值（TYPE=STRING）、百分比（TYPE=STRING、UNIT=%）# 注意事项 - 先一步一步思考再生成代码 - 尽量复用代码，参考现有媒体的实现方式 - 遵循现有代码结构和命名规范 - 做好错误处理和日志记录 - 保持与现有功能的兼容性 - 添加必要的注释说明 - 判断开发场景，使用对应的开发场景步骤执行 - 有提供Doc的前提下，从Doc查询对应的指标和属性 - 对于数值型指标，注意处理精度和格式化 - 对于字符串型指标，注意处理空值和多值聚合 - 测试新增功能，确保不影响现有功能
```

04

# 总结

通过合理使用 Cursor，我们可以显著提高广告监控业务场景的开发效率，同时保证代码质量和一致性。这不仅减少了开发人员的工作负担，也为后续的维护和扩展提供了良好的基础。主要体现在：

1. 效率提升

* 减少重复性代码编写（如可以用生成可复用的定时任务）
* 降低人为错误
* 标准化开发流程

2. 质量保证

* 统一的代码风格
* 自动化的错误处理
* 完整的注释说明

3. 最佳实践

* 制定清晰的规则文件
* 建立完整的模板库
* 保持规则文件的及时更新
* 定期优化自动化流程