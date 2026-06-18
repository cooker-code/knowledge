> 已吸收至：[[02_Agent与AI工程/0206_AI编码方式/020603_SpecCode/020603_核心知识点/SpecCode规格驱动与验收边界|SpecCode规格驱动与验收边界]]
---
title: AI时代的Scrum：当一个人带领三个Agent,两天干完两周的活，用openspec还是superpower?
author: AI人工智能精益敏捷教练领导力
date:
url: https://mp.weixin.qq.com/s?__biz=MzIwNjkzNjk4Nw==&mid=2247483738&idx=1&sn=ab77c064ce647e37e125b7d553c9f837&chksm=96072d02a4b773246e840724a5a4e2f3eb91f37108e8756dab6d23ad9982c00a9475169a9abf&mpshare=1&scene=24&srcid=0307e9KrWTMs5ljbeYpRY6ml&sharer_shareinfo=a9d4293ff4f700c67abf647da6324545&sharer_shareinfo_first=a9d4293ff4f700c67abf647da6324545#rd
---

周五下午5点,产品经理在群里扔下一句话："能不能下周一给个收藏管理工具的Demo?"

以前我会骂娘。7个工程师的团队,一个需求评审会开2小时,再拆分任务到Jira,等前端和后端扯皮完接口规范,

下周一？

做梦。

但这次不一样。

周日晚上11点,我在终端里敲下最后一个命令:

```
git push origin feature/bookmark-system
```

三个Claude Code会话同时返回:

* Agent-Backend: "数据层测试覆盖率94.2%,已推送"
* Agent-Frontend: "搜索组件E2E测试全部通过,已推送"
* Agent-Test: "集成测试套件运行完毕,无失败用例"

我花了2天时间。准确说,是我花了4小时做决策,Agent们干了44小时的活。

周一早上9点,Demo准时上线。PO看完沉默了10秒,问了一句:"你周末加班了?"

我说:"没有,我带了三个新同事。"

这不是吹牛。这是2026年真实发生的事。关键是,我用的方法你也能复制。

(**字数统计**:约8200字，**阅读时间**:约18分钟)

---

## 产品的原始需求问题：找不到自己收藏的资料

说实话,这个工具最开始是为我自己做的。

作为一个重度AI用户,我收藏了大量好东西:

* 网上看到的各种优秀提示词模板
* 自己写的Claude Code Skills
* 技术文档的书签
* GitHub上看到的优秀项目

时间长了问题来了:

* 有些提示词在Notion笔记里
* 有些Skills装在本地`.claude/skills/`文件夹
* 有些是Chrome书签
* 有些是散落的markdown文件
* **我完全不记得自己装过哪些Skills**

最痛苦的是:如果要重装系统,我根本不知道会漏掉什么。

所以我需要一个工具:

1. 自动扫描本地的Skills和文档
2. 用LLM自动打标签分类
3. 支持模糊搜索快速定位
4. 能导出清单方便迁移

MVP阶段先做本地版本,云同步以后再说。

---

## 选工具:OpenSpec、SpecKit、Superpowers到底选哪个?

在开始前,我面临一个选择:用什么AI 框架来组织这个项目?

市面上主流的有三种SDD工具,它们的设计哲学完全不同:

### OpenSpec:任务清单式管理

目录结构:

```
project/
├── openspec/
│   ├── specs/              
│   │   ├── Product_Brief.md    # 产品愿景
│   │   ├── Architecture.md     # 技术架构
│   │   └── User_Stories.md     # 用户故事
│   ├── changes/            
│   │   ├── 001-add-search/     # 单个功能的提案
│   │   │   ├── proposal.md
│   │   │   ├── tasks.md
│   │   │   └── design.md
│   │   └── archive/            # 完成的功能
│   └── AGENTS.md           
└── src/                    
```

**核心思想**:像看板一样管理任务。Agent只看`changes/active/`,完成后归档到`archive/`。

**优势**:

* 极轻量,纯markdown
* 变更与长期规格分离
* 适合快速迭代的小项目

**劣势**:

* 缺少强制性约束
* 多人协作时容易混乱

### SpecKit:宪法式规范管理

目录结构:

```
project/
├── .speckit/
│   ├── memory/
│   │   └── constitution.md  # 团队"宪法"(不可违反)
│   └── specs/
│       └── 001-feature-name/
│           ├── spec.md
│           ├── requirements.md
│           ├── plan.md
│           └── tasks.md
└── src/
```

**核心思想**:先定"宪法",再干活。所有决策必须符合constitution.md的约束。

**优势**:

* 有强制性规范(constitution)
* 适合需要合规性的企业项目
* 阶段严格:specify → plan → implement

**劣势**:

* 需要Python环境,安装稍复杂
* 过于严格,不适合探索性开发

### Superpowers:技能包式增强

目录结构:

```
project/
├── docs/
│   ├── design/             # brainstorming技能生成
│   │   └── feature-x.md     
│   └── plans/              # writing-plans技能生成
│       └── feature-x.md     
├── skills/                 # 项目专属技能
│   └── our-standards/
│       └── SKILL.md
└── src/

# 全局技能(通过软链接)
~/.claude/skills/
├── superpowers/            
│   ├── brainstorming/
│   ├── writing-plans/  
│   ├── preserving-productive-tensions/
│   └── tdd/
└── my-custom-skills/       
```

**核心思想**:不是给Agent写文档,而是给它装"技能包"。每个技能是一套完整工作流。

**优势**:

* 强调工作流:brainstorm → design → plan → implement
* 技能可复用,全局共享
* 注重TDD和代码质量

**劣势**:

* 学习曲线陡峭
* 需要理解Skills的加载机制
* 初期配置较繁琐

### 我的选择:OpenSpec

为什么?

1. **需求会变**:这是个探索性项目,我不确定最终功能会长成什么样
2. **一个人开发**:不需要SpecKit那种多人协作的严格流程
3. **快速验证**:我要2天内出Demo,OpenSpec的轻量级最合适

安装超简单:

```
npm install -g @fission-ai/openspec
cd my-project
openspec init
```

选择Claude Code工具,目录自动生成好。

---

## Day 1上午：写需求规格,Agent一行代码都别碰

### 定义产品愿景

在`openspec/specs/Product_Brief.md`写下:

```
# SkillHub - 提示词与技能管理工具

## 问题陈述
- 提示词和Skills散落各处(Notion/本地文件/浏览器)
- 忘记安装过哪些Skills
- 缺少统一的标签和搜索
- 重装系统后不知道遗漏了什么

## 核心价值
统一管理所有AI相关资源,一键搜索,永不丢失。

## MVP范围
1. 扫描本地指定目录(如.claude/skills/)
2. 自动提取Skills的元信息(名称/描述)
3. 调用Claude API自动打标签
4. 提供搜索界面(模糊匹配)
5. 导出清单到JSON

## 不做(Out of Scope)
- 云端同步(v2.0考虑)
- 浏览器插件
- 移动端适配
```

### 设计技术架构

在`openspec/specs/Architecture.md`写下:

```
# 技术架构

## 技术选型

### 后端
- Node.js + Express
- SQLite (better-sqlite3)存储
- chokidar监听文件变化

### 前端
- Vite + React + TypeScript
- TailwindCSS
- Fuse.js做模糊搜索

### AI集成
- Anthropic API自动打标签
- 批量处理,控制token消耗

## 数据模型

技能表(skills):
- id: TEXT PRIMARY KEY
- name: TEXT                  
- description: TEXT
- path: TEXT                  
- tags: TEXT (JSON数组)
- created_at: INTEGER
- last_accessed: INTEGER

## 核心模块

1. FileScanner:扫描指定目录,解析SKILL.md
2. TagGenerator:调用Claude API生成标签
3. SearchEngine:基于Fuse.js的模糊搜索
4. API Layer:REST接口

## 接口设计

GET /api/skills - 获取所有技能
POST /api/skills/scan - 触发扫描
GET /api/skills/:id - 获取详情
PUT /api/skills/:id/tags - 更新标签
```

### 拆解用户故事

在`openspec/specs/User_Stories.md`写下:

```
# 用户故事

## Story 1:文件扫描
作为用户,我希望系统能自动扫描我的.claude/skills目录,
这样我就不用手动录入每个Skill了。

验收标准:
- [ ] 支持递归扫描子目录
- [ ] 解析SKILL.md的frontmatter提取name和description
- [ ] 将结果存入SQLite
- [ ] 扫描完成后在控制台打印统计信息

## Story 2:AI自动打标签
作为用户,我希望系统能自动给Skills打上合适的标签,
这样我可以快速按分类查找。

验收标准:
- [ ] 批量调用Claude API(每次最多10个Skill)
- [ ] 每个Skill生成3-5个标签
- [ ] 标签存为JSON数组
- [ ] 显示API调用进度

## Story 3:搜索界面
作为用户,我希望能通过关键词快速找到Skill,
即使拼写有误也能模糊匹配。

验收标准:
- [ ] 搜索框支持实时搜索
- [ ] 300ms防抖避免频繁请求
- [ ] 高亮匹配的关键词
- [ ] 支持按标签过滤
- [ ] 点击Skill显示完整路径
```

到这里,上午10点。我一行代码都没写,但**整个项目的"骨架"已经立起来了**。

---

## Day 1下午：让三个Agent各自打开"平行时空"

### 现在关键来了:**如何让三个Agent同时干活,互不干扰?**

传统方式的问题是什么?就像你在客厅写作业,突然被叫去厨房帮忙,回来发现作业本被弟弟画花了。

### **我需要给每个Agent一个独立的"房间"。**

Claude Code官方推荐的方案是:Git Worktrees(工作树)。

简单理解:

* 你有一个主项目文件夹(主卧室)
* 用worktree可以创建多个"镜像房间"
* 每个房间都是完整的项目副本
* 但它们共享同一套Git记录(就像共享一个仓库)

**最关键的是:这些镜像房间不会互相污染。**

Backend Agent在房间A改数据结构,Frontend Agent在房间B写界面,完全不会冲突。等大家都干完了,再统一合并。

我的操作:

给Backend创建专属房间: 项目主目录 → 创建skillhub-backend文件夹 → 对应backend分支

给Frontend创建专属房间: 项目主目录 → 创建skillhub-frontend文件夹 → 对应frontend分支

给Test创建专属房间: 项目主目录 → 创建skillhub-test文件夹 → 对应test分支

现在我的电脑上有四个文件夹:

* skillhub/ (主房间,main分支)
* skillhub-backend/ (Backend专属)
* skillhub-frontend/ (Frontend专属)
* skillhub-test/ (Test专属)

**这四个文件夹共享同一套Git历史记录,但文件内容完全独立。**

就像你有四套衣服放在四个衣柜里,但洗衣记录都记在同一本账本上。

### **Agent分配任务:关键是写清楚"工作手册"**

**Backend Agent的工作手册**

我在Backend的专属文件夹里创建了一个CLAUDE.md文件,这就是它的"工作手册":

**你负责什么**:文件扫描、数据存储、API接口

**技术约束**:

* 只用Node.js + SQLite,不要搞复杂的ORM
* API必须返回标准格式
* 禁止过度设计

**当前任务**:实现文件扫描和AI自动打标签功能

**完成标准**:

* 测试全部通过
* 覆盖率超过90%
* 在主项目的协作文档里记录你暴露的API接口

启动Backend Agent后,它就开始按照这个手册干活了。

**Frontend Agent的工作手册**

Frontend的手册稍有不同:

**你负责什么**:搜索界面,调用Backend的API

**技术约束**:

* React + TypeScript
* 只用TailwindCSS做样式
* 用Zustand管理状态

**依赖关系**:Backend暴露的API接口在主项目的协作文档里,先去读

**完成标准**:

* 前端能成功构建
* E2E测试通过
* 搜索响应快(200毫秒内)

**Test Agent的工作手册**

Test Agent最简单:

**你负责什么**:写集成测试,确保Backend和Frontend能协同工作

**测试场景**: 用户打开应用 → 点扫描按钮 → 等待完成 → 搜索"docx" → 验证结果 → 点击查看详情

**完成标准**:所有测试通过,生成测试报告

---

## Day 1晚上：三个Agent的"异步早会"和“看板”

传统的Scrum Daily Standup是这样的:

早上9点,团队站在一起:

* "小王,你昨天做了什么?"
* "小李,你遇到什么阻塞吗?"
* "大家今天的计划是?"

现在的"早会"是这样的:

晚上8点,我打开主项目的`INTEGRATION.md`:

```
# 集成状态 - 2026年2月8日 20:15

## Backend (Agent-Backend)
状态: IN_PROGRESS
分支: feature/backend-api
完成:
- [DONE] 文件扫描模块
- [DONE] SQLite数据库Schema
- [DOING] Claude API集成(预计22:00完成)

暴露接口:
- GET /api/skills - 返回所有技能
  返回格式: {success: true, data: Skill[]}
- POST /api/skills/scan - 触发扫描
  参数: {directory: string}
- POST /api/skills/generate-tags - 批量生成标签
  参数: {skillIds: string[]}

阻塞项: 无

## Frontend (Agent-Frontend)  
状态: BLOCKED
分支: feature/frontend-ui
完成:
- [DONE] 搜索框组件
- [BLOCKED] 等待Backend的GET /api/skills接口

阻塞原因: Backend的API还在开发中

计划: Backend完成后立即测试接口,预计今晚23:00前完成UI

## Test (Agent-Test)
状态: TODO
分支: feature/test-suite
等待: Backend和Frontend都ready后开始集成测试
```

看到Frontend被阻塞了。我在Backend的terminal执行:

```
# 检查Backend进度
cd ~/skillhub-backend
claude "INTEGRATION.md显示Frontend在等你的API,当前进度如何?"
```

Backend Agent回复:

```
API接口已完成,正在写单元测试。预计10分钟后推送到feature/backend-api分支。
已更新INTEGRATION.md标记为READY。
```

10分钟后,我在Frontend的terminal:

```
cd ~/skillhub-frontend
claude "Backend已就绪,读取最新的INTEGRATION.md,开始集成你的UI"
```

Frontend Agent立即开始工作。

**这就是新时代的"异步早会":不是人说给人听,而是Agent把状态写到共享文档,人类只需要协调阻塞点。**

---

## Day 2上午：让Agent写"工作日记"

第二天早上,我想知道三个Agent昨晚到底干了什么。

传统方式:打开每个分支,看代码diff,累死。

更聪明的方式:看Agent的Git commit记录。

**关键是让Agent写有意义的commit message。**

我给每个Agent的工作手册里加了一条规则:

每次提交代码时,commit message必须说明:

* 做了什么(标题)
* 为什么这么做(详细说明)

比如Backend Agent的一次提交:

```
集成Claude API自动打标签

使用批量处理方式,每次最多10个Skill,避免单次请求过大。
提示词要求返回JSON格式,方便解析。
添加了错误重试机制:失败后等2秒重试,最多3次。

选择批量而非单个调用是为了节省API次数。
```

这样我只需要看commit记录,就知道Agent为什么这么写代码。

比看代码本身还清楚。

### 防止Agent"偷懒"

我还配置了一个自动检查:

每次Agent要提交代码时,系统会自动检查:

1. commit message格式对不对
2. 如果是新功能,有没有写测试

**不符合规范,拒绝提交。**

这样Agent就不能随便写个"fix bug"这种没营养的message了。

---

## Day 2下午：三个分支合并,像拼拼图一样简单

下午3点,三个Agent都完成了各自的任务。

现在要把它们的工作合并到主分支。

### 传统合并的噩梦

以前合并代码是这样的:

* 把Backend分支合并进来 → 冲突了
* 手动解决冲突 → 改坏了
* 再合并Frontend分支 → 又冲突
* 再手动解决 → 又改坏了
* 最后跑测试 → 全挂了

**一个字:崩溃。**

### 现在的做法：让Integration Manager来干

我创建了第四个Agent:Integration Manager(集成经理)。

它的工作手册很简单:

**职责**:负责合并三个feature分支

**流程**:

1. 先合并Backend分支
2. 跑Backend的测试,确保通过
3. 再合并Frontend分支
4. 跑前端构建,确保无错
5. 最后合并Test分支
6. 跑完整E2E测试

**如果某个阶段失败**:

* 立即停止
* 在协作文档记录失败原因
* 通知对应的Agent修复

**成功标准**:

* 所有测试通过
* 代码覆盖率>85%
* 无TypeScript错误
* 无语法警告

我启动Integration Manager后,它开始自动工作:

```
[下午3:05] 正在合并Backend分支...
[下午3:06] Backend合并成功,运行测试...
[下午3:07] ✓ 32个测试全部通过

[下午3:07] 正在合并Frontend分支...
[下午3:08] Frontend合并成功,运行构建...
[下午3:09] ✓ 构建完成,无警告

[下午3:09] 正在合并Test分支...
[下午3:10] 运行E2E测试...
[下午3:13] ✓ 8个E2E场景全部通过

集成完成!代码覆盖率: 89.3%
已推送到主分支。
```

**整个过程全自动。我只是喝着咖啡看着。**

### Definition of Done也自动化了

以前的DoD(完成定义)是这样的:

```
完成标准:
□ 代码review通过
□ 单元测试通过
□ 集成测试通过  
□ 文档更新
```

现在DoD变成了可执行的检查清单:

Integration Manager会逐条检查:

* 测试全部通过?
* 覆盖率达标?
* 类型检查通过?
* 语法无警告?
* API文档更新了?
* commit记录规范?

**全部通过才算Done。**

不是人说了算,是系统说了算。

---

## 九、回到Scrum:本质没变,工具变了

很多人说"AI时代Agile过时了"。

错。

Scrum的核心是什么?

1. **小步迭代**:我们用2天完成一个MVP,就是迭代
2. **持续反馈**:Agent每次commit都有message,Integration Manager实时反馈
3. **透明可见**:INTEGRATION.md就是看板,所有人(Agent)都能看到当前状态
4. **自组织团队**:三个Agent各司其职,阻塞了自己在INTEGRATION.md标记

**这些原则一点都没变。**

变的是什么?

### 以前:人人协作

* Daily Standup:每天早上开会
* Sprint Planning:团队一起拆任务
* Sprint Review:演示给stakeholder
* Sprint Retrospective:回顾哪里做得好

**沟通成本极高。**

### 现在:人机协作

* Daily Sync:Agent更新INTEGRATION.md,人类看一眼
* Planning:人类写specs,Agent自己拆subtasks
* Review:直接看git log和测试报告
* Retro:分析CLAUDE.md哪里写得不清楚,导致Agent理解错了

**沟通成本接近零,但决策质量提高了。**

## Scrum Master还有必要吗？

我现在的角色不再是"写代码的人",而是:

### 1. 架构设计师

在`openspec/specs/Architecture.md`定义技术选型:

* 用什么数据库
* API返回什么格式
* 前端用什么状态管理

这些Agent自己决定不了,需要人类做决策。

### 2. 规格审查员

每天Review三个文件:

* `Product_Brief.md`:需求有没有跑偏
* `INTEGRATION.md`:Agent之间有没有理解偏差
* `Definition_of_Done.md`:质量标准够不够严格

**代码我基本不看。Agent写的代码质量比我高。**

### 3. 阻塞点协调员

当Frontend被Backend阻塞,我要:

1. 看Backend的git log,了解进度
2. 在Backend的terminal问:"还要多久?"
3. 告诉Frontend:"等10分钟"

就像项目经理协调两个团队的依赖关系。

### 4. 质量把关人

Integration Manager合并后,我要:

1. 跑一遍Demo,看交互是否流畅
2. 检查边界条件(比如扫描空目录会不会报错)
3. 看E2E测试覆盖了哪些场景,有没有遗漏

**不是检查代码语法,而是检查业务逻辑。**

---

## 三种SDD工具的选择建议

回到最开始的问题:OpenSpec、SpecKit、Superpowers到底选哪个?

### 选择矩阵

```
你的情况                          推荐工具           理由
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
个人项目,快速试错                OpenSpec          轻量,安装快
团队3人以下,需求经常变            OpenSpec          灵活,无严格流程
团队5人以上,需要规范              SpecKit           有constitution强制约束
企业项目,需要合规性                SpecKit           阶段清晰,可审计
追求代码质量,重视TDD              Superpowers       内置TDD工作流
需要复用工作流(如多个项目)        Superpowers       Skills全局共享
```

### 组合使用

实际上这三者可以混用:

```
主框架: OpenSpec (管理变更提案)
+
约束层: SpecKit的constitution.md (定义不可违反的规则)
+  
增强层: Superpowers的skills (给Agent装能力)
```

例如在我的项目:

```
openspec/
  specs/
    Product_Brief.md        # OpenSpec
    Architecture.md         # OpenSpec
    constitution.md         # SpecKit借鉴,定义技术栈约束
    User_Stories.md         # OpenSpec
  
.claude/
  skills/
    superpowers/            # 软链接到全局
      tdd/                  # Superpowers的TDD技能
      brainstorming/        # Superpowers的脑暴技能
```

**工具是为人服务的,不是教条。**

---

## 你可以做什么

如果你想复制我的工作流,这是最快的路径:

### Week 1:熟悉基础

**Day 1-2**:安装OpenSpec,写第一个Product\_Brief.md

```
npm install -g @fission-ai/openspec
mkdir my-first-project && cd my-first-project
openspec init
```

在`openspec/specs/Product_Brief.md`写一个超简单的需求,比如"做一个TODO列表"。

**Day 3-4**:让一个Agent基于spec开发

```
claude "读取openspec/specs/,实现一个基本的TODO列表"
```

观察它生成的代码,调整你的spec描述。

**Day 5-7**:练习Git Worktree

```
git worktree add ../my-project-feature-a -b feature-a
cd ../my-project-feature-a
claude "实现添加TODO功能"
```

### Week 2:多Agent协作

**Day 8-10**:用Worktree运行两个Agent

* 一个做Backend
* 一个做Frontend
* 手动协调它们的依赖(在INTEGRATION.md记录接口)

**Day 11-14**:配置CLAUDE.md和commit规范

给每个Agent写专属的CLAUDE.md,定义它的职责和代码规范。

配置`.gitmessage`模板,要求Agent写有意义的commit。

### Week 3:自动化质量

**Day 15-17**:写第一个Git Hook

在`.claude/hooks/pre-commit`检查commit message格式。

**Day 18-21**:创建Integration Manager Agent

让它负责合并分支和运行测试。

### Week 4:应用到实际项目

选一个真实需求,完整走一遍流程:

1. 写specs (Product Brief + Architecture + User Stories)
2. 创建worktrees
3. 启动多个Agent
4. 协调依赖
5. 自动化集成

**用2周时间,你就能掌握这套工作流。**

然后你会发现:一个人 + 三个Agent = 一个7人团队的产出。

---

## 最后想说的话

三个月前,我还在担心AI会不会取代工程师。

现在我明白了:**真正的问题不是"AI会不会取代人",而是"不会用AI的人会不会被会用的人取代"。**

我现在花在"写代码"上的时间不到10%。

90%的时间在:

* 设计架构(什么技术选型,什么数据结构)
* 写清晰的规格(让Agent理解我要什么)
* 协调依赖(Backend和Frontend的接口怎么对齐)
* 把关质量(业务逻辑对不对,边界条件考虑了吗)

**这才是2026年一个优秀工程师的核心竞争力。**

Scrum没有过时。 Agile没有过时。

过时的是那种"坐在椅子上敲8小时代码"的工作方式。

现在是:

* 2小时写specs
* 启动3个Agent,去喝咖啡
* 回来看git log和测试报告
* 调整specs,再启动下一轮迭代

**效率提升的不是10倍,是100倍。**

周一早上,PM看完Demo,问我:"这是你一个人做的?"

我说:"不,是我和三个同事一起做的。只不过,它们不需要工资,不会请假,24小时随时待命。"

PM沉默了一会儿,问:"我们公司什么时候能推广这种工作方式?"

我笑着说:"现在。只要你愿意,每个团队都可以这么做。"

---

好了,现在打开你的终端,输入:

```
npm install -g @fission-ai/openspec
mkdir your-project && cd your-project
openspec init
```

创建你的第一个`Product_Brief.md`。

写下你想做的东西。

然后启动你的第一个Agent。

新时代的Scrum,从这一刻开始。

---

## 附录:三种工具的目录结构完整对比

### OpenSpec

```
project/
├── .openspec/           
│   ├── config.json      # 工具配置
│   └── templates/       # 文档模板
├── openspec/
│   ├── specs/           # 长期存在的规格
│   │   ├── Product_Brief.md
│   │   ├── Architecture.md  
│   │   └── User_Stories.md
│   ├── changes/         # 短期的变更提案
│   │   ├── 001-add-search/
│   │   │   ├── proposal.md   # 为什么要做
│   │   │   ├── tasks.md      # 拆解的任务
│   │   │   └── design.md     # 技术设计
│   │   └── archive/          # 已完成
│   │       └── 001-add-search/
│   └── AGENTS.md        # Agent行为规范
└── src/                 # 源代码
```

**哲学**:像看板,任务流动(TODO → DOING → DONE)

**Agent交互点**:主要读`openspec/specs/`,有新功能时读对应的`changes/xxx/`

**历史管理**:物理移动文件夹到`archive/`

---

### SpecKit

```
project/
├── .speckit/
│   ├── config.yaml      
│   └── memory/
│       └── constitution.md  # 项目"宪法"
├── specs/               
│   ├── 001-auth-feature/
│   │   ├── spec.md          # 功能规格
│   │   ├── requirements.md  # 详细需求
│   │   ├── plan.md          # 实施计划
│   │   └── tasks.md         # 任务清单
│   └── 002-api-redesign/
│       └── ...
├── scripts/             # 验证脚本
│   └── validate-spec.py
└── src/
```

**哲学**:像蓝图,分阶段严格推进(specify → plan → implement)

**Agent交互点**:先读`constitution.md`(不可违反的约束),再读具体的`specs/xxx/`

**历史管理**:通过文件内部的YAML状态字段(`status: proposed/approved/implemented`)

---

### Superpowers

```
project/
├── .claudecode/         # Claude Code配置
│   └── CLAUDE.md
├── .worktrees/          # Agent的工作区(不提交到Git)
│   ├── feature-a/
│   └── feature-b/
├── docs/
│   ├── design/          # brainstorming技能生成的
│   │   └── feature-x.md     
│   └── plans/           # writing-plans技能生成的
│       └── feature-x.md     
├── skills/              # 项目专属技能
│   └── our-coding-standards/
│       └── SKILL.md
└── src/

# 全局技能库(通过软链接)
~/.claude/skills/
├── superpowers/            
│   ├── brainstorming/      # 脑暴阶段的技能
│   │   └── SKILL.md
│   ├── writing-plans/      # 计划阶段的技能
│   │   └── SKILL.md
│   ├── tdd/                # TDD阶段的技能
│   │   └── SKILL.md
│   └── preserving-productive-tensions/
│       └── SKILL.md
└── my-custom-skills/       
```

**哲学**:像大脑插件,给Agent装"能力包"

**Agent交互点**:自动加载全局的`~/.claude/skills/`,再加载项目的`skills/`

**历史管理**:记录在`docs/thoughts/`(Agent的思考过程)

---

### 如何选择?看你的痛点

**痛点:需求经常变,改来改去**→ 用OpenSpec,它的`changes/`机制支持频繁迭代

**痛点:团队成员水平参差不齐,需要强制规范**→ 用SpecKit,`constitution.md`是所有人必须遵守的

**痛点:重复性工作太多,每个项目都要写相似的代码**→ 用Superpowers,把通用流程封装成Skills复用

**痛点:想要三者的优点**→ 组合使用,用OpenSpec做任务管理,SpecKit定义约束,Superpowers增强能力

## 关于作者

申导，一个在AI时代重新定义"工程师"角色的实践者和敏捷教练。

曾经每天写代码12小时,现在每天写规格2小时,效率提升50倍。

如果这篇文章对你有帮助,点个在看,让更多人看到AI时代真正的工作方式。