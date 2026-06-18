---
title: ESLint v10.0.0-beta.0 发布
author: eslint-cn
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzk2NDMxNDQ4Mg==&mid=2247483668&idx=1&sn=9ec9b3ae1c0384a1fb8b7d366004ee64&chksm=c511d91734a0ace72aea4af4df4baef62096386c299a870a216777a404b74d614d93c20c2de9&mpshare=1&scene=24&srcid=1215oulmO8wdntvTOlvlW6Ee&sharer_shareinfo=56a34ab577767eb5e9c8853804101f9b&sharer_shareinfo_first=56a34ab577767eb5e9c8853804101f9b#rd
---

# ESLint v10.0.0-beta.0 发布

我们刚刚发布了 ESLint v10.0.0-beta.0，这是 ESLint 的一次重大版本升级。本次发布新增了一些功能，并修复了之前版本中的若干错误。此版本包含一些破坏性变更，请仔细阅读以下内容。

## 亮点

此版本 ESLint 尚未准备好用于生产环境，旨在最终版发布前收集社区反馈。如果您遇到任何问题或有任何反馈，请在我们的 GitHub 仓库创建 issue。

此版本的大部分亮点都是破坏性变更，并在迁移指南中有详细讨论。以下是重大变更的摘要。（不太重要的变更包含在迁移指南中。）

此预发布版本的 ESLint 有单独的文档部分。

## RuleTester 断言选项

RuleTester#run() 方法现在支持**断言选项**，特别是 requireMessage 和 requireLocation，让开发者可以在规则测试中强制执行更严格的要求。

**requireMessage**

* 确保每个测试用例包含消息检查
* 接受值：

+ • true：必须使用对象数组作为 errors
+ • "message"：必须仅使用 message 检查
+ • "messageId"：必须仅使用 messageId 检查

* 目的：防止测试在没有验证实际消息的情况下通过

**requireLocation**

* 确保每个测试用例包含位置检查
* 接受值：true
* 要求 errors 数组中的每个对象包含 line 和 column
* 目的：保证测试验证错误的位置

使用示例：

```
1

2

3

4

5

6

7

8

9

10

11

ruleTester.run("my-rule", rule, {  
  valid: [{ code: "var foo = true;" }],  
  invalid: [{  
    code: "var invalidVariable = true;",  
 errors:[{message:"Unexpected invalid variable.", line: 1, column: 5 }]  
  }],  
  assertionOptions: {  
    requireMessage: true,  
    requireLocation: true  
  }});
```

## JSX 引用跟踪

ESLint v10.0.0 现在跟踪 JSX 引用，实现了对 JSX 元素的正确作用域分析。

之前，JSX 标识符不被作为引用跟踪，这可能导致依赖作用域信息的规则得出不正确的结果。

从 v10.0.0 开始，JSX 元素被视为对作用域中变量的普通引用。这消除了令人困惑的误报/漏报，使 JSX 处理与开发者的期望保持一致。

如果您的代码库包含 JSX，您可能会开始看到新的 linting 报告。要修复它们，请根据需要更新代码或调整规则配置。

## 格式化器上下文中的 color 属性

当在命令行指定 --color 或 --no-color 选项时，ESLint 会在传递给格式化器的上下文对象上设置一个额外的 color 属性。

对于默认的 "stylish" 格式化器，在决定是否应用颜色时，--color 或 --no-color 选项现在优先于 Node.js 检查的其他规则。

## 安装

由于这是一个预发布版本，npm 不会自动升级。您必须在安装时指定 next 标签：

```
1

npm i eslint@next --save-dev
```

或直接指定版本：

```
1

npm i eslint@10.0.0-beta.0 --save-dev
```

## 迁移指南

由于变更很多，我们创建了迁移指南，详细描述了破坏性变更以及您应采取的措施。

## 破坏性变更

* • 用 styleText 替换 chalk 并向 ResultsMeta 添加 color
* • 启用 JSX 引用跟踪

## 新功能

* • 添加错误断言选项

## Bug 修复

* • 修正 loadESLint() 和 shouldUseFlatConfig() 的类型定义
* • 修正 RuleTester 类型定义
* • 严格检查已移除的格式化器
* • 修正 no-restricted-import 消息

## 文档更新

* • 更新 README
* • 阐明 no-unused-vars 的 "local" 选项
* • 改进文档站点 README 的清晰度、语法和措辞
* • 向规则元数据文档添加 messages 属性
* • 从规则文档中移除 Examples 标题

## 构建相关

* • 向 knip 添加 .scss 文件条目

## 杂项更新

* • 创建包管理器测试
* • 简化 JSDoc 注释检测逻辑
* • 将 @eslint-community/regexpp 更新至 4.12.2
* • 将依赖 @eslint/eslintrc 更新至 ^3.3.3

**原文链接：** https://eslint.org/blog/2025/12/eslint-v10-0-0-beta-0-released/

（注：微信公众号中链接无法直接点击，请读者手动复制查看）