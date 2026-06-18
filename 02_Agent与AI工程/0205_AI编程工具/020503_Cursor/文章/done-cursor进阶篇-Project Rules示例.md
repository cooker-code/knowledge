> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020503_Cursor/020503_核心知识点/Cursor工程使用与上下文边界|Cursor工程使用与上下文边界]]
---
title: cursor进阶篇-Project Rules示例
author: AIGC小牛独立开发日记
date:
url: https://mp.weixin.qq.com/s?__biz=MzkwNzg4MjgzNQ==&mid=2247484192&idx=1&sn=f99523747775c9fc8c4677751235b165&chksm=c1d33e518eb39fc8c733f154abbd30e7afe0b98d3dc7ed45164c8c2ad10c4b3c3d824ef62aeb&mpshare=1&scene=24&srcid=11147jfXkDiCyS6NabxbEWuJ&sharer_shareinfo=84f10901ce0ef4b373bc76b3939f4979&sharer_shareinfo_first=84f10901ce0ef4b373bc76b3939f4979#rd
---

本文综合网络上的共享文档，搜集并整理了以下Project Rules示例，请欣赏。

# 一、通用规范

如果是通用规则，在Cursor中的Rule type一般选择**Always、Agent Requsted**或**Manual**，如果是编程语言规则或框架规则，对应的Rule type一般选择Auto Attached。这里给大家复习下这四种Rule type：

* Always：这个规则会自动附加到每个聊天和命令上
* Auto Attahced：文件模式匹配，能应用到特定的文件或文件夹上
* Agent Requested：描述这个规则有助于完成什么任务（Agent会根据描述自己来调用）
* Manual：这个规则需要被手动@ 才能被包含在上下文

## 1.1 项目通用开发规范

Rule type：Always

```
```
# 项目通用开发规范

## 1. 项目结构规范
### 目录结构
```
project_name/
├── docs/                 # 项目文档
├── src/                  # 源代码
│   ├── core/            # 核心功能
│   ├── utils/           # 工具函数
│   └── tests/           # 测试文件
├── scripts/             # 脚本文件
├── config/              # 配置文件
├── .github/             # CI/CD 配置
├── package.json         # 项目配置
└── README.md           # 项目说明
```
### 组织原则
- 保持项目结构清晰，遵循模块化原则
- 相关功能应放在同一目录下
- 使用适当的目录命名，反映其包含内容

## 2. 代码规范
### 命名规范
```
类名：PascalCase（大驼峰）
函数名：camelCase（小驼峰）或 snake_case
常量：UPPER_SNAKE_CASE
变量：camelCase 或 snake_case
```
### 代码质量原则
- 遵循 SOLID 设计原则
- 避免代码重复（DRY原则）
- 保持代码简洁、清晰、易读
- 考虑代码的可维护性和可扩展性

### 异常处理
- 合理使用异常处理机制
- 提供清晰的错误信息
- 记录必要的错误日志
- 优雅处理边界情况

## 3. 文档规范
### 代码注释
```
/**
 * 函数功能说明
 * 
 * @param {参数类型} 参数名 - 参数说明
 * @returns {返回类型} 返回值说明
 */
```
### 项目文档
- 及时更新 README 和技术文档
- 使用中文编写文档
- 包含必要的安装和使用说明
- 记录重要的架构决策

### 国际化准备
- 代码注释使用中文
- 错误信息和日志使用中文描述
- 预留国际化支持的接口

## 4. 开发环境
### 依赖管理
- 使用项目对应语言的包管理工具
- 锁定依赖版本，确保构建稳定性
- 定期更新依赖，修复安全隐患
- 优先使用现有库和工具，避免重新发明轮子

## 5. 版本控制规范
### 基础配置
- 使用 Git 作为版本控制系统
- 设置合适的 .gitignore 文件
- 保护主分支，实施分支权限控制

### 分支管理
- 采用规范的分支开发流程（如 Git Flow 或 Trunk Based Development）
- 主分支保持稳定，开发在特性分支进行
- 定期清理过期分支

### 提交规范
- 使用清晰的 commit 信息
- 每个提交专注于单一功能或修复
- 合理使用 tag 标记版本

### 代码审查
- 所有代码变更必须经过审查
- 遵循代码审查清单
- 及时响应审查意见
- 确保代码质量和一致性

## 6. 测试规范
### 测试原则
- 遵循测试金字塔原则
- 保持测试代码的整洁和可维护性
- 避免测试代码重复
- 关注测试覆盖率

### 测试类型
- 单元测试：测试独立功能单元
- 集成测试：测试模块间交互
- 端到端测试：测试完整业务流程
- 性能测试：关键功能性能验证

## 7. 安全规范
- 使用环境变量存储敏感配置
- 避免在代码中硬编码敏感信息
- 定期更新依赖包版本
- 实施安全扫描
```
```

由于项目通用开发规范的颗粒度还是比较大，因此我们可以进一步细分，比如文档撰写规范、代码风格规范、前端开发规范、后端开发规范、Git版本管理规范等等。

## 1.2 文档撰写规范

Rules Types：Always

```
```
# 文档撰写规范

## 1. 文件结构规范
### 必需文件
项目中必须包含以下Markdown文档：
- `README.md` - 项目主文档
- `CHANGELOG.md` - 版本更新日志
- 其他相关的 `.md` 文档

### README.md 规范
1. **文档结构**
   - 保持文档结构清晰，使用适当的Markdown标记
   - **重要**: 每次修改必须保留 README.md 中的二级目录"Cursor 历史下载链接"部分，不要进行删除

2. **必需章节**
   README.md 必须包含以下部分：
   - 项目简介
   - 安装说明
   - 使用方法
   - 贡献指南（如适用）
   - 许可证信息
   - Cursor 历史下载链接（必须保留）

### CHANGELOG.md 规范
1. **版本号格式**
   使用语义化版本号（如：v1.0.0）

2. **更新内容格式**

   ## v1.0.0

   - 新增功能：[功能描述]
   - 修复bug：[bug描述]


3. **分类标签**
   使用以下标签分类更新内容：
   - 新增功能
   - 修复bug
   - 改进优化
   - 破坏性变更

## 2. 文档更新原则
1. **同步更新**
   - 保持文档与代码同步更新
   - 文档变更必须与代码变更一起提交

2. **语言规范**
   - 使用简洁明了的语言
   - 避免使用晦涩难懂的专业术语
   - 必要时提供术语解释

3. **示例规范**
   - 提供足够的示例和说明
   - 示例代码必须经过测试验证
   - 示例应当简单易懂，突出重点

4. **格式统一**
   - 确保文档格式一致
   - 使用统一的标题层级
   - 保持列表格式统一
   - 代码块使用适当的语言标记

## 3. Markdown 格式规范
1. **标题使用**
   - 一级标题 (#) 用于文档标题
   - 二级标题 (##) 用于主要章节
   - 三级及以下标题用于子章节
   - 标题层级不应超过四级

2. **列表格式**
   - 无序列表使用 `-` 
   - 有序列表使用 `1.` 
   - 保持列表项缩进一致

3. **代码块**

   - 使用三个反引号包裹代码
   - 指定代码语言类型
   - 保持代码格式化 4. **强调语法**
   - 使用 `**文本**` 进行加粗强调
   - 使用 `*文本*` 进行斜体标记
   - 重要信息使用加粗标记

## 4. 文档维护
1. **定期审查**
   - 定期检查文档的准确性
   - 更新过时的信息
   - 补充必要的新内容

2. **版本控制**
   - 文档变更要有版本记录
   - 重大变更需要在CHANGELOG中记录
   - 保留历史版本信息

## 4. 贡献指南
1. **文档贡献**
   - 遵循现有的文档格式
   - 提供清晰的变更说明
   - 确保文档更新经过审核

2. **质量控制**
   - 提交前进行拼写和语法检查
   - 确保链接可用
   - 验证示例代码的正确性 
```
```

## 1.3 代码风格规范

Rules Types：Always

```
```
# 代码风格规范

## **1. 基本原则**
- 保持代码简洁、一致且可读
- 使用统一的缩进和格式
- 避免过长的函数和文件
- 遵循"干净代码"原则

## 2. 格式化
### 缩进和空格
- 使用2个或4个空格进行缩进（项目内保持一致）
- 不使用制表符（Tab）
- 运算符两侧添加空格
- 逗号后添加空格

### 换行
- 每行代码长度不超过80-120个字符（根据团队约定）
- 链式调用应换行并对齐
- 函数参数过多时进行换行
- 长表达式应适当换行增加可读性

### 括号和空行
- 控制语句的左花括号与声明在同一行
- 添加适当的空行分隔代码逻辑段
- 保持括号匹配和对齐
- 避免多余的括号和空行

## 3. 命名约定
### 通用原则
- 名称应清晰表达意图，避免缩写（除了广泛接受的缩写）
- 名称长度应与作用域大小成正比
- 避免使用单字母变量名（除了临时变量如循环计数器）
- 避免使用无意义的名称（如 temp, foo, bar）

### 命名风格
- 类/接口：使用大驼峰命名法（PascalCase）
  class UserService {}
  interface DatabaseConnection {}

- 变量/函数：使用小驼峰命名法（camelCase）或蛇形命名法（snake_case）（根据语言惯例）
  let userData = {};
  function calculateTotal() {}

- 常量：使用大写蛇形命名法（UPPER_SNAKE_CASE）
  const MAX_RETRY_COUNT = 3;
  const API_BASE_URL = 'https://api.example.com';

## 4. 注释规范
### 代码注释
- 使用 JSDoc, JavaDoc 或语言对应的注释格式
- 每个函数、类和复杂代码块添加注释
- 注释应解释"为什么"而不仅仅是"做什么"
- 及时更新注释，避免过时

### 行内注释
- 复杂逻辑处添加行内注释
- 保持简洁，避免冗余
- 使用统一的注释前缀标记特殊情况：

## 代码组织
### 文件结构
- 一个文件只包含一个主要类/组件/模块
- 相关代码组织在一起
- 按照功能进行文件分组，而不是按类型
- 导入顺序：标准库 > 第三方库 > 项目内模块

### 代码块组织
- 相关功能的代码应放在一起
- 从抽象到具体，从高层到低层
- 公有方法在前，私有方法在后
- 按照逻辑分组代码，而不是按字母顺序

## 工具支持
- 使用 ESLint, Prettier, EditorConfig 等工具强制代码风格
- 提供适用于常见编辑器的配置文件
- 在 CI 流程中检查代码风格
- 使用 Git Hooks (如 husky) 在提交前自动格式化代码

## 最佳实践
- 定期代码审查确保风格一致性
- 风格问题不应阻碍开发进度，但应在代码审查中解决
- 风格规范可随项目发展进行调整，但需要达成共识 
```
```

## 1.4 前端开发规范

Rules Types：Agent Requested

这份前端开发规范，其实包含了多个编程语言和框架的规范在里面，当然，大家也可以使用对应编程语言和框架的规范，因为后者还可以更精准地定义。

```
```
# 前端开发规范

## 1. 基本原则
- **一致性**：全项目采用统一的代码风格和最佳实践
- **可维护性**：代码组织应便于阅读、理解和维护
- **性能优先**：重视代码质量和性能优化
- **响应式设计**：确保在各种设备上提供良好的用户体验
- **可访问性**：遵循无障碍设计原则
- **可测试性**：编写便于测试的代码

## 2. 项目结构与命名规范
### 项目目录结构
```
src/
├── assets/          # 静态资源（图片、字体等）
├── components/      # 通用组件
│   ├── common/      # 跨业务的通用组件
│   └── business/    # 业务相关组件
├── hooks/           # 自定义React Hooks
├── layouts/         # 页面布局组件
├── pages/           # 页面组件
├── services/        # API请求服务
├── store/           # 状态管理
├── styles/          # 全局样式
├── types/           # TypeScript类型定义
├── utils/           # 工具函数
└── App.tsx          # 应用入口组件
```

### 命名规范
1. **文件命名**：
   - React组件文件采用大驼峰命名法（PascalCase）：`Button.tsx`、`UserProfile.tsx`
   - 工具函数、hooks等非组件文件采用小驼峰命名法（camelCase）：`formatDate.ts`、`useWindowSize.ts`
   - 样式文件与组件同名：`Button.module.css`、`UserProfile.scss`
   - 测试文件与被测文件同名，后跟`.test`或`.spec`：`Button.test.tsx`

2. **组件命名**：
   - 使用大驼峰命名法
   - 名称应清晰表明组件的用途
   - 避免使用缩写（除非是广为接受的缩写如URL）

3. **变量命名**：
   - 使用小驼峰命名法
   - 布尔值变量使用`is`、`has`、`can`等前缀
   - 常量使用全大写，下划线分隔：`MAX_COUNT`、`API_URL`

4. **CSS类命名**：
   - 采用BEM（Block Element Modifier）命名规范
   - 格式：`block__element--modifier`
   - 例如：`card__title--active`

## 3. HTML规范
1. **语义化标签**：
   - 使用HTML5语义化标签（`<header>`、`<footer>`、`<section>`等）
   - 确保标签使用符合其语义

2. **无障碍性**：
   - 使用适当的ARIA属性增强可访问性
   - 确保表单元素有关联的label
   - 提供适当的alt文本给图片
   - 保持正确的标题层级（h1-h6）

3. **最佳实践**：
   - 减少不必要的嵌套层级
   - 避免过多的内联样式
   - 确保HTML结构清晰可读

## 4. CSS规范
1. **CSS组织**：
   - 优先使用CSS模块或styled-components等CSS-in-JS方案
   - 避免全局样式污染
   - 使用预处理器（如SASS）组织复杂样式

2. **样式编写**：
   - 使用类选择器，避免使用ID选择器
   - 选择器避免嵌套过深（不超过3级）
   - 使用简写属性（如`margin`、`padding`等）

3. **响应式设计**：
   - 采用移动优先（Mobile First）的响应式设计方法
   - 使用相对单位（rem, em, vh, vw）而非绝对单位（px）
   - 使用媒体查询适配不同设备

4. **性能优化**：
   - 避免使用`*`选择器
   - 减少使用会触发重排（reflow）的属性
   - 适当使用CSS硬件加速（`transform`、`opacity`等）

## 5. JavaScript/TypeScript规范
1. **基本规则**：
   - 使用ES6+语法标准
   - 优先使用const，其次使用let，避免使用var
   - 使用箭头函数表示回调函数
   - 优先使用模板字符串拼接字符串

2. **类型规范（TypeScript）**：
   - 新代码必须使用TypeScript编写
   - 避免使用`any`类型，必要时使用`unknown`
   - 为函数参数和返回值添加类型注解
   - 使用接口（interface）定义对象结构
   - 使用类型别名（type）定义联合类型或交叉类型

3. **注释规范**：
   - 使用JSDoc格式为函数、类和复杂逻辑添加注释
   - 注释应解释"为什么"而不仅仅是"做了什么"
   - 删除无用或过时的注释

4. **异步编程**：
   - 使用async/await处理异步操作，避免回调地狱
   - 正确处理Promise的错误捕获
   - 避免嵌套Promise

## 6. React开发规范
1. **组件设计**：
   - 遵循单一职责原则，每个组件只做一件事
   - 按功能拆分组件，避免过大的组件
   - 使用函数组件和React Hooks，避免使用类组件
   - 提取可复用的逻辑到自定义Hooks中

2. **状态管理**：
   - 局部状态使用useState或useReducer
   - 跨组件状态使用Context API或状态管理库（Redux/Zustand等）
   - 避免过深的props传递（props drilling）

3. **组件通信**：
   - 父子组件通过props传递数据
   - 子父组件通过回调函数通信
   - 兄弟组件或远亲组件通过状态提升或全局状态管理
   - 使用自定义事件机制处理复杂通信场景

4. **性能优化**：
   - 使用React.memo避免不必要的重渲染
   - 使用useMemo缓存计算结果
   - 使用useCallback缓存回调函数
   - 合理使用key属性优化列表渲染
   - 使用React.lazy和Suspense实现代码分割
   - 避免在渲染函数中创建新函数或对象

5. **表单处理**：
   - 使用受控组件处理表单
   - 复杂表单考虑使用表单库（如Formik、React Hook Form）
   - 实现适当的表单验证和错误反馈

6. **Hook规则**：
   - 遵循Hook的使用规则
   - 只在顶层调用Hook
   - 只在React函数组件或自定义Hook中调用Hook
   - 确保useEffect的依赖数组正确设置

## 7. 状态管理规范
1. **Redux使用规则**（如适用）：
   - 使用Redux Toolkit简化Redux代码
   - 按领域（feature）组织Store
   - 使用标准命名约定（actions, reducers, selectors）
   - 避免在reducer中进行复杂计算，使用selector处理
   - 遵循不可变状态更新模式

2. **Zustand使用规则**（如适用）：
   - 创建多个小型store而非一个大store
   - 使用selector优化渲染性能
   - 合理组织store的结构和API

3. **通用规则**：
   - 保持状态扁平化，避免深层嵌套
   - 一致使用不可变更新模式
   - 使用选择器（selectors）访问状态
   - 异步操作使用中间件（如redux-thunk、redux-saga）

## 8. API请求规范
1. **请求封装**：
   - 使用Axios或Fetch API封装请求
   - 创建统一的请求实例，配置拦截器
   - 处理通用错误和加载状态

2. **数据获取**：
   - 使用React Query或SWR等工具管理服务器状态
   - 实现数据预取、缓存和失效处理
   - 处理乐观更新（Optimistic Updates）

3. **错误处理**：
   - 实现全局错误处理机制
   - 为不同错误提供友好的用户反馈
   - 记录错误日志以便调试

## 9. 测试规范
1. **单元测试**：
   - 使用Jest作为测试框架
   - 为工具函数、自定义hooks和小型组件编写单元测试
   - 遵循AAA模式（Arrange-Act-Assert）

2. **组件测试**：
   - 使用React Testing Library测试组件
   - 测试用户交互而非实现细节
   - 为关键用户流程编写测试

3. **端到端测试**：
   - 使用Cypress或Playwright进行E2E测试
   - 为关键用户流程编写端到端测试
   - 模拟真实用户操作测试完整功能

4. **覆盖率目标**：
   - 业务逻辑单元测试覆盖率 >= 80%
   - UI组件测试覆盖率 >= 70%
   - 关键用户流程必须有端到端测试覆盖

## 10. 性能优化规范
1. **加载性能**：
   - 实现代码分割（Code Splitting）
   - 延迟加载非关键组件和资源
   - 对图片进行优化（压缩、合适尺寸、使用WebP格式）
   - 使用合适的预加载策略（preload、prefetch）

2. **运行时性能**：
   - 避免不必要的渲染
   - 优化长列表渲染（虚拟列表）
   - 防抖和节流处理频繁事件
   - 优化动画性能（使用CSS动画或Web Animations API）

3. **性能监控**：
   - 使用Lighthouse或WebPageTest进行性能评估
   - 监控核心Web指标（Core Web Vitals）
   - 实施性能预算（Performance Budget）

## 11. 构建与部署规范
1. **构建优化**：
   - 使用webpack或vite进行构建
   - 配置合理的代码分割策略
   - 启用资源压缩和Tree Shaking
   - 实施现代和传统浏览器的差异化构建

2. **部署流程**：
   - 实施CI/CD自动化部署
   - 使用环境变量区分开发、测试和生产环境
   - 实现静态资源的长期缓存策略
   - 配置适当的缓存控制和CORS策略

## 12. 代码质量与审查
1. **代码检查**：
   - 使用ESLint进行代码检查
   - 使用Prettier统一代码格式
   - 集成TypeScript类型检查
   - 使用husky和lint-staged实现提交前检查

2. **代码审查**：
   - 所有代码变更通过Pull Request提交
   - 至少一名团队成员审查通过后方可合并
   - 关注代码质量、性能和安全性
   - 使用自动化工具辅助代码审查

## 13. 依赖管理
1. **依赖选择**：
   - 选择社区活跃、维护良好的依赖
   - 评估依赖的大小、性能和兼容性
   - 避免引入过多小型依赖

2. **版本控制**：
   - 锁定依赖版本（使用package-lock.json或yarn.lock）
   - 定期更新依赖并测试兼容性
   - 使用语义化版本控制依赖更新范围

3. **安全措施**：
   - 定期运行`npm audit`检查安全漏洞
   - 配置自动化依赖更新（如Dependabot）
   - 评估并修复关键安全问题

## 14. 国际化与本地化
1. **i18n策略**：
   - 使用react-i18next或react-intl实现国际化
   - 所有用户可见文本使用翻译键代替硬编码文本
   - 考虑文本长度变化对布局的影响

2. **最佳实践**：
   - 将翻译文件组织为按语言和命名空间分类
   - 实现语言切换和持久化用户语言偏好
   - 考虑日期、数字和货币的格式化

## 15. 安全规范
1. **安全最佳实践**：
   - 实现适当的认证和授权机制
   - 防范XSS攻击（使用内容安全策略CSP）
   - 防范CSRF攻击（使用CSRF令牌）
   - 使用HTTPS进行安全通信

2. **敏感数据处理**：
   - 避免在前端存储敏感信息
   - 使用HttpOnly Cookie存储会话信息
   - 实现合适的数据脱敏策略

3. **第三方集成**：
   - 评估第三方脚本的安全性
   - 使用子资源完整性（SRI）校验第三方资源
   - 限制第三方脚本的权限
```
```

精简版的前端开发规范：

```
```
# 前端开发规范
## 1. 基本原则
- **一致性**：统一代码风格和最佳实践
- **可维护性**：易于阅读和维护的代码组织
- **性能优先**：注重代码质量和性能
- **响应式设计**：适配各种设备
- **可访问性**：遵循无障碍设计
- **可测试性**：便于测试的代码结构

## 2. 项目结构
src/
├── assets/ # 静态资源
├── components/ # 组件（common/通用、business/业务）
├── hooks/ # 自定义Hooks
├── pages/ # 页面组件
├── services/ # API请求
├── store/ # 状态管理
├── styles/ # 全局样式
├── types/ # TypeScript类型
├── utils/ # 工具函数
└── App.tsx # 入口组件

## 3. 命名规范
- **组件文件**：大驼峰（PascalCase）- `Button.tsx`
- **工具/Hooks**：小驼峰（camelCase）- `formatDate.ts`
- **变量命名**：小驼峰，布尔值使用`is/has/can`前缀
- **常量**：全大写下划线分隔 - `MAX_COUNT`
- **CSS类**：BEM命名法 - `block__element--modifier`

## 4. HTML最佳实践
- 使用语义化标签（`header`, `section`等）
- 确保无障碍性（ARIA属性，label关联，alt文本）
- 减少嵌套，避免过多内联样式

## 5. CSS最佳实践
- 使用CSS模块或CSS-in-JS避免全局污染
- 选择器不超过3级嵌套
- 移动优先响应式设计，使用相对单位
- 避免`*`选择器，减少重排属性，适当使用硬件加速

## 6. JavaScript/TypeScript规范
- 使用ES6+语法，优先const/let，避免var
- 避免`any`类型，为函数添加类型注解
- 使用JSDoc注释解释"为什么"而非仅"做了什么"
- 使用async/await处理异步，避免嵌套Promise

## 7. React开发要点
- **组件设计**：单一职责，函数组件+Hooks优先
- **状态管理**：局部用useState/useReducer，全局用Context/Redux/Zustand
- **性能优化**：
  - 使用memo/useMemo/useCallback避免不必要重渲染
  - 使用key优化列表
  - 代码分割（React.lazy + Suspense）
- **Hook规则**：只在顶层和函数组件/自定义Hook中调用

## 8. API请求最佳实践
- 统一封装Axios/Fetch，配置拦截器
- 使用React Query/SWR管理服务器状态
- 实现全局错误处理和用户友好反馈

## 9. 性能优化要点
- **加载性能**：代码分割，延迟加载，图片优化
- **运行时性能**：避免不必要渲染，虚拟列表，防抖节流
- **关注核心指标**：Core Web Vitals，性能预算

## 10. 安全最佳实践
- 防范XSS（CSP）和CSRF攻击
- 前端不存储敏感信息
- 使用HTTPS，评估第三方脚本安全性

## 11. 工程化实践
- ESLint+Prettier保障代码质量
- 自动化测试（单元测试、组件测试、E2E测试）
- 合理选择和管理依赖
- CI/CD自动化部署

## 12. 国际化简要指南
- 使用翻译键代替硬编码文本
- 考虑文本长度变化对布局影响
- 合理格式化日期、数字和货币
```
```

## 1.5 后端开发规范

Rules Types：Agent Requested

```
```
# 后端开发规范

## 1. 基本原则
- **可维护性**：代码易于理解、修改和扩展
- **可测试性**：便于单元测试和集成测试
- **可伸缩性**：系统能水平扩展应对负载增长
- **安全性**：防范常见安全威胁
- **性能优化**：高效利用系统资源
- **一致性**：统一编码风格和架构模式

## 2. 架构设计
### 分层架构
```
表示层/API层 → 业务逻辑层 → 数据访问层 → 数据库
```

- **表示层**：处理请求响应、参数验证、权限控制
- **业务层**：核心业务规则、事务管理、领域模型
- **数据访问层**：数据库交互、CRUD操作、数据转换

### 核心设计模式
- **依赖注入**：减少组件间耦合
- **仓储模式**：抽象数据访问，便于测试
- **工厂模式**：创建复杂对象，隐藏创建逻辑
- **策略模式**：动态切换算法实现

## 3. 代码规范
### 命名与格式
- **类/接口**：大驼峰命名法(PascalCase)
- **方法/变量**：小驼峰命名法(camelCase)
- **常量**：全大写下划线分隔(MAX_SIZE)
- **缩进**：统一使用4空格或2空格
- **行长度**：不超过120字符

### 注释规范
- 使用标准文档注释(JavaDoc等)
- 注释解释"为什么"而非"是什么"
- 复杂逻辑必须添加注释说明

## 4. API设计
### RESTful API核心原则
- 资源用名词复数，如`/users`
- 正确使用HTTP方法(GET/POST/PUT/PATCH/DELETE)
- 标准状态码(200/201/400/401/403/404/500等)
- 统一的响应格式与错误处理
- URL或请求头版本控制

### API文档
- 使用Swagger/OpenAPI自动生成文档
- 包含端点描述、参数、响应格式和示例

## 5. 数据库设计
### 数据库最佳实践
- **命名规范**：表名字段名小写下划线分隔
- **索引设计**：为常用查询字段创建索引，避免过度索引
- **数据类型**：合理选择，日期时间用标准类型，金额用DECIMAL
- **ORM使用**：避免N+1问题，合理使用懒加载与预加载

## 6. 安全规范
### 核心安全措施
- **认证授权**：使用JWT/OAuth2，RBAC权限控制
- **数据安全**：
  - 输入验证(防注入攻击)
  - 敏感数据加密
  - HTTPS传输
  - 参数化SQL查询
  - CSRF防护

## 7. 性能优化
### 关键优化点
- **代码优化**：合适的数据结构和算法，避免O(n²)复杂度
- **数据库优化**：索引设计，避免全表扫描，分页查询
- **缓存策略**：多级缓存，合理的过期策略，防止缓存雪崩和穿透

## 8. 异常与日志
### 异常处理
- 全局统一异常处理机制
- 区分业务异常和系统异常
- 提供明确的错误信息

### 日志规范
- 合理使用日志级别(ERROR/WARN/INFO/DEBUG)
- 记录关键业务操作和异常
- 避免记录敏感信息

## 9. 测试与部署
### 测试策略
- 单元测试覆盖核心业务逻辑
- 集成测试验证服务间交互
- CI/CD流水线集成测试

### 部署最佳实践
- 环境隔离与配置管理
- 容器化部署(Docker+Kubernetes)
- 应用监控与健康检查
- 语义化版本管理
```
```

## 1.6 Git提交规范

Rules Types：Manual

```
```
# Git 提交规范 (Git Commit Rules)

## 1. 提交信息格式
### 1.1 基本格式
```
<type>(<scope>): <subject>
<body>
<footer>
```
### 类型（Type）
- feat: 新功能
- fix: 修复bug
- docs: 文档更改
- style: 代码格式调整
- refactor: 重构代码
- perf: 性能优化
- test: 测试相关
- chore: 构建过程或辅助工具变动

### 主题（Subject）
- 简短描述，不超过50字符
- 使用现在时态
- 首字母不大写，结尾不加句号

## 2. 分支管理核心
- main/master: 主分支，稳定版本
- develop: 开发分支
- feature/*: 功能分支
- bugfix/*: 修复分支
- hotfix/*: 紧急修复分支
- release/*: 发布分支

## 3. 提交最佳实践
### 原子提交
- 每次提交只做一件事
- 相关改动放在同一提交
- 不相关改动分开提交

### 提交频率
- 经常小批量提交
- 功能完成时提交
- 解决冲突后提交

### 提交前准备
- 拉取最新代码
- 确保测试通过
- 检查代码风格
- 更新必要文档

### 安全考虑
- 不提交敏感信息
- 使用.gitignore忽略不必要文件
- 保护主分支

## 4. 版本管理
- 遵循语义化版本：主版本号.次版本号.修订号
- 重要发布打标签：`git tag -a v1.0.0 -m "version 1.0.0"`

## 5. 代码审查
- 使用Pull Request进行代码审查
- 解决所有评论后再合并
- 合并后删除临时分支
```
```

## 1.7 API设计规范

Rules Types：Manual

```
```
# API 设计规范

## 1. RESTful API 设计原则
### 1.1 资源命名
- 使用名词复数形式，小写字母和连字符
- 避免动词，保持简洁明确

### 1.2 HTTP 方法使用
- GET：获取资源
- POST：创建资源
- PUT：完整更新资源
- PATCH：部分更新资源
- DELETE：删除资源

### 1.3 查询参数
- 分页：page, per_page
- 排序：sort, order
- 过滤：filter, q
- 字段选择：fields

## 2. 版本与兼容性
### 2.1 版本控制
```javascript
// URL 路径版本
/api/v1/users

// 请求头版本
Accept: application/vnd.company.api+json;version=1

### 2.2 版本兼容性
- 向后兼容性保证
- 版本升级策略
- 废弃流程
- 过渡期支持

### 2.3 文档要求
```markdown

## 3. 请求/响应规范
### 3.1 响应格式
```javascript
// 成功响应
{
  "data": {
    "id": "123",
    "type": "users",
    "attributes": { ... },
    "relationships": { ... }
  },
  "meta": { "total_count": 100 }
}

// 错误响应
{
  "errors": [{
    "status": "400",
    "code": "VALIDATION_ERROR",
    "title": "验证错误",
    "detail": "邮箱格式不正确",
    "source": { "pointer": "/data/attributes/email" }
  }]
}

### 3.2 状态码使用
200 OK              // 成功获取资源
201 Created         // 成功创建资源
204 No Content      // 成功处理但无返回内容
400 Bad Request     // 请求格式错误
401 Unauthorized    // 未认证
403 Forbidden       // 无权限
404 Not Found       // 资源不存在
422 Unprocessable   // 验证错误
500 Server Error    // 服务器错误
```

## 4. 安全与性能
### 4.1 认证与授权
- 使用JWT或OAuth2.0
- 基于角色的权限控制

## 5. 文档与监控
### 5.1 API文档
- 使用OpenAPI/Swagger规范
- 提供详细示例代码

### 5.2 监控与日志
- 记录关键操作信息
- 设置性能指标和告警
```
```

# 二、编程语言规范

这里只整理了初学者比较常见的几种语言，如果大家后续遇到其它新的编程语言，按照类似的结构让Cursor进行整理仿写即可。

## 2.1 HTML编码规范

可以在File pattern matches中把你想要支持的html文件都放进去；

当然，如果你整个项目都只是HTML，那就设置成Always。

```
```
# HTML 项目开发规范

## 1. HTML 版本规范
- 使用 HTML5 进行开发
- 在文档开头使用正确的 DOCTYPE 声明
- 指定文档的语言属性

## 2. 代码风格规范
### 基本规范
- 使用两个空格进行缩进
- 使用小写标签和属性名
- 属性值使用双引号
- 所有元素和属性使用小写字母

### 命名规范
```html
<!-- ID：使用小驼峰命名法 -->
<div id="userProfile"></div>

<!-- 类名：使用连字符分隔 -->
<div class="user-avatar"></div>

<!-- 文件名：使用小写字母和连字符 -->
<!-- user-profile.html -->
```

### 注释规范
```html
<!-- 单行注释 -->

<!--
  多行注释
  详细说明此区块的功能
  更新日期：2023-10-01
-->
```

## 3. 项目结构规范
推荐的HTML项目结构：
```
project_name/
├── index.html
├── pages/
│   ├── about.html
│   ├── contact.html
│   └── products.html
├── assets/
│   ├── css/
│   ├── js/
│   └── images/
├── components/
│   ├── header.html
│   ├── footer.html
│   └── sidebar.html
└── README.md
```

## 3. 资源管理
### 外部资源
- CSS文件放在`<head>`标签内
- JavaScript文件放在`</body>`标签前
- 为外部资源添加适当的属性（async, defer等）

## 4. 元素属性规范
- 属性顺序：id, class, name, data-*, src/href, alt/title, aria-*, role
- 布尔属性不指定值

## 5. 语义化规范
- 使用语义化标签增强可访问性和SEO

## 6. 表单规范
### 表单元素
- 使用`<label>`标签关联表单控件
- 为表单元素提供有意义的name和id
- 使用适当的input类型

## 7. 可访问性规范
- 使用适当的ARIA角色和属性
- 确保颜色对比度符合WCAG标准
- 为图片提供alt属性

## 8. 性能优化
- 压缩HTML文件
- 延迟加载非关键资源
- 使用响应式图片

## 9. 文档规范
- 编写详细的 README.md
- 包含项目结构说明
- 提供开发和部署指南
- 维护更新日志
```
```

## 2.2 JavaScript/Typescript编码规范

可以在File pattern matches中把你想要支持的.js/.ts/.d.ts等文件都放进去；

当然，如果你整个项目都只是.js，那就设置成Always。

```
```
# JavaScript/TypeScript 编码规范

## 1. 版本规范
### JavaScript
- 支持 ES6+ (ECMAScript 2015+)
- 使用 Babel 进行代码转译
- 明确指定 Node.js 版本要求（建议 14.x+）

### TypeScript
- 使用 TypeScript 4.x+ 版本
- 启用严格模式 (`strict: true`)
- 配置 `tsconfig.json` 明确编译选项

## 2. 代码风格规范
### 命名规范
```typescript
// 类名：使用大驼峰命名法
class UserService {
  // ...
}

// 接口名：使用大驼峰命名法，可以加 I 前缀
interface IUserData {
  // ...
}

// 函数名：使用小驼峰命名法
function calculateTotalPrice() {
  // ...
}

// 变量名：使用小驼峰命名法
const userData = { ... };

// 常量：使用大写字母和下划线
const MAX_RETRY_COUNT = 3;

// 组件名：使用大驼峰命名法
const UserProfile = () => {
  // ...
};
```

### 注释规范
```typescript
/**
 * 处理用户数据并返回处理后的结果
 * 
 * @param {number} userId - 用户ID
 * @param {Object} data - 原始用户数据
 * @returns {Promise<Object>} 处理后的用户数据
 * @throws {Error} 当用户ID不存在时抛出
 */
async function processUserData(userId: number, data: object): Promise<object> {
  // ...
}
```

## 3. 项目结构规范
推荐的项目结构：
```
project_name/
├── src/
│   ├── components/      # React/Vue组件
│   ├── services/       # 业务服务
│   ├── utils/         # 工具函数
│   ├── types/         # TypeScript类型定义
│   └── constants/     # 常量定义
├── tests/
│   ├── unit/
│   └── integration/
├── public/
├── dist/              # 构建输出
├── node_modules/
├── package.json
├── tsconfig.json     # TypeScript配置
├── .eslintrc.js     # ESLint配置
├── .prettierrc      # Prettier配置
└── README.md
```

## 4. 依赖管理
- 使用 `npm` 或 `yarn` 管理依赖
- 锁定依赖版本（package-lock.json/yarn.lock）
- 区分 dependencies 和 devDependencies
- 定期更新依赖版本，处理安全隐患

## 5. TypeScript 特性使用
### 类型定义
```typescript
// 使用接口定义对象类型
interface User {
  id: number;
  name: string;
  age?: number;
}

// 使用类型别名定义联合类型
type Status = 'active' | 'inactive' | 'pending';

// 泛型使用
function getList<T>(items: T[]): T[] {
  return items;
}
```

### 类型断言
```typescript
// 使用 as 语法而不是尖括号
const value = someValue as string;
```

## 5. 异步编程规范
- 优先使用 async/await
- 正确处理异步错误
- 避免回调地狱

## 6. 前端框架特定规范
### React
- 使用函数组件和 Hooks
- 合理划分组件职责
- 使用 PropTypes 或 TypeScript 类型检查

### Vue
- 遵循 Vue 3 组合式 API 规范
- 使用 `<script setup>` 语法
- 使用 TypeScript 装饰器

## 性能优化
- 代码分割和懒加载
- 合理使用缓存
- 优化打包体积
- 使用性能分析工具

## 安全规范
- 防止 XSS 攻击
- 避免 SQL 注入
- 使用 HTTPS
- 保护敏感信息

## 文档规范
- 使用 JSDoc 或 TypeDoc 生成文档
- 编写详细的 README.md
- 组件文档使用 Storybook
- API 文档使用 Swagger/OpenAPI 
```
```

或者这个版本也可以：

```
```
## 概述

[tRPC](https://trpc.io/) 支持端到端的类型安全 API，使您无需依赖模式定义、代码生成或担心运行时错误，即可构建和使用 API。这些规则将帮助您遵循 tRPC v11 的最佳实践。

## 项目结构

为了保持清晰的 tRPC 设置，请遵循以下推荐的项目结构：
```
.
├── src
│   ├── pages
│   │   ├── _app.tsx  # 在此处添加 `createTRPCNext` 设置
│   │   ├── api
│   │   │   └── trpc
│   │   │       └── [trpc].ts  # tRPC HTTP 请求处理器
│   │   ├── server
│   │   │   ├── routers
│   │   │   │   ├── _app.ts  # 主应用路由
│   │   │   │   ├── [feature].ts  # 特定功能的路由
│   │   │   │   └── [...]
│   │   │   ├── context.ts   # 创建应用上下文
│   │   │   └── trpc.ts      # 过程辅助函数
│   │   └── utils
│   │       └── trpc.ts  # 类型安全的 tRPC Hooks
```

## 服务端设置

### 初始化 tRPC 后端

```typescript
// server/trpc.ts
import { initTRPC } from '@trpc/server';

// 初始化 tRPC 后端（每个后端应仅执行一次）
const t = initTRPC.create();

// 导出可复用的路由和过程辅助函数
export const router = t.router;
export const publicProcedure = t.procedure;
```

### 创建路由

```typescript
// server/routers/_app.ts
import { z } from 'zod';
import { router, publicProcedure } from '../trpc';

export const appRouter = router({
  // 您的过程定义放在这里
  greeting: publicProcedure
    .input(z.object({ name: z.string() }))
    .query(({ input }) => {
      return `你好 ${input.name}`;
    }),
});

// 导出 API 的类型定义（而不是路由本身！）
export type AppRouter = typeof appRouter;
```

## 客户端设置

### Next.js 集成

```typescript
// utils/trpc.ts
import { httpBatchLink } from '@trpc/client';
import { createTRPCNext } from '@trpc/next';
import type { AppRouter } from '../server/routers/_app';

function getBaseUrl() {
  if (typeof window !== 'undefined') return ''; // 浏览器应使用相对路径
  if (process.env.VERCEL_URL) return `https://${process.env.VERCEL_URL}`; // SSR 应使用 Vercel URL
  return `http://localhost:${process.env.PORT ?? 3000}`; // 开发环境 SSR 应使用 localhost
}

export const trpc = createTRPCNext<AppRouter>({
  config() {
    return {
      links: [
        httpBatchLink({
          url: `${getBaseUrl()}/api/trpc`,
          // 需要时包含认证头信息
          async headers() {
            return {
              // authorization: getAuthCookie(),
            };
          },
        }),
      ],
    };
  },
  ssr: false, // 如果要使用服务端渲染，请设置为 true
});
```

## 最佳实践

1. **使用 Zod 进行输入验证**：始终使用 Zod 验证过程输入，以获得更好的类型安全和运行时验证。 ```typescript
    import { z } from 'zod';

    procedure
      .input(z.object({ 
        id: z.string().uuid(),
        email: z.string().email(),
        age: z.number().min(18) 
      }))
      .mutation(({ input }) => { /* 你的代码 */ })
    ```

2. **按功能组织路由**：将您的路由拆分为逻辑域/功能，而不是使用一个庞大的路由。 ```typescript
    // server/routers/user.ts
    export const userRouter = router({
      list: publicProcedure.query(() => { /* ... */ }),
      byId: publicProcedure.input(z.string()).query(({ input }) => { /* ... */ }),
      create: publicProcedure.input(/* ... */).mutation(({ input }) => { /* ... */ }),
    });

    // server/routers/_app.ts
    import { userRouter } from './user';
    import { postRouter } from './post';

    export const appRouter = router({
      user: userRouter,
      post: postRouter,
    });
    ```

3. **使用中间件处理通用逻辑**：应用中间件处理认证、日志记录或其他横切关注点。 ```typescript
    const isAuthed = t.middleware(({ next, ctx }) => {
      if (!ctx.user) {
        throw new TRPCError({ code: 'UNAUTHORIZED' });
      }
      return next({
        ctx: {
          // 将用户信息添加到上下文
          user: ctx.user,
        },
      });
    });

    const protectedProcedure = t.procedure.use(isAuthed);
    ```

4. **使用适当的错误处理**：利用 tRPC 的错误处理机制实现一致的错误响应。 ```typescript
    import { TRPCError } from '@trpc/server';

    publicProcedure
      .input(z.string())
      .query(({ input }) => {
        const user = getUserById(input);
        if (!user) {
          throw new TRPCError({
            code: 'NOT_FOUND',
            message: `未找到 ID 为 ${input} 的用户`,
          });
        }
        return user;
      });
    ```

5. **使用数据转换器**：使用 SuperJSON 自动处理日期、Map、Set 等数据类型。 ```typescript
    import { initTRPC } from '@trpc/server';
    import superjson from 'superjson';

    const t = initTRPC.create({
      transformer: superjson,
    });
    ```

6. **利用 React Query 集成**：对于 React 项目，使用 tRPC 的 React Query 工具进行数据获取、变更和缓存。 ```tsx
    function UserProfile({ userId }: { userId: string }) {
      const { data, isLoading, error } = trpc.user.byId.useQuery(userId);

      if (isLoading) return <div>加载中...</div>;
      if (error) return <div>错误：{error.message}</div>;

      return <div>{data.name}</div>;
    }
    ```

7. **上下文创建**：创建适当的上下文对象，以便在过程之间共享资源。 ```typescript
    // server/context.ts
    import { inferAsyncReturnType } from '@trpc/server';
    import * as trpcNext from '@trpc/server/adapters/next';
    import { prisma } from './prisma';

    export async function createContext({
      req,
      res,
    }: trpcNext.CreateNextContextOptions) {
      const user = await getUser(req);
      return {
        req,
        res,
        prisma,
        user,
      };
    }

    export type Context = inferAsyncReturnType<typeof createContext>;
    ```

8. **类型导出**：从服务器代码向客户端代码仅导出类型，而不是实际的路由实现。 ```typescript
    // 导出路由类型签名，而不是路由本身
    export type AppRouter = typeof appRouter;
    ```

9. **过程类型**：为不同的授权级别使用不同的过程类型。 ```typescript
    export const publicProcedure = t.procedure;
    export const protectedProcedure = t.procedure.use(isAuthed);
    export const adminProcedure = t.procedure.use(isAdmin);
    ```

10. **性能优化**：使用批处理和预加载优化数据加载。 ```typescript
    // Next.js 设置中的客户端批处理
    httpBatchLink({
      url: `${getBaseUrl()}/api/trpc`,
      maxURLLength: 2083,
    })

    // Next.js 中的数据预取
    export async function getStaticProps() {
      const ssg = createServerSideHelpers({
        router: appRouter,
        ctx: {},
      });

      await ssg.post.byId.prefetch('1');

      return {
        props: {
          trpcState: ssg.dehydrate(),
        },
        revalidate: 1,
      };
    }
    ```

## 版本兼容性

本指南适用于 tRPC v11，它要求：
- TypeScript >= 5.7.2
- 严格 TypeScript 模式（tsconfig.json 中设置 `"strict": true`）

## 更多资源

- [官方文档](https://trpc.io/docs)
- [GitHub 仓库](https://github.com/trpc/trpc)
- [示例应用](https://trpc.io/docs/example-apps)
```
```

## 2.3 CSS编码规范

虽然CSS并不是一种编程语言，但我们依然可以为它单独制作一份类似的开发规范。

```
```
# CSS 编码规范

## 1. CSS 版本与预处理器规范
- 使用 CSS3+ 特性进行开发
- 建议使用 Sass/SCSS 作为 CSS 预处理器
- 使用 PostCSS 进行后处理
- 明确指定项目的浏览器兼容性要求（在 `.browserslistrc` 中配置）

## 2. 代码风格规范
### 基本规范
- 使用两个空格进行缩进
- 每个声明块后使用一个空行
- 在 `{` 前添加一个空格
- 在属性值前添加一个空格
- 每个属性声明末尾都要加分号
- 使用十六进制颜色代码时使用小写字母

### 命名规范
```css
/* BEM命名规范 */
.block {}
.block__element {}
.block--modifier {}

/* 组件命名示例 */
.user-profile {}
.user-profile__avatar {}
.user-profile--active {}

/* 功能类命名 */
.text-center {}
.mt-20 {} /* margin-top: 20px */
.flex-container {}
```

### 注释规范
```css
/* 单行注释 */

/**
 * 多行注释
 * @desc 详细说明此区块的功能
 * @author 作者名
 * @date 2024-03-20
 */
```

## 3. 项目结构规范
推荐的CSS项目结构：
```
styles/
├── base/
│   ├── _reset.scss
│   ├── _typography.scss
│   └── _variables.scss
├── components/
│   ├── _buttons.scss
│   ├── _forms.scss
│   └── _cards.scss
├── layouts/
│   ├── _header.scss
│   ├── _footer.scss
│   └── _grid.scss
├── utils/
│   ├── _mixins.scss
│   └── _functions.scss
├── vendors/
│   └── _normalize.scss
└── main.scss
```

## 4. 选择器规范
### 选择器优先级
- 避免使用 `!important`
- 避免过度嵌套（最多3层）
- 使用类选择器而不是ID选择器
- 避免使用标签选择器和通配符选择器

```css
/* 推荐 */
.header .nav-item {}

/* 不推荐 */
#header div.nav div.nav-item {}
```

### 属性书写顺序
推荐按以下顺序书写属性：
1. 布局属性（position, display, float等）
2. 盒模型属性（width, height, margin, padding等）
3. 文字排版（font, line-height, text-align等）
4. 视觉效果（color, background, border等）
5. 其他属性（cursor, overflow等）

## 5. 响应式设计规范
### 媒体查询
- 使用移动优先的设计理念
- 统一断点值

### 弹性单位
- 优先使用相对单位（rem, em, vh, vw）
- 设置基准字体大小

## 6. 性能优化规范
### CSS选择器性能
- 避免深层嵌套
- 避免使用通配符
- 利用继承特性

### 资源优化
- 合理使用CSS Sprites
- 使用字体图标或SVG图标
- 压缩CSS文件
- 使用CSS预加载

## 7. 变量与主题规范
### CSS变量
```css
:root {
  /* 颜色变量 */
  --primary-color: #007bff;
  --secondary-color: #6c757d;

  /* 间距变量 */
  --spacing-unit: 8px;
  --container-padding: calc(var(--spacing-unit) * 2);

  /* 字体变量 */
  --font-family-base: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto;
  --font-size-base: 16px;
}
```

### 主题切换
```css
[data-theme="dark"] {
  --bg-color: #1a1a1a;
  --text-color: #ffffff;
}

[data-theme="light"] {
  --bg-color: #ffffff;
  --text-color: #1a1a1a;
}
```

## 8. 文档规范
- 维护样式指南（Style Guide）
- 编写组件文档
- 记录主题配置
- 提供使用示例
- 维护更新日志

## 9. 最佳实践
### 避免
- 避免使用行内样式
- 避免CSS和HTML强耦合
- 避免使用浏览器引擎前缀（使用Autoprefixer）
- 避免重复的样式声明

### 推荐
- 使用CSS自定义属性（变量）
- 模块化CSS
- 使用CSS Grid和Flexbox进行布局
- 实现响应式设计
- 考虑可访问性 
```
```

## 2.4 Python编码规范

可以在File pattern matches中把你想要支持的.py等文件都放进去；

当然，如果项目都只是.py,那就设置成Always。

```
```
# Python 项目开发规范

## 1. Python 版本规范
- 使用 Python 3.8+ 版本进行开发
- 明确指定项目的 Python 版本要求
- 在 `pyproject.toml` 或 `setup.py` 中声明支持的 Python 版本范围

## 2. 代码风格规范
### PEP 8 规范
- 严格遵循 [PEP 8](https://www.python.org/dev/peps/pep-0008/) 风格指南
- 使用工具自动化格式检查（如 `black`, `flake8`, `pylint`）

### 命名规范
```python
# 类名：使用大驼峰命名法
class UserAccount:
    pass

# 函数名：使用小写字母和下划线
def calculate_total_amount():
    pass

# 变量名：使用小写字母和下划线
user_name = "张三"

# 常量：使用大写字母和下划线
MAX_CONNECTIONS = 100

# 模块名：使用小写字母，可以使用下划线
import data_processor
```

### 注释规范
```python
def process_user_data(user_id: int, data: dict) -> dict:
    """
    处理用户数据并返回处理后的结果。

    Args:
        user_id (int): 用户ID
        data (dict): 原始用户数据

    Returns:
        dict: 处理后的用户数据

    Raises:
        ValueError: 当用户ID不存在时抛出
        KeyError: 当必要的数据字段缺失时抛出
    """
    pass
```

## 3. 项目结构规范
推荐的Python项目结构：
```
project_name/
├── src/
│   └── project_name/
│       ├── __init__.py
│       ├── core/
│       ├── utils/
│       └── config/
├── tests/
│   ├── __init__.py
│   ├── test_core/
│   └── test_utils/
├── docs/
├── scripts/
├── pyproject.toml  # 或 setup.py
├── requirements.txt
├── requirements-dev.txt
└── README.md
```

## 4. 依赖管理
### 虚拟环境
- 使用 `venv` 或 `conda` 创建独立的虚拟环境
- 将虚拟环境文件夹添加到 `.gitignore`

### 包管理
- 使用 `pip` 或 `poetry` 管理依赖
- 区分开发依赖和生产依赖
- 锁定依赖版本，使用 `requirements.txt` 或 `poetry.lock`

## 5. 类型提示
- 使用类型注解增强代码可读性和可维护性
```python
from typing import List, Dict, Optional

def get_user_info(user_id: int) -> Optional[Dict[str, str]]:
    pass

def process_items(items: List[Dict[str, any]]) -> None:
    pass
```

## 6. 性能优化
- 使用生成器处理大数据集
- 适当使用列表推导式
- 避免全局变量
- 使用 `collections` 模块的高效数据结构

## 7. 文档规范
- 使用 Sphinx 生成文档
- 编写详细的 README.md
- 包含安装说明和使用示例
- 提供 API 文档 
```
```

或者可以参考下面这个示例：

```
```
## 核心能力概述

**数据分析与处理**
```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import train_test_split

# 设置绘图样式
plt.style.use('seaborn-v0_8')
%matplotlib inline
```

**数据质量验证框架**
```python
def validate_dataset(df):
    """执行全面的数据质量检查"""
    validation_report = {
        'shape': df.shape,
        'missing_values': df.isnull().sum(),
        'data_types': df.dtypes,
        'duplicates': df.duplicated().sum(),
        'numeric_summary': df.describe() if df.select_dtypes(include=np.number).shape[1] > 0 else None
    }
    return validation_report

def handle_missing_data(df, strategy='auto'):
    """智能处理缺失值"""
    if strategy == 'auto':
        # 数值列用中位数，分类列用众数
        for col in df.columns:
            if df[col].dtype in ['float64', 'int64']:
                df[col].fillna(df[col].median(), inplace=True)
            else:
                df[col].fillna(df[col].mode()[0] if not df[col].mode().empty else 'Unknown', inplace=True)
    return df
```

## 可视化最佳实践

**可重用的绘图函数**
```python
def create_distribution_plot(data, numerical_columns, figsize=(15, 10)):
    """创建数值变量的分布图"""
    fig, axes = plt.subplots(2, 2, figsize=figsize)
    axes = axes.ravel()

    for idx, col in enumerate(numerical_columns[:4]):
        axes[idx].hist(data[col], bins=30, alpha=0.7, color='skyblue', edgecolor='black')
        axes[idx].set_title(f'Distribution of {col}')
        axes[idx].set_xlabel(col)
        axes[idx].set_ylabel('Frequency')

    plt.tight_layout()
    return fig

def create_correlation_heatmap(data, method='pearson'):
    """创建相关性热力图"""
    plt.figure(figsize=(12, 8))
    correlation_matrix = data.corr(method=method)
    mask = np.triu(np.ones_like(correlation_matrix, dtype=bool))

    sns.heatmap(correlation_matrix, mask=mask, annot=True, cmap='coolwarm', 
                center=0, square=True, linewidths=0.5)
    plt.title('Feature Correlation Heatmap')
    plt.tight_layout()
    return plt.gcf()
```

## 高效数据处理模式

**方法链式操作示例**
```python
def preprocess_dataframe(raw_df):
    """使用方法链进行数据预处理"""
    processed_df = (raw_df
                   .pipe(handle_missing_data)
                   .assign(
                       total_amount=lambda x: x['quantity'] * x['unit_price'],
                       transaction_date=lambda x: pd.to_datetime(x['transaction_date']),
                       month=lambda x: x['transaction_date'].dt.month_name()
                   )
                   .query('total_amount > 0')  # 过滤无效交易
                   .sort_values('transaction_date')
                   .reset_index(drop=True)
                  )
    return processed_df

# 高效分组聚合
def calculate_aggregate_metrics(df, group_cols, value_cols):
    """计算分组聚合指标"""
    aggregation_rules = {
        col: ['mean', 'median', 'std', 'sum'] for col in value_cols
    }

    aggregate_df = (df
                   .groupby(group_cols)
                   .agg(aggregation_rules)
                   .round(2)
                   .reset_index()
                  )

    # 扁平化多级列名
    aggregate_df.columns = ['_'.join(col).strip() if col[1] else col[0] 
                           for col in aggregate_df.columns]

    return aggregate_df
```

## 性能优化技巧

```python
def optimize_dataframe_memory(df):
    """优化DataFrame内存使用"""
    initial_memory = df.memory_usage(deep=True).sum() / 1024**2  # MB

    for col in df.columns:
        col_type = df[col].dtype

        if col_type == 'object':
            # 转换为分类类型以节省内存
            if df[col].nunique() / len(df) < 0.5:
                df[col] = df[col].astype('category')

        elif col_type in ['int64', 'float64']:
            # 向下转换数值类型
            c_min = df[col].min()
            c_max = df[col].max()

            if col_type == 'int64':
                if c_min > np.iinfo(np.int8).min and c_max < np.iinfo(np.int8).max:
                    df[col] = df[col].astype(np.int8)
                elif c_min > np.iinfo(np.int16).min and c_max < np.iinfo(np.int16).max:
                    df[col] = df[col].astype(np.int16)
                elif c_min > np.iinfo(np.int32).min and c_max < np.iinfo(np.int32).max:
                    df[col] = df[col].astype(np.int32)
            else:
                if c_min > np.finfo(np.float32).min and c_max < np.finfo(np.float32).max:
                    df[col] = df[col].astype(np.float32)

    final_memory = df.memory_usage(deep=True).sum() / 1024**2
    print(f"内存使用优化: {initial_memory:.2f} MB -> {final_memory:.2f} MB")

    return df
```

## Jupyter Notebook工作流模板

```python
# 1. 环境设置和数据加载
def setup_analysis_environment():
    """设置分析环境"""
    import warnings
    warnings.filterwarnings('ignore')

    print("✅ 分析环境已就绪")
    print(f"📊 pandas版本: {pd.__version__}")
    print(f"📈 matplotlib版本: {plt.matplotlib.__version__}")

# 2. 数据探索模板
def exploratory_data_analysis(df, target_column=None):
    """执行探索性数据分析"""
    print("🔍 开始探索性数据分析...")

    # 基础信息
    print(f"数据集形状: {df.shape}")
    print("\n数据类型:")
    print(df.dtypes.value_counts())

    # 数据质量报告
    validation_report = validate_dataset(df)
    print(f"\n重复行数量: {validation_report['duplicates']}")

    # 可视化探索
    if target_column:
        create_target_analysis(df, target_column)

    return validation_report

# 3. 特征分析函数
def analyze_feature_relationships(df, numerical_features, categorical_features):
    """分析特征关系"""
    fig, axes = plt.subplots(1, 2, figsize=(16, 6))

    # 数值特征相关性
    if len(numerical_features) > 1:
        sns.heatmap(df[numerical_features].corr(), annot=True, ax=axes[0])
        axes[0].set_title('Numerical Features Correlation')

    # 分类特征分布
    if categorical_features:
        top_category = df[categorical_features[0]].value_counts().head(10)
        sns.barplot(x=top_category.values, y=top_category.index, ax=axes[1])
        axes[1].set_title(f'Top 10 {categorical_features[0]} Categories')

    plt.tight_layout()
    plt.show()
```

## 错误处理和验证

```python
def safe_data_operation(operation_func, *args, **kwargs):
    """安全执行数据操作"""
    try:
        result = operation_func(*args, **kwargs)
        print(f"✅ {operation_func.__name__} 执行成功")
        return result
    except Exception as e:
        print(f"❌ {operation_func.__name__} 执行失败: {str(e)}")
        return None

def validate_feature_importance(importance_df, threshold=0.01):
    """验证特征重要性"""
    significant_features = importance_df[importance_df['importance'] > threshold]

    if len(significant_features) == 0:
        raise ValueError("没有发现显著重要的特征，请检查数据或调整阈值")

    return significant_features
```
```
```

## 2.5 Java编码规范

```
```
# Java 项目开发规范

## 1. 版本规范
- 使用 Java 11+ LTS 版本
- 明确指定 JDK 版本要求
- 使用 Maven 或 Gradle 进行项目管理

## 2. 代码风格规范
### 命名规范
```java
// 类名：使用大驼峰命名法
public class UserService {
    // ...
}

// 接口名：使用大驼峰命名法
public interface UserRepository {
    // ...
}

// 方法名：使用小驼峰命名法
public void processUserData() {
    // ...
}

// 变量名：使用小驼峰命名法
private String userName;

// 常量：使用大写字母和下划线
public static final int MAX_RETRY_COUNT = 3;

// 包名：全小写，用点分隔
package com.company.project.module;
```

### 注释规范
```java
/**
 * 用户服务类，处理用户相关的业务逻辑
 * 
 * @author 作者名
 * @version 1.0
 */
public class UserService {
    /**
     * 处理用户数据并返回处理后的结果
     * 
     * @param userId 用户ID
     * @param data 原始用户数据
     * @return 处理后的用户数据
     * @throws UserNotFoundException 当用户不存在时抛出
     */
    public UserDTO processUserData(Long userId, UserData data) throws UserNotFoundException {
        // ...
    }
}
```

## 3. 项目结构规范
推荐的项目结构：
```
project_name/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/company/project/
│   │   │       ├── config/        # 配置类
│   │   │       ├── controller/    # 控制器
│   │   │       ├── service/       # 服务层
│   │   │       ├── repository/    # 数据访问层
│   │   │       ├── model/         # 数据模型
│   │   │       ├── dto/           # 数据传输对象
│   │   │       └── util/          # 工具类
│   │   └── resources/
│   │       ├── application.yml    # 应用配置
│   │       └── static/           # 静态资源
│   └── test/
│       └── java/
├── pom.xml                        # Maven配置
└── README.md
```

## 4. 依赖管理
### Maven
```xml
<properties>
    <java.version>11</java.version>
    <spring-boot.version>2.7.0</spring-boot.version>
</properties>

<dependencies>
    <!-- 明确指定依赖版本 -->
</dependencies>
```

### Gradle
```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '2.7.0'
}

dependencies {
    // 使用依赖版本管理
    implementation platform('org.springframework.boot:spring-boot-dependencies:2.7.0')
}
```

## 5. 设计模式规范
- 遵循 SOLID 原则
- 优先使用组合而非继承
- 使用依赖注入
- 单一职责原则

```java
@Service
public class UserServiceImpl implements UserService {
    private final UserRepository userRepository;

    @Autowired
    public UserServiceImpl(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

## 6. 异常处理规范
- 使用自定义异常类
- 统一异常处理
- 合理使用检查型和非检查型异常

```java
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleUserNotFoundException(UserNotFoundException ex) {
        // 异常处理逻辑
    }
}
```

## 7. 数据库访问规范
- 使用 JPA 或 MyBatis
- 正确使用事务
- 优化 SQL 查询
- 使用连接池

```java
@Transactional
public User createUser(UserDTO userDTO) {
    // 数据库操作
}
```

## 8. 安全规范
- 密码加密存储
- 使用 Spring Security
- 防止 SQL 注入
- 实施 RBAC 权限控制

## 9. 性能优化
- 使用缓存（Redis等）
- 异步处理（@Async）
- 批处理操作
- JVM 调优

## 10. 文档规范

- 使用 Javadoc 生成文档
- 使用 Swagger/OpenAPI 文档
- 编写详细的 README.md
- 维护 CHANGELOG.md
```
```

## 2.6 Swift/Objective-C编码规范

```
```
# Swift/Objective-C 编码规范

## 1. 版本规范
### Swift
- 使用 Swift 5.0+ 版本
- 使用 SwiftUI 和 Combine（iOS 13+）
- 使用 Swift Package Manager 管理依赖
- 明确指定最低部署目标版本

### Objective-C
- 使用现代 Objective-C 特性
- 支持 ARC（自动引用计数）
- 使用 CocoaPods 或 Carthage 管理依赖
- 遵循 Apple 官方编码规范

## 2. 代码风格规范
### Swift 命名规范
```swift
// 类型名（类、结构体、枚举、协议）：大驼峰命名法
class UserService {
    // ...
}

struct UserProfile {
    // ...
}

enum UserStatus {
    case active
    case inactive
    case pending
}

protocol UserRepository {
    func fetchUser(id: Int) -> User?
}

// 函数名：小驼峰命名法
func processUserData(user: User) -> Result<ProcessedData, Error> {
    // ...
}

// 变量名：小驼峰命名法
let userName = "张三"
var userAge = 25

// 常量：小驼峰命名法
let maxRetryCount = 3
```

### Objective-C 命名规范

```objc
// 类名：使用前缀
@interface XYZUserService : NSObject
// ...
@end

// 方法名：描述性命名
- (void)processUserData:(XYZUser *)user 
           completion:(void (^)(XYZResult *result, NSError *error))completion;

// 变量名：小驼峰命名法
@property (nonatomic, strong) NSString *userName;
@property (nonatomic, assign) NSInteger userAge;

// 常量：使用前缀和下划线
static const NSInteger XYZ_MaxRetryCount = 3;
```

### 注释规范

```swift
/// 用户服务类，处理用户相关的业务逻辑
///
/// 示例：
/// ```swift
/// let service = UserService()
/// let user = try service.getUser(id: 1)
/// ```
class UserService {
    /// 处理用户数据
    /// - Parameters:
    ///   - userId: 用户ID
    ///   - data: 原始数据
    /// - Returns: 处理后的数据
    /// - Throws: UserError 如果处理失败
    func processUserData(userId: Int, data: Data) throws -> ProcessedData {
        // ...
    }
}
```

## 3. 项目结构规范
推荐的项目结构：
```
ProjectName/
├── Sources/
│   ├── AppDelegate.swift
│   ├── SceneDelegate.swift
│   ├── Features/           # 功能模块
│   │   ├── User/
│   │   ├── Profile/
│   │   └── Settings/
│   ├── Core/              # 核心功能
│   │   ├── Network/
│   │   ├── Storage/
│   │   └── Utils/
│   ├── Resources/         # 资源文件
│   │   ├── Assets.xcassets/
│   │   └── Localizations/
│   └── Supporting Files/  # 配置文件
├── Tests/
│   ├── UnitTests/
│   └── UITests/
├── Frameworks/           # 第三方框架
├── Products/            # 构建产物
├── ProjectName.xcodeproj/
├── ProjectName.xcworkspace/
└── README.md
```

## 4. 依赖管理
### Swift Package Manager
```swift
// Package.swift
let package = Package(
    name: "ProjectName",
    platforms: [
        .iOS(.v13),
        .macOS(.v10_15)
    ],
    dependencies: [
        .package(url: "https://github.com/Alamofire/Alamofire.git", .upToNextMajor(from: "5.0.0"))
    ]
)
```

### CocoaPods
```ruby
# Podfile
platform :ios, '13.0'

target 'ProjectName' do
  use_frameworks!

  pod 'Alamofire', '~> 5.0'
  pod 'SwiftyJSON', '~> 5.0'

  target 'ProjectNameTests' do
    inherit! :search_paths
    pod 'Quick'
    pod 'Nimble'
  end
end
```

## 5. SwiftUI 和 UIKit 规范
### SwiftUI
```swift
struct ContentView: View {
    @StateObject private var viewModel = ViewModel()

    var body: some View {
        NavigationView {
            List(viewModel.items) { item in
                ItemRow(item: item)
            }
            .navigationTitle("列表")
        }
    }
}
```

### UIKit
```swift
class ViewController: UIViewController {
    private lazy var tableView: UITableView = {
        let table = UITableView(frame: .zero, style: .plain)
        table.delegate = self
        table.dataSource = self
        return table
    }()

    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
    }
}
```

## 错误处理规范

```swift
enum UserError: LocalizedError {
    case notFound(id: Int)
    case invalidData
    case networkError(Error)

    var errorDescription: String? {
        switch self {
        case .notFound(let id):
            return "用户 \(id) 未找到"
        case .invalidData:
            return "无效的数据格式"
        case .networkError(let error):
            return "网络错误: \(error.localizedDescription)"
        }
    }
}
```

## 6. 并发编程规范
### Swift
```swift
// 使用 async/await
func fetchUserData() async throws -> User {
    let data = try await networkService.fetch(endpoint: .user)
    return try JSONDecoder().decode(User.self, from: data)
}

// 使用 Actor 保护共享状态
actor UserManager {
    private var users: [Int: User] = [:]

    func getUser(id: Int) -> User? {
        users[id]
    }
}
```

### Objective-C
```objc
// 使用 GCD
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    // 异步操作
    dispatch_async(dispatch_get_main_queue(), ^{
        // UI 更新
    });
});
```

## 7. 性能优化

- 使用 Instruments 进行性能分析
- 避免主线程阻塞
- 图片缓存和懒加载
- 视图重用
- 合理使用内存

## 8. 安全规范

- 使用 Keychain 存储敏感数据
- 实现适当的数据加密
- 网络安全（证书固定、HTTPS）
- 输入验证和清理
- 避免硬编码敏感信息

## 9. 文档规范
- 使用 DocC 生成文档
- 编写详细的 README.md
- 维护 CHANGELOG.md
- 示例代码和使用说明

## 10. 本地化规范
```swift
// 使用结构化字符串文件
"welcome.message" = "欢迎，%@";

// 代码中使用
let message = String(format: NSLocalizedString("welcome.message", comment: ""), userName)
```

## 11. 辅助功能支持
- VoiceOver 支持
- 动态字体大小
- 颜色对比度
- 辅助功能标签和提示

## 12. 最佳实践
- 遵循 SOLID 原则
- 使用 Swift 的现代特性
- 代码复用和模块化
- 适当的设计模式
- 持续重构和优化
- 编写可测试的代码 
```
```

## 2.7 Kotlin编码规范

```
```
# Kotlin 项目开发规范

## 版本规范
- 使用 Kotlin 1.8+ 版本
- 使用 Gradle 7.0+ 构建工具
- 支持 Java 11+ 运行时
- 使用 Kotlin DSL 配置构建脚本

## 代码风格规范

### 命名规范

```kotlin
// 类名：使用大驼峰命名法
class UserService {
    // ...
}

// 接口名：使用大驼峰命名法
interface UserRepository {
    fun findById(id: Long): User?
}

// 函数名：使用小驼峰命名法
fun processUserData(user: User): Result<ProcessedData> {
    // ...
}

// 变量名：使用小驼峰命名法
val userName = "张三"
var userAge = 25

// 常量：使用大写字母和下划线
const val MAX_RETRY_COUNT = 3

// 伴生对象常量
companion object {
    const val DEFAULT_TIMEOUT = 30L
}

// 数据类：使用大驼峰命名法
data class UserProfile(
    val id: Long,
    val name: String,
    val email: String
)
```

### 注释规范
```kotlin
/**
 * 用户服务类，处理用户相关的业务逻辑
 *
 * @property repository 用户数据仓库
 * @property logger 日志记录器
 */
class UserService(
    private val repository: UserRepository,
    private val logger: Logger
) {
    /**
     * 处理用户数据并返回结果
     *
     * @param userId 用户ID
     * @param data 原始数据
     * @return 处理后的数据
     * @throws UserNotFoundException 当用户不存在时抛出
     */
    fun processUserData(userId: Long, data: Data): ProcessedData {
        // ...
    }
}
```

## 项目结构规范
推荐的项目结构：
```
project_name/
├── app/                      # 应用模块
│   ├── src/
│   │   ├── main/
│   │   │   ├── kotlin/
│   │   │   ├── res/
│   │   │   └── AndroidManifest.xml
│   │   └── test/
│   └── build.gradle.kts
├── domain/                   # 领域模块
│   ├── src/
│   │   ├── main/
│   │   │   └── kotlin/
│   │   └── test/
│   └── build.gradle.kts
├── data/                     # 数据模块
│   ├── src/
│   │   ├── main/
│   │   │   └── kotlin/
│   │   └── test/
│   └── build.gradle.kts
├── buildSrc/                 # 构建配置
│   └── src/main/kotlin/
├── gradle/
├── build.gradle.kts
├── settings.gradle.kts
└── README.md
```

## 依赖管理
### Gradle 配置（Kotlin DSL）
```kotlin
// build.gradle.kts
plugins {
    kotlin("jvm") version "1.8.0"
    kotlin("kapt") version "1.8.0"
}

dependencies {
    implementation(kotlin("stdlib"))
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.6.4")

    // 测试依赖
    testImplementation("org.junit.jupiter:junit-jupiter:5.8.2")
    testImplementation("io.mockk:mockk:1.12.0")
}
```

## 协程使用规范
```kotlin
class UserService(
    private val repository: UserRepository,
    private val scope: CoroutineScope
) {
    // 使用结构化并发
    fun processUsers() = scope.launch {
        supervisorScope {
            val users = repository.getAllUsers()
            users.forEach { user ->
                launch {
                    processUser(user)
                }
            }
        }
    }

    // 使用流
    fun getUserUpdates(): Flow<User> = flow {
        while (true) {
            val users = repository.getUpdatedUsers()
            users.forEach { emit(it) }
            delay(1000)
        }
    }
}
```

## 依赖注入规范
### 使用 Hilt（Android）
```kotlin
@HiltAndroidApp
class MyApplication : Application()

@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    @Provides
    @Singleton
    fun provideUserRepository(): UserRepository {
        return UserRepositoryImpl()
    }
}

@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    @Inject
    lateinit var userService: UserService
}
```

### 使用 Koin（纯 Kotlin）
```kotlin
val appModule = module {
    single<UserRepository> { UserRepositoryImpl() }
    factory { UserService(get()) }
}

fun main() {
    startKoin {
        modules(appModule)
    }
}
```

## 性能优化
- 使用协程处理并发
- 合理使用挂起函数
- 避免不必要的对象创建
- 使用序列处理大数据集
- 使用 Flow 处理数据流

```kotlin
// 使用序列
fun processLargeData() = sequence {
    for (item in largeDataSet) {
        yield(processItem(item))
    }
}.filter { it.isValid }
 .map { it.toResult() }
 .toList()
```

## 安全规范
- 使用密封类处理状态
- 避免使用 !!
- 正确处理空值
- 使用 Result 类型处理错误
- 实现适当的数据加密

## CI/CD 配置
在 `.github/workflows/kotlin-ci.yml` 中配置：
- 代码检查（ktlint）
- 单元测试
- 构建检查
- 自动部署

## 文档规范
- 使用 KDoc 生成文档
- 编写详细的 README.md
- 维护 CHANGELOG.md
- 示例代码和使用说明

## 最佳实践
- 使用数据类
- 使用密封类
- 使用扩展函数
- 使用委托属性
- 使用 DSL 构建器
- 使用 Kotlin 特有功能

## 多平台开发（KMP）
- 共享业务逻辑
- 平台特定实现
- 依赖管理
- 测试策略
```
```

# 三、框架规范

框架规范是三个层级规范里颗粒度最小的规范，主要还是看大家对项目细节把控的需求有没有到这种程度。这里同样整理了几种常见框架的规范：

## 3.1 React框架规范

```
```
# React 开发规范

## 组件开发规范

### 组件分类
- 展示组件：只负责渲染UI，不包含业务逻辑
- 容器组件：处理数据和业务逻辑，可以包含展示组件
- 页面组件：路由级别的组件，整合多个容器组件

### 组件设计原则
- 单一职责：每个组件只做一件事
- 组件尽量保持纯函数
- 避免过深的组件嵌套（不超过3层）
- 合理拆分组件（单文件不超过300行）
- 组件通信优先使用 props，避免过度使用 Context
- 及时清理副作用，防止内存泄漏

## Hooks 使用规范

### 核心规则
- 只在最顶层使用 Hooks
- 只在 React 函数组件和自定义 Hooks 中使用 Hooks
- 自定义 Hooks 必须以 "use" 开头
- 一个 Hook 只做一件事，保持功能单一

### 常见场景
```tsx
// 状态管理：简单状态用 useState，复杂状态用 useReducer
const [user, setUser] = useState(null);
const [state, dispatch] = useReducer(reducer, initialState);

// 副作用处理：useEffect 依赖项要完整，注意清理
useEffect(() => {
  const subscription = api.subscribe(data => {
    // 处理数据
  });
  return () => subscription.unsubscribe(); // 清理副作用
}, [必要的依赖项]);

// 性能优化：适时使用 useMemo 和 useCallback
const memoizedValue = useMemo(() => computeExpensive(a), [a]);
const memoizedCallback = useCallback(() => doSomething(a), [a]);
```

### 自定义 Hooks 原则
- 抽象可复用的状态逻辑
- 保持简单和可测试性
- 明确的命名和文档
- 处理加载和错误状态

## 状态管理规范

### 原则
- 能用组件状态就不用全局状态
- 多组件共享状态才使用全局状态
- 遵循单向数据流

### 状态管理方案选择
```tsx
// 1. 组件内状态：适用于组件内部的简单状态
const [count, setCount] = useState(0);

// 2. Context：适用于中等规模的状态共享，注意控制范围避免过度渲染
const UserContext = createContext(null);

// 3. Redux/Mobx：适用于以下场景
// - 大规模的状态管理
// - 复杂的状态逻辑
// - 需要状态持久化
// - 需要状态回溯
```

## 性能优化关键点

- 使用 React.memo() 避免不必要的重渲染
- 大列表使用虚拟滚动
- 路由懒加载
- 合理使用 useMemo 和 useCallback
- 避免在渲染期间进行复杂计算
- 及时清理不需要的监听器和定时器

## 常见错误处理

```tsx
// 使用错误边界捕获渲染错误
<ErrorBoundary fallback={<ErrorUI />}>
  <MyComponent />
</ErrorBoundary>

// 异步错误处理
try {
  await api.getData();
} catch (error) {
  // 区分错误类型，给出合适的用户提示
  if (error instanceof NetworkError) {
    showNetworkError();
  } else {
    showGeneralError();
  }
}
```

## 最佳实践

- 优先使用函数组件和 Hooks
- 及时清理副作用
- 避免过度优化
- 保持组件的可测试性
- 使用 TypeScript 增强代码健壮性
- 遵循 React 的设计理念，保持单向数据流
- 合理使用 React 开发者工具进行调试 
```
```

## 3.2 Tailwind CSS框架规范

```
```
# Tailwind CSS开发规范

## 基本原则
- **实用优先**：使用Tailwind的工具类构建UI，避免编写自定义CSS
- **响应式设计**：一致使用Tailwind的响应式前缀实现多设备适配
- **组件抽象**：将重复的模式抽象为可复用组件
- **设计系统一致**：使用项目的设计令牌保持视觉一致性
- **最小定制**：优先使用Tailwind默认配置，仅在必要时定制

## 项目设置
### 安装与配置
1. **基础安装**：
   ```bash
   npm install -D tailwindcss postcss autoprefixer
   npx tailwindcss init -p

2. **配置文件设置**：
   ```js
   // tailwind.config.js
   module.exports = {
     content: [
       "./src/**/*.{js,jsx,ts,tsx}",
       "./public/index.html",
     ],
     theme: {
       extend: {
         // 项目特定的扩展
         colors: {
           'primary': '#3490dc',
           'secondary': '#ffed4a',
           'danger': '#e3342f',
         },
       },
     },
     plugins: [],
   }

3. **引入Tailwind指令**：
   ```css
   /* src/index.css */
   @tailwind base;
   @tailwind components;
   @tailwind utilities;

### 生产优化
1. **PurgeCSS配置**：
   - 确保`content`配置正确以移除未使用的样式
   - 避免动态类名生成，影响优化效果

2. **构建优化**：
   ```js
   // tailwind.config.js
   module.exports = {
     // ...其他配置
     // 仅在生产环境启用最小化
     minify: process.env.NODE_ENV === 'production',
   }

## 类名使用规范

### 命名约定
1. **HTML结构与类名**：
   - 类名保持简洁，遵循Tailwind的语法
   - 按逻辑顺序组织类名（布局→间距→尺寸→外观→状态）
   - 长类名列表考虑使用多行格式提高可读性

2. **响应式设计**：
   - 使用内置的响应式前缀（sm、md、lg、xl、2xl）
   - 默认情况下为移动设备优先，然后添加更大屏幕的样式
   - 响应式前缀放在相关功能类前面

3. **状态变体**：
   - 使用状态前缀（hover、focus、active、disabled等）
   - 状态前缀放在响应式前缀之后，功能类之前
   - 可以组合响应式和状态前缀

### 组织与简化
1. **使用@apply抽取组件样式**：
   - 对频繁使用的组合使用`@apply`
   - 在组件文件或集中的样式文件中定义
   - 避免过度使用，保持实用优先的理念

2. **提取反复使用的模式**：
   - 使用CSS组件层提取复用模式
   - 在Tailwind配置中注册常用组件模式

## 主题与设计系统

### 颜色系统
1. **定义品牌颜色**：
   - 在Tailwind配置中扩展默认调色板
   - 使用有意义的名称（避免color-1，color-2这类命名）
   - 保持颜色系统的一致性和层次感

2. **使用语义化颜色变量**：
   - 基于UI功能而非视觉定义颜色
   - 便于全局主题更改和暗模式适配

### 响应式设计

1. **断点使用规范**：
   - 默认Tailwind断点：sm(640px)、md(768px)、lg(1024px)、xl(1280px)、2xl(1536px)
   - 根据项目需求调整或扩展断点

2. **容器使用**：
   - 使用Tailwind的container类控制内容宽度
   - 根据设计需求定制容器行为

2. **布局组件**：
   - 创建常用的布局模式组件
   - 使用Tailwind的网格和Flex布局

### 组件组织

1. **组件库结构**：
   - 按功能分类组织组件
   - 创建组件变体而非重复组件
   - 实现组件文档和示例 ```
   src/
   ├── components/
   │   ├── ui/              # 基础UI组件
   │   │   ├── Button.jsx
   │   │   ├── Card.jsx
   │   │   └── Input.jsx
   │   ├── layout/          # 布局组件
   │   │   ├── Container.jsx
   │   │   ├── Grid.jsx
   │   │   └── Stack.jsx
   │   ├── data/            # 数据展示组件
   │   │   ├── Table.jsx
   │   │   ├── List.jsx
   │   │   └── Chart.jsx
   │   └── forms/           # 表单组件
   │       ├── TextField.jsx
   │       ├── Select.jsx
   │       └── Checkbox.jsx


2. **组件API设计**：
   - 保持简单一致的API
   - 使用清晰的属性名
   - 提供默认值和适当的类型检查

## 响应式设计最佳实践

1. **移动优先设计**：
   - 默认样式为移动设备设计
   - 使用响应式前缀为更大屏幕添加样式
   - 避免为移动设备使用复杂布局

2. **响应式导航**：
   - 实现小屏幕的汉堡菜单或抽屉式导航
   - 大屏幕显示完整导航栏
   - 使用状态管理响应式交互

## 可访问性
1. **颜色对比度**：
   - 确保文本和背景颜色符合WCAG标准（至少4.5:1）
   - 使用Tailwind的颜色级别系统选择适当对比度

2. **键盘可访问性**：
   - 使用Tailwind的focus变体添加焦点样式
   - 确保所有可交互元素可以通过键盘访问

3. **屏幕阅读器支持**：
   - 使用sr-only类隐藏视觉元素但保持屏幕阅读器可访问
   - 添加适当的ARIA属性

## 性能优化
1. **CSS文件大小优化**：
   - 使用PurgeCSS移除未使用的样式
   - 避免使用@apply生成过多自定义类
   - 生产环境中压缩CSS文件

2. **渲染性能**：
   - 谨慎使用复杂的CSS动画或过渡
   - 优先使用transform和opacity进行动画
   - 考虑使用will-change属性提示浏览器

## 与JS框架集成

1. **React集成**：
   - 使用`clsx`或`classnames`库动态组合类名
   - 结合Styled Components或Emotion使用twin.macro
   - 使用组件属性控制样式变体

2. **Vue集成**：
   - 使用`:class`绑定动态应用类
   - 在组件中使用计算属性组合类名
```
```

## 3.3 VUE框架规范

```
```
# Vue 开发规范

## 版本选择

- 新项目应使用 Vue 3，除非有特殊兼容性要求
- 使用 Composition API 进行开发
- 使用 Vue Router 4.x 和 Pinia (或 Vuex 4.x) 搭配 Vue 3
- 使用官方支持的构建工具：Vite 或 Vue CLI
- 使用 TypeScript 提高代码健壮性

## 项目结构

推荐的项目结构：

```
vue-project/
├── public/                 # 静态资源
├── src/
│   ├── assets/            # 项目资源文件
│   │   ├── base/          # 基础组件
│   │   └── common/        # 业务通用组件
│   ├── composables/       # 可复用的组合式函数
│   ├── directives/        # 自定义指令
│   ├── router/            # 路由配置
│   ├── stores/            # Pinia/Vuex状态管理
│   ├── utils/             # 工具函数
│   ├── views/             # 页面级组件
│   ├── App.vue            # 根组件
│   └── main.js/ts         # 入口文件
├── tests/                  # 测试文件
├── .eslintrc.js           # ESLint配置
├── .prettierrc            # Prettier配置
├── vite.config.js/ts      # Vite配置
└── package.json
```

## 组件规范

### 组件命名

- 组件文件名使用 PascalCase（大驼峰）
- 基础组件以 Base 前缀命名
- 单例组件以 The 前缀命名
- 紧密耦合的组件使用父组件名作为前缀

### 组件结构

- 使用单文件组件 (SFC) 格式
- `<script setup>` 声明放在顶部
- 按照模板、脚本、样式的顺序组织代码
- 使用语义化的模板结构

### Props 定义

- 始终使用详细的 props 定义，包括类型和默认值
- 明确指定必选项和可选项
- 使用驼峰命名法定义 prop，使用短横线分隔法在模板中使用
- 提供适当的验证器

## Composition API 使用规范

### 可组合函数 (Composables)

- 将可复用逻辑抽象为可组合函数
- 可组合函数文件名使用 camelCase，以 `use` 开头
- 遵循关注点分离原则
- 所有可组合函数应当是纯函数，不应有副作用

### 生命周期钩子使用

- 避免过度使用生命周期钩子
- 在 `setup` 中的副作用应当在适当的生命周期钩子中处理
- 在 `onUnmounted` 中清理资源和事件监听器
- 避免在 `onMounted` 中执行复杂操作，考虑使用异步加载

## 路由配置
- 路由配置应使用懒加载
- 使用命名路由和命名视图
- 使用路由元信息管理权限和页面特性
- 使用路由守卫处理导航逻辑

## 状态管理
### Pinia (推荐)

- 使用 Pinia 作为首选状态管理库
- 按照领域模型或功能模块组织 Store
- 使用 TypeScript 定义 Store 类型
- 保持 Store 简洁，避免过度集中状态

### Vuex (替代方案)
- 按模块组织状态
- 遵循 Vuex 的单向数据流
- 使用 actions 处理异步操作
- 避免在组件中直接修改状态

## 样式规范
- 优先使用 scoped CSS
- 考虑使用 CSS Modules 或 CSS-in-JS 方案
- 遵循 BEM 命名约定
- 使用 SCSS/LESS 等预处理器管理样式
- 使用变量管理颜色、字体等

## 性能优化

- 使用 `v-memo` 缓存模板部分
- 避免不必要的组件重渲染
- 使用 `shallowRef` 处理大型非响应式对象
- 使用 `v-once` 渲染静态内容
- 异步组件和路由懒加载
- Keep-alive 缓存适当的组件
- 谨慎使用计算属性和侦听器，避免过度使用

## 测试
- 单元测试使用 Vitest 或 Jest
- 组件测试使用 Vue Test Utils
- 端到端测试使用 Cypress 或 Playwright
- 为关键组件编写测试用例
- 使用快照测试验证组件渲染

## 部署和构建
- 使用环境变量管理不同环境的配置
- 配置合理的构建优化
- 使用现代模式构建
- 分析并优化打包体积
- 配置适当的缓存策略

## 最佳实践
- 避免过度嵌套的组件结构
- 注重组件的职责划分和重用
- 使用 TypeScript 提高代码健壮性
- 遵循 Vue 官方风格指南
- 定期更新 Vue 及其生态系统库
- 使用 ESLint 和 Prettier 保持代码质量
- 参考 Vue 官方文档中的最佳实践 
```
```

## 3.4 Flutter框架规范

```
```
# Flutter 开发规范

## Widget 开发规范

### Widget 分类
- StatelessWidget：无状态组件，用于纯展示UI
- StatefulWidget：有状态组件，处理动态数据和交互
- 混合Widget：组合多个Widget形成完整页面

### Widget 设计原则
- 单一职责：每个Widget只做一件事
- 优先使用 StatelessWidget
- 避免Widget嵌套过深（建议不超过5层）
- 合理拆分Widget（单文件不超过300行）
- 正确使用 const 构造函数优化性能
- 及时释放资源，防止内存泄漏

## 状态管理规范

### 核心规则
- 明确区分本地状态和全局状态
- 选择合适的状态管理方案
- 遵循单向数据流
- 避免状态冗余

### 状态管理方案选择
```dart
// 1. setState：适用于简单的组件内状态
setState(() {
  _counter++;
});

// 2. InheritedWidget/Provider：适用于以下场景
// - 跨组件状态共享
// - 中等规模的状态管理
// 示例：
class MyProvider extends ChangeNotifier {
  int _count = 0;
  int get count => _count;

  void increment() {
    _count++;
    notifyListeners();
  }
}

// 3. GetX/Bloc/Riverpod：适用于以下场景
// - 大规模状态管理
// - 复杂的状态逻辑
// - 需要状态隔离
// - 需要依赖注入
```

## 生命周期管理

### StatefulWidget 生命周期
```dart
class MyWidget extends StatefulWidget {
  @override
  State<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  @override
  void initState() {
    super.initState();
    // 初始化操作
  }

  @override
  void dispose() {
    // 清理资源
    super.dispose();
  }

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    // 依赖变化处理
  }
}
```

### 资源管理原则
- 及时释放控制器（Controller）
- 取消订阅和监听
- 关闭流（Stream）
- 释放动画资源

## 性能优化关键点

- 合理使用 const Widget
- 使用 ListView.builder 处理长列表
- 图片缓存和预加载
- 避免在build方法中进行复杂计算
- 使用 RepaintBoundary 隔离重绘区域
- 合理使用 GlobalKey

## 路由导航规范
```dart
// 命名路由
MaterialApp(
  initialRoute: '/',
  routes: {
    '/': (context) => HomeScreen(),
    '/detail': (context) => DetailScreen(),
  },
);

// 参数传递
Navigator.pushNamed(
  context,
  '/detail',
  arguments: {'id': 1},
);

// 自定义路由过渡动画
Navigator.push(
  context,
  CustomPageRoute(
    builder: (context) => DetailScreen(),
  ),
);
```

## 最佳实践
- 遵循 Flutter 的设计理念和风格指南
- 使用 Flutter Lint 规范代码
- 保持 Widget 的可测试性
- 合理组织代码结构
- 优先使用 Flutter 原生 Widget
- 遵循平台特定的设计规范
- 合理使用 DevTools 进行调试和性能分析
```
```

## 3.5 微信小程序开发规范

```
```
# 小程序开发规范

## 适用范围

本规范适用于以下小程序开发：
- 微信小程序
- 支付宝小程序
- 百度小程序
- 字节跳动小程序
- QQ小程序
- 跨平台框架（如 Taro、uni-app、mpvue）

## 项目结构

### 基础目录结构

```
miniprogram/
├── components/         # 自定义组件
├── pages/              # 页面文件
├── images/             # 图片资源
├── styles/             # 公共样式
├── utils/              # 工具函数
├── services/           # 网络请求和业务逻辑
├── config/             # 配置文件
├── app.js              # 应用逻辑
├── app.json            # 全局配置
├── app.wxss            # 全局样式
└── project.config.json # 项目配置
```

### 跨平台项目结构（Taro/uni-app）

```
miniprogram/
├── src/
│   ├── components/     # 自定义组件
│   ├── pages/          # 页面文件
│   ├── assets/         # 静态资源
│   │   ├── images/     # 图片资源
│   │   └── styles/     # 样式资源
│   ├── utils/          # 工具函数
│   ├── services/       # 接口服务
│   ├── config/         # 配置文件
│   ├── app.js/ts       # 应用入口
│   ├── app.config.js   # 应用配置
│   └── app.scss        # 全局样式
├── config/             # 编译配置
├── dist/               # 构建产物
└── package.json        # 项目配置
```

## 文件命名规范

- 页面文件夹：小写字母 + 短横线（kebab-case）
  ```
  pages/
  ├── home/
  ├── user-profile/
  ├── order-detail/
  ```

- 组件文件夹：大驼峰命名（PascalCase）
  ```
  components/
  ├── SearchBar/
  ├── ProductCard/
  ├── UserAvatar/
  ```

- JS/TS 工具文件：小驼峰命名（camelCase）
  ```
  utils/
  ├── httpRequest.js
  ├── dateFormatter.js
  ├── localStorage.js
  ```

## 代码规范

### JS/TS 规范

- 使用 ES6+ 语法，推荐使用 TypeScript
- 遵循[JavaScript 规范](../编程语言规则/JAVASCRIPT_TYPESCRIPT_GUIDELINES.md)中的基本要求
- 特别注意小程序环境的限制（如不支持某些 ES6+ 特性）

### WXML/AXML（模板）规范

- 标签名使用小写字母
- 属性名使用小驼峰（bindtap, catchtap）
- 适当缩进，保持结构清晰

### WXSS/ACSS（样式）规范

- 使用 rpx 作为尺寸单位，适配不同屏幕
- 使用类选择器，避免使用 ID 选择器和标签选择器
- 遵循 BEM 命名规范
- 避免使用 !important

## 组件开发规范

### 组件设计原则

- 单一职责：一个组件只负责一个功能
- 可复用性：组件设计应考虑复用
- 可维护性：避免过于复杂的组件
- 适当封装：内部实现细节对外部透明

### 组件通信

- 属性传递（props）：父组件向子组件传递数据
- 事件通信：子组件向父组件传递事件
- 全局状态管理：使用 globalData 或状态管理库

## 页面开发规范

### 页面结构

- 采用逻辑、视图、样式分离的架构
- 页面生命周期函数按顺序排列
- 相关函数放在一起
- 复杂页面拆分为多个组件

### 数据管理

- 避免过大的 data 对象
- 只存储页面所需的数据
- 使用 setData 更新视图，避免频繁调用
- 可考虑使用第三方状态管理（如 mobx-miniprogram）

## 网络请求规范

### 请求封装

- 统一封装网络请求方法
- 处理通用错误和重试逻辑
- 管理请求头和认证信息
- 支持请求拦截和响应拦截

```

### API 服务

- 按照业务领域划分 API 服务
- 解耦数据获取和业务逻辑
- 方便添加缓存和数据转换

## 性能优化

### 启动性能

- 减少启动时网络请求数量
- 核心功能优先加载，次要功能懒加载
- 优化 app.js 中的同步逻辑
- 注意分包配置，控制主包大小

### 渲染性能

- 避免频繁的 setData
- 长列表使用虚拟列表
- 合理使用 wx:if 和 wx:hidden
- 减少不必要的视图层通信

### 资源管理

- 图片资源压缩和优化
- 合理使用云存储和 CDN
- 缓存不经常变化的数据
- 注意小程序包大小限制

## 安全规范

- 敏感信息加密存储
- 避免在本地存储重要用户数据
- 防范 XSS 和注入攻击
- 遵循最小权限原则申请用户授权
- 服务端校验所有客户端数据

## 调试和测试

- 使用小程序开发者工具进行调试
- 编写单元测试和集成测试
- 使用模拟数据进行开发和测试
- 多设备和多环境测试
- 性能和安全测试

## 发布和版本控制

- 遵循语义化版本规范
- 使用小程序的条件编译功能
- 不同环境（开发、测试、生产）配置分离
- 使用体验版进行内部测试
- 制定合理的更新策略

## 跨平台开发建议
### Taro 开发规范
- 遵循 React/Vue 开发规范
- 注意各平台差异和兼容性处理
- 使用条件编译处理平台特定逻辑

### uni-app 开发规范

- 遵循 Vue 开发规范
- 使用条件编译
- 注意平台差异和 API 兼容性

## 多端兼容策略

- 保持核心功能一致性
- 平台特定功能通过条件编译实现
- 使用抽象层处理平台差异
- 持续监控各平台兼容性问题
- 测试覆盖所有目标平台

## 最佳实践

- 保持小程序版本更新
- 定期重构和优化代码
- 关注官方文档和最新特性
- 重视用户体验和性能
- 开发前充分了解平台限制和特性 
```
```

## 3.6 FastAPI框架规范

```
```
# FastAPI开发规范

## 基本原则

- **性能优先**：利用FastAPI的高性能特性
- **类型安全**：充分利用Python类型提示
- **自动文档**：设计良好的API自动生成完整文档
- **标准化**：遵循RESTful API设计原则
- **可维护性**：组织代码结构便于扩展和维护

## 项目结构

### 推荐项目布局

```
project_name/                  # 项目根目录
├── app/                       # 应用代码
│   ├── __init__.py
│   ├── main.py                # 应用入口点
│   ├── core/                  # 核心模块
│   │   ├── __init__.py
│   │   ├── config.py          # 配置设置
│   │   ├── security.py        # 安全相关
│   │   └── logging.py         # 日志配置
│   ├── api/                   # API路由
│   │   ├── __init__.py
│   │   ├── deps.py            # 依赖项（如获取DB会话）
│   │   ├── v1/                # API版本1
│   │   │   ├── __init__.py
│   │   │   ├── endpoints/     # API端点
│   │   │   │   ├── __init__.py
│   │   │   │   ├── users.py
│   │   │   │   └── items.py
│   │   │   └── router.py      # V1 API路由
│   │   └── base.py            # API基础设置
│   ├── models/                # 数据模型
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── item.py
│   ├── schemas/               # Pydantic模型
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── item.py
│   ├── crud/                  # CRUD操作
│   │   ├── __init__.py
│   │   ├── base.py
│   │   ├── user.py
│   │   └── item.py
│   ├── db/                    # 数据库
│   │   ├── __init__.py
│   │   ├── base.py
│   │   └── session.py
│   └── utils/                 # 工具函数
│       ├── __init__.py
│       └── helpers.py
├── migrations/                # Alembic迁移
│   ├── versions/
│   ├── env.py
│   └── alembic.ini
├── tests/                     # 测试
│   ├── __init__.py
│   ├── conftest.py
│   ├── test_api/
│   └── test_utils/
├── .env                       # 环境变量
├── requirements.txt           # 依赖项
├── Dockerfile                 # Docker配置
└── README.md                  # 项目文档
```

## 编码规范

### 代码风格

1. **PEP 8**：
   - 遵循[PEP 8](https://peps.python.org/pep-0008/)规范
   - 使用`black`和`isort`格式化代码
   - 使用`flake8`或`pylint`进行代码检查

2. **导入顺序**：
   ```python
   # 标准库
   import os
   from typing import List, Optional

   # 第三方库
   from fastapi import FastAPI, Depends, HTTPException
   from sqlalchemy.orm import Session

   # 本地模块
   from app.db.session import get_db
   from app.models.user import User
   from app.schemas.user import UserCreate, UserResponse
   ```

3. **类型提示**：
   - 为所有函数参数和返回值添加类型注解
   - 使用`typing`模块中的类型
   - 使用`Optional`标记可选参数 ```python
   def get_user_by_id(user_id: int, db: Session) -> Optional[User]:
       return db.query(User).filter(User.id == user_id).first()
   ```

## API设计

### 路由组织

1. **路由分组**：
   - 按资源或功能组织路由
   - 使用APIRouter创建模块化路由
   - 支持版本化API

2. **URL命名**：
   - 使用名词表示资源，复数形式表示集合
   - 使用连字符（不用下划线或驼峰）
   - 保持URL简短明了

3. **HTTP方法**：
   - GET：获取资源
   - POST：创建资源
   - PUT：完全替换资源
   - PATCH：部分更新资源
   - DELETE：删除资源

### 请求和响应模型

1. **Pydantic模型**：
   - 为每个请求和响应定义专用模型
   - 使用继承减少重复定义
   - 实现自定义验证逻辑

2. **响应模型配置**：
   - 使用`response_model`参数指定响应格式
   - 使用`response_model_exclude_unset=True`排除未设置的默认值
   - 使用`orm_mode=True`支持ORM模型转换

3. **状态码**：
   - 使用`status_code`参数指定响应状态码
   - 遵循HTTP标准状态码

## 依赖注入

1. **依赖函数**：
   - 创建可复用的依赖函数
   - 集中定义在`deps.py`文件中
   - 使用`Depends()`注入依赖

2. **依赖嵌套**：
   - 构建依赖链实现复杂验证
   - 分离关注点提高代码复用

3. **路由依赖**：
   - 路由级别依赖用于整个路由的认证或权限检查
   - 在APIRouter创建时指定

## 数据库交互

1. **ORM使用**：
   - 使用SQLAlchemy ORM
   - 基础CRUD操作封装为通用类
   - 创建专用CRUD类处理业务逻辑

2. **会话管理**：
   - 使用依赖注入提供数据库会话
   - 确保会话正确关闭

3. **事务处理**：
   - 使用上下文管理器处理事务
   - 异常时回滚事务

## 认证与授权

1. **JWT认证**：
   - 使用`python-jose`处理JWT令牌
   - 实现令牌生成和验证
   - 使用依赖注入进行认证

2. **密码处理**：
   - 使用`passlib`和`bcrypt`处理密码哈希
   - 实现密码验证和重置

3. **权限控制**：
   - 实现基于角色的访问控制
   - 使用依赖注入检查权限

## 异常处理

1. **HTTP异常**：
   - 使用`HTTPException`抛出HTTP错误
   - 提供详细错误信息

2. **全局异常处理**：
   - 使用异常处理器统一处理异常
   - 自定义错误响应格式

3. **验证错误处理**：
   - 自定义验证错误响应
   - 提供友好错误信息

## 测试规范

1. **测试框架**：
   - 使用`pytest`作为测试框架
   - 使用`pytest-asyncio`测试异步代码
   - 使用`requests`或`httpx`测试HTTP端点

2. **测试夹具**：
   - 创建数据库和客户端夹具
   - 使用临时数据库进行测试
   - 实现测试用户和认证夹具

3. **编写测试**：
   - 每个端点编写至少一个测试
   - 测试成功和失败情况
   - 测试不同权限级别的访问

## 性能优化

1. **异步操作**：
   - 使用异步视图函数处理IO密集型操作
   - 使用异步ORM（如SQLAlchemy 1.4+的异步支持）
   - 使用异步HTTP客户端调用外部API

2. **背景任务**：
   - 使用`BackgroundTasks`处理非关键任务
   - 使用Celery处理复杂的后台任务

3. **缓存**：
   - 使用Redis缓存频繁访问的数据
   - 实现缓存装饰器

## 安全最佳实践
1. **API安全**：
   - 实现速率限制防止滥用
   - 设置CORS策略
   - 使用HTTPS

2. **输入验证**：
   - 使用Pydantic模型验证所有输入
   - 实现自定义验证器处理复杂规则
   - 定义严格的类型注解

3. **敏感数据保护**：
   - 不返回敏感字段（如密码哈希）
   - 使用环境变量存储密钥和密码
   - 实现数据访问审计
```
```

## 3.7 React Native 框架规范

```
```
# React Native 开发规范
## 组件开发规范
### 组件分类
- 函数组件（Function Components）：无状态组件，用于纯展示UI
- 类组件（Class Components）：有状态组件，处理动态数据和交互
- 高阶组件（HOC）：用于组件逻辑复用和增强
- 复合组件：组合多个组件形成完整页面

### 组件设计原则
- 单一职责：每个组件只做一件事
- 优先使用函数组件和 Hooks
- 避免组件嵌套过深（建议不超过5层）
- 合理拆分组件（单文件不超过300行）
- 使用 React.memo() 优化性能
- 及时清理副作用，防止内存泄漏

## 状态管理规范
### 核心规则
- 明确区分本地状态和全局状态
- 选择合适的状态管理方案
- 遵循单向数据流
- 避免状态冗余

## 生命周期管理
### 函数组件生命周期（Hooks）
### 资源管理原则
- 及时清理定时器和事件监听器
- 取消异步操作和订阅
- 释放动画资源
- 清理 Native 模块引用

## 性能优化关键点
- 合理使用 React.memo() 和 useMemo()
- 使用 FlatList/VirtualizedList 处理长列表
- 图片懒加载和缓存策略
- 避免在渲染函数中进行复杂计算
- 使用 PureComponent 或 shouldComponentUpdate
- 合理使用 React.lazy() 和 Suspense 实现代码分割

## 最佳实践
- 遵循 React Native 的设计理念和风格指南
- 使用 ESLint 和 TypeScript 规范代码
- 保持组件的可测试性
- 合理组织代码结构
- 优先使用 React Native 原生组件
- 遵循平台特定的设计规范
- 合理使用开发者工具进行调试和性能分析

### 代码组织示例
```
src/
├── components/     # 可复用组件
├── screens/        # 页面
├── models/         # 数据模型
├── services/       # 服务
├── utils/          # 工具类
├── navigation/     # 导航配置
├── hooks/          # 自定义 Hooks
└── App.tsx         # 入口文件
```

### 平台适配规范
- 使用 Platform.select() 处理平台差异
- 合理使用 .ios.ts 和 .android.ts 文件
- 遵循各平台的设计规范和交互习惯
- 使用 react-native-safe-area-context 处理安全区域

### 原生模块集成规范
- 遵循模块化原则
- 做好错误处理和资源释放
- 提供清晰的 TypeScript 类型定义
- 考虑向后兼容性 
```
```