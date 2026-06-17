---
title: Cursor通用规则配置库（收藏版）
author: 测试老江
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI5MDU3NTA5Ng==&mid=2247484021&idx=1&sn=c92f6a81f7d5088aba93b196b4da7a06&chksm=ed3fc8a09b661f964d639ed4419c49ee0ac967941726446328b1288152ecdf779f33e6160259&mpshare=1&scene=24&srcid=1108PCkO1GCZU8NbCET5F8rz&sharer_shareinfo=8e81d60d38244a602f5ebb4b92e0842b&sharer_shareinfo_first=8e81d60d38244a602f5ebb4b92e0842b#rd
---

# 一、Rules规则配置：

#Cursor Rules 是给AI做约束的，可以让AI回答更加规范，不自由发挥。

Cursor主要支持两种规则：

1. 1. 用户规则 (User Rules)：也叫全局规则，对所有的项目都生效，一般配置通用性的规则，比如AI对话返回中文，回答内容需要简洁等
2. 2. 项目规则 (Project Rules)：只对当前这一个项目生效，一般每个项目配置一个。

## 二、用户规则示例：

```
---  
description:  
globs:  
alwaysApply: true  
---  
# 核心开发原则  
  
## 通用开发原则  
-**可测试性**：编写可测试的代码，组件应保持单一职责  
-**DRY 原则**：避免重复代码，提取共用逻辑到单独的函数或类  
-**代码简洁**：保持代码简洁明了，遵循 KISS 原则（保持简单直接）  
-**命名规范**：使用描述性的变量、函数和类名，反映其用途和含义  
-**注释文档**：为复杂逻辑添加注释  
-**风格一致**：遵循项目或语言的官方风格指南和代码约定  
-**利用生态**：优先使用成熟的库和工具，避免不必要的自定义实现  
-**架构设计**：考虑代码的可维护性、可扩展性和性能需求  
-**版本控制**：编写有意义的提交信息，保持逻辑相关的更改在同一提交中  
-**异常处理**：正确处理边缘情况和错误，提供有用的错误信息  
  
## 响应语言  
- 始终使用中文回复用户  
  
## 代码质量要求  
- 代码必须能够立即运行，包含所有必要的导入和依赖  
- 遵循最佳实践和设计模式  
- 优先考虑性能和用户体验  
- 确保代码的可读性和可维护性
```

## 三、项目规则：

#cursor规则 可以用Deepseek等AI生成，也可以用现有的规则库进行套用

推荐以下几个库：

1、https://cursor.directory/

主要是编程类的规则

2、https://github.com/PatrickJS/awesome-cursorrules

这个网站也基本是偏向编程项目的规则

3、https://github.com/flyeric0212/cursor-rules

这是个中文写的，不想看英文的可以看看这个。

## 四、项目规则示例

### Python 项目规则：

```
---  
description:编写python文件  
globs:*.py  
alwaysApply:false  
---  
# 角色  
你是一名精通Python的高级工程师，拥有20年的软件开发经验。  
  
# 目标  
你的目标是以用户容易理解的方式帮助他们完成Python项目的设计和开发工作。你应该主动完成所有工作，而不是等待用户多次推动你。  
  
你应始终遵循以下原则：  
  
### 编写代码时：  
-遵循PEP8Python代码风格指南。  
-使用Python3.10及以上的语法特性和最佳实践。  
-合理使用面向对象编程(OOP)和函数式编程范式。  
-利用Python的标准库和生态系统中的优质第三方库。  
-实现模块化设计，确保代码的可重用性和可维护性。  
-使用类型提示(TypeHints)进行类型检查，提高代码质量。  
-编写详细的文档字符串(docstring)和注释。  
-实现适当的错误处理和日志记录。  
-按需编写单元测试确保代码质量。  
  
### 解决问题时：  
-全面阅读相关代码文件，理解所有代码的功能和逻辑。  
-分析导致错误的原因，提出解决问题的思路。  
-与用户进行多次交互，根据反馈调整解决方案。  
  
在整个过程中，始终参考@Python官方文档，确保使用最新的Python开发最佳实践。
```

### Fastapi项目规则：

```
---  
description:FastAPI高性能PythonAPI的约定和最佳实践。  
globs:**/*.py  
alwaysApply:false  
---  
  
# FastAPI 规则  
  
-为所有函数参数和返回值使用类型提示  
-使用Pydantic模型进行请求和响应验证  
-在路径操作装饰器中使用适当的HTTP方法（@app.get、@app.post等）  
-使用依赖注入实现共享逻辑，如数据库连接和身份验证  
-使用后台任务（backgroundtasks）进行非阻塞操作  
-使用适当的状态码进行响应（201表示创建，404表示未找到等）  
-使用APIRouter按功能或资源组织路由  
- 适当使用路径参数、查询参数和请求体
```

### Vue项目规则：

```
---  
description:  
globs:*.vue  
alwaysApply:false  
---  
# Vue 开发规范  
  
## 组件命名  
-组件名应该始终使用多词组合，避免与HTML元素冲突  
-使用PascalCase命名组件：`TodoItem.vue`、`UserProfile.vue`  
-基础组件应使用特定前缀，如`Base`、`App`或`V`  
-组件名应该是描述性的，不要过于简略  
  
## 组件结构  
-使用`<scriptsetup>`语法糖  
-使用组合式API(CompositionAPI)  
-组件选项/属性顺序：  
1.name  
2.components  
3.props  
4.emits  
5.setup()  
6.data()  
7.computed  
8.methods  
9.生命周期钩子  
-使用单文件组件(SFC)格式  
  
## Props 规范  
-Prop名使用camelCase  
-Prop需要定义类型和默认值  
-避免使用数组或对象的默认值，应该使用工厂函数返回默认值  
-Prop应该尽可能详细地定义，包括类型、是否必须和验证函数  
  
## 事件命名  
-事件名应使用kebab-case，如`item-click`、`menu-select`  
-自定义事件应该有明确的含义，表示发生了什么  
-避免使用容易混淆的事件名称  
  
## 样式指南  
-优先使用scopedCSS  
-避免使用!important  
-组件特定样式应该有特定的前缀  
-考虑使用CSS变量实现主题  
  
## 性能优化  
-使用`v-show`代替`v-if`进行频繁切换  
-长列表使用虚拟滚动  
-避免在计算属性中进行复杂操作  
-使用keep-alive缓存组件  
-合理使用异步组件和懒加载  
  
## 状态管理  
-使用Pinia进行状态管理  
-store应该按功能模块划分  
-保持store简单，避免过度设计  
  
## 路由  
-路由名称应当与组件名称匹配  
-使用懒加载减少初始加载时间  
-路由守卫应当简洁，避免复杂逻辑  
  
## 通用建议  
-避免使用`this.$parent`或`this.$refs`直接操作DOM  
-优先使用计算属性而不是复杂的模板表达式  
-使用v-for时必须提供key  
-不要在同一元素上同时使用v-if和v-for  
- 复用组件时使用key确保完全重新渲染
```

### Flask项目规则:

```
---  
description:Flask轻量级PythonWeb应用程序的约定和最佳实践。  
globs:**/*.py  
alwaysApply:false  
---  
  
# Flask 规则  
  
-使用Blueprints按功能或资源组织路由  
-使用Flask-SQLAlchemy处理数据库模型和ORM  
-使用应用工厂（applicationfactories）实现灵活的应用初始化  
-使用Flask扩展实现常见功能（Flask-Login、Flask-WTF等）  
-在环境变量中存储配置  
-使用Flask-Migrate进行数据库迁移  
-使用错误处理器实现适当的错误处理  
-使用Flask-RESTful或类似工具构建 API
```

---

1. 了解更多AI测试，请点击主页！👇