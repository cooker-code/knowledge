---
title: 开发者新神器！OpenSpec：让项目规划、协作与落地一次到位的“变更驱动开发框架”
author: 无限航行指南
date: 
url: https://mp.weixin.qq.com/s?__biz=MzYyMjc3MTE2Mg==&mid=2247483866&idx=1&sn=c7d7919bf0bdf33c7397a3c1f4306da7&chksm=fea8a69b64c9486a80f70013a6bb269511223b77fbc38aa9fe7896437026b1fb18476d2cc994&mpshare=1&scene=24&srcid=1028gMOfV3Mkh69Up1yTFNlX&sharer_shareinfo=22dde8ee510aad1f170ae04c102fff3b&sharer_shareinfo_first=22dde8ee510aad1f170ae04c102fff3b#rd
---

# 🚀 开发者新神器！OpenSpec：让项目规划、协作与落地一次到位的“变更驱动开发框架”

* 官网地址: https://openspec.dev/
* github地址：https://github.com/Fission-AI/OpenSpec/

## 安装

```
npm install -g @fission-ai/openspec@latest
```

这里我选择的ide是cursor

```
 openspec --version  
0.13.0  
(base) PS G:\code\ai\copy-chrome-extension> openspec init  
  
 ████   █████   ██████  ██  ██   █████  █████   ██████   █████  
██  ██  ██  ██  ██      ███ ██  ██      ██  ██  ██      ██  
██  ██  █████   █████   ██ ███   ████   █████   █████   ██  
██  ██  ██      ██      ██  ██      ██  ██      ██      ██  
 ████   ██      ██████  ██  ██  █████   ██      ██████   █████  
  
Welcome to OpenSpec!  
  
Step 3/3  
  
Review selections  
Press Enter to confirm or Backspace to adjust.  
  
▌ Cursor  
▌ OpenSpec structure created  
▌ AI tools configured  
  
✔ OpenSpec initialized successfully!  
  
Tool summary:  
▌ Root AGENTS.md stub created for other assistants  
▌ Created: Cursor  
▌ Skipped: Auggie, Claude Code, Cline, CodeBuddy Code, Crush, Factory Droid, OpenCode, Kilo Code, Windsurf, Codex, GitHub Copilot, and Amazon Q Developer  
  
Use `openspec update` to refresh shared OpenSpec instructions in the future.  
  
Next steps - Copy these prompts to Cursor:  
────────────────────────────────────────────────────────────  
1. Populate your project context:  
   "Please read openspec/project.md and help me fill it out  
    with details about my project, tech stack, and conventions"  
  
2. Create your first change proposal:  
   "I want to add [YOUR FEATURE HERE]. Please create an  
    OpenSpec change proposal for this feature"  
  
3. Learn the OpenSpec workflow:  
   "Please explain the OpenSpec workflow from openspec/AGENTS.md  
    and how I should work with you on this project"
```

这里我直接按照它的意思，将最下面3个步骤直接复制粘贴到`cursor`对话框

```
1. Populate your project context:  
   "Please read openspec/project.md and help me fill it out  
    with details about my project, tech stack, and conventions"  
  
2. Create your first change proposal:  
   "I want to add [YOUR FEATURE HERE]. Please create an  
    OpenSpec change proposal for this feature"  
  
3. Learn the OpenSpec workflow:  
   "Please explain the OpenSpec workflow from openspec/AGENTS.md  
    and how I should work with you on this project"
```

翻译过来的意思就是

```
我将帮助你开始使用 OpenSpec！让我先阅读相关文件，以了解当前的项目状态以及需要填写的内容。  
[调用了3个工具]  
现在，让我查看根目录，了解这个 Chrome 扩展的功能：  
[调用了5个工具]  
非常好！我看到这是一个刚创建的 Chrome 扩展项目，OpenSpec 框架已经初始化完成。目前项目中仅包含 OpenSpec 的基础设置。接下来，我将帮助你完成以下三个任务：  
📋 任务 1：填写 project.md  
我注意到你的 project.md 目前是一个模板。既然这是一个全新的 Chrome 扩展项目，我需要了解一些背景信息，以便为你填充内容。  
关键问题如下：  
你的 Chrome 扩展是做什么的？  
 （例如：“复制带格式的文本”、“截图工具”、“效率助手”等）  
你计划使用什么技术栈？  
 （例如：Vanilla JS、TypeScript、React、Vue 等）  
是否会使用特定的 Chrome API？  
 （例如：clipboard、tabs、storage、contextMenus 等）  
目标 Manifest 版本？  
 （Manifest V2 还是 V3？）  
🔄 任务 3：OpenSpec 工作流说明  
在正式开始之前，我先向你解释 OpenSpec 的工作流，这样我们在后续协作中就会更加顺畅。  
OpenSpec 三阶段工作流  
阶段 1：创建变更（规划阶段）  
当你希望添加新功能或进行较大改动时：  
查看现有工作内容：运行 openspec list 和 openspec list --specs  
创建提案：选择唯一的变更 ID（如 add-copy-button）  
撰写规范：用场景描述需求  
验证规范：运行 openspec validate <change-id> --strict  
审批通过后 再进行实现  
阶段 2：实现变更（开发阶段）  
阅读 proposal.md，了解目标  
阅读 tasks.md，查看任务清单  
按顺序完成任务  
仅在任务完成后 标记为完成  
未经批准请勿开始开发！  
阶段 3：归档变更（完成阶段）  
部署完成后：  
将变更移动至 changes/archive/YYYY-MM-DD-[name]/  
更新主规范文件（位于 specs/ 目录）  
再次验证归档状态  
关键概念  
specs/ = 当前系统规范（系统“现在”是什么样）  
changes/ = 提案目录（系统“将要”如何变化）  
archive/ = 历史记录（系统“过去”做过什么改动）  
何时需要创建提案  
✅ 应创建提案的情况：  
新功能或新特性  
破坏性变更（API、数据结构等）  
架构性调整  
改变系统行为的性能优化  
安全模式更新  
❌ 无需创建提案的情况：  
修复 bug（恢复原本行为）  
拼写或格式修正  
非破坏性依赖更新  
配置文件修改  
针对现有功能的测试  
规范文档的基本格式  
每条需求应包含以下内容：  
### Requirement: 功能名称  
系统应当实现某种功能……  
  
#### Scenario: 成功场景  
- **WHEN（当）** 用户执行某操作    
- **THEN（则）** 系统应有的响应结果  
重点提示：  
 请务必使用 ####（四个井号）来标注“场景”，不要用普通项目符号！  
🎯 下一步  
选项 A：手动填写 project.md  
 请回答我上面的几个问题，我将为你生成一份完整、定制化的 project.md 项目说明。  
选项 B：告诉我你想构建的功能  
 如果你已经有目标功能，我将：  
根据你的描述智能推测技术栈；  
自动生成合适的 project.md；  
帮你创建首个变更提案（change proposal）。
```

上面是我要做的一个插件的给我提供的3个问题，你得根据你自己的项目进行回答。

---

你根据它给你的提问输入完你的需求之后，它会给你一些建议，或者你有什么要修改的可以告诉它，然后它会执行它的一个命令来生成提案、实施清单、需求和场景、验证提案等文档

```
需要我现在创建这个提案吗？如果同意，我会：  
1. 运行 openspec list --specs 检查现有规范  
2. 创建 openspec/changes/add-copy-bypass-core/ 目录结构  
3. 编写 proposal.md（为什么、改什么、影响）  
4. 编写 tasks.md（实施清单）  
5. 创建规范增量 specs/copy-bypass/spec.md（需求和场景）  
6. 运行 openspec validate 验证提案  
或者，如果你想先修改 project.md 的内容，或者想创建不同的功能，请告诉我！
```

---

我没啥想改的，直接按照它说的，创建提案

```
好的！我现在为你创建第一个OpenSpec变更提案。让我按照标准流程来操作：  
  
[4 tools called]  
  
完美！项目是全新的，没有现有的规范和变更。现在让我创建第一个变更提案：  
  
[1 tool called]  
  
现在创建提案文件：  
  
[3 tools called]  
  
太好了！提案文件已创建。现在让我验证提案是否符合OpenSpec规范：  
  
[1 tool called]  
  
完美！✅ **验证通过**！让我查看提案的详细信息：  
  
[2 tools called]  
  
## 🎉 提案创建成功！  
  
### ✅ 已完成的任务  
  
我已经成功创建了你的第一个OpenSpec变更提案 `add-copy-bypass-core`：  
  
**1. 提案结构** ✓  
```  
openspec/changes/add-copy-bypass-core/  
├── proposal.md          # 提案说明（为什么、改什么、影响）  
├── tasks.md             # 30项实施任务清单  
└── specs/  
    └── copy-bypass/  
        └── spec.md      # 6个核心需求和15个场景  
```  
  
**2. 核心需求规范** ✓  
- ✅ CSS User-Select Bypass（移除CSS限制）  
- ✅ JavaScript Event Listener Removal（移除事件监听器）  
- ✅ Event Handler Reset（重置事件处理器）  
- ✅ Selection API Protection（保护选择API）  
- ✅ Content Script Injection（脚本注入）  
- ✅ Non-Invasive Operation（非侵入运行）  
  
**3. 验证通过** ✓  
```  
✓ 提案格式正确  
✓ 所有需求都有场景  
✓ 符合OpenSpec规范  
```  
  
---  
  
## 📋 OpenSpec三阶段工作流程回顾  
  
### 🎯 **阶段1：创建变更（已完成）** ✅  
  
我们刚完成的步骤：  
- ✅ 检查现有规范和变更  
- ✅ 创建提案目录和文件  
- ✅ 编写需求和场景  
- ✅ 验证提案格式  
  
### 🛠️ **阶段2：实施变更（待进行）**  
  
下一步你可以：  
1. **查看任务列表**：`openspec show add-copy-bypass-core` 查看30项任务  
2. **开始实施**：按照 `tasks.md` 逐项完成  
3. **标记完成**：完成后将 `- [ ]` 改为 `- [x]`  
  
### 📦 **阶段3：归档变更（部署后）**  
  
实施并测试完成后：  
```bash  
openspec archive add-copy-bypass-core --yes  
```  
  
---  
  
## 🚀 下一步建议  
  
你现在有两个选择：  
  
### **选项A：立即开始实施** ⭐ 推荐  
我可以帮你：  
1. 创建项目基础结构（package.json, vite.config.js等）  
2. 实现核心的content script功能  
3. 配置Manifest V3文件  
4. 设置Vue 3开发环境  
  
**只需说**："开始实施" 或 "开始开发"  
  
### **选项B：审查提案**  
你可以：  
1. 查看 `openspec/changes/add-copy-bypass-core/proposal.md` 审查提案  
2. 查看 `openspec/changes/add-copy-bypass-core/tasks.md` 查看完整任务列表  
3. 查看 `openspec/changes/add-copy-bypass-core/specs/copy-bypass/spec.md` 查看详细需求  
4. 如需修改，告诉我你想调整什么  
  
---  
  
## 📚 提案摘要  
  
**目标**：创建Chrome扩展绕过网站复制限制  
  
**技术栈**：  
- Vue 3 + JavaScript  
- Vite构建工具  
- Manifest V3  
- Chrome APIs: `activeTab`, `scripting`  
  
**核心功能**：  
- 移除CSS `user-select: none`  
- 移除JavaScript事件监听器  
- 保护Selection API  
- 监听动态DOM变化  
  
**任务数量**：30项（6个大类）  
  
---  
  
**你想怎么继续？** 🤔
```

这里我针对它的`tasks`里测试要求添加`chrome devtools mcp`模拟测试

```
好主意！使用 chrome-devtools MCP 可以实现自动化测试，让测试更加可靠和可重复。让我更新 tasks.md：  
  
[2 tools called]  
  
完美！我已经更新了 `tasks.md`，加入了使用 chrome-devtools MCP 的自动化测试部分。让我验证更新后的提案：  
  
[2 tools called]  
  
## ✅ 更新完成！  
  
我已经成功将 chrome-devtools MCP 自动化测试集成到测试和验证部分。  
  
### 📊 更新摘要  
  
**任务数量**：30 → **40 个任务**（新增10个测试任务）  
  
### 🔄 主要变更  
  
#### 1️⃣ **项目初始化部分**  
- ✅ 新增 1.3：安装测试依赖（配置 chrome-devtools MCP）  
- ✅ 更新 1.5：添加 `tests/` 目录  
  
#### 2️⃣ **测试和验证部分**（重构为4个子部分）  
  
**5.1 手动测试设置**  
- 构建和加载扩展的基础步骤  
  
**5.2 自动化测试 - chrome-devtools MCP** ⭐ 新增  
- 创建测试脚本和测试网页模板  
- **CSS user-select 测试**：使用 `navigate_page`, `take_snapshot`, `evaluate_script`  
- **JavaScript 事件测试**：验证 copy, contextmenu, selectstart 事件  
- **事件处理器测试**：验证 oncopy, onselectstart, oncontextmenu 重置  
- **Selection API 测试**：验证保护机制防止选择被清除  
- **MutationObserver 测试**：验证动态元素处理  
  
**5.3 兼容性测试**  
- 测试正常网站功能  
- 使用 `list_console_messages` 检查错误  
- 性能验证  
  
**5.4 真实网站验证**  
- 手动测试真实反复制网站  
  
### 🛠️ Chrome-DevTools MCP 工具使用  
  
你将使用以下 MCP 工具进行自动化测试：  
  
| 工具 | 用途 |  
|------|------|  
| `navigate_page` | 访问测试页面 |  
| `take_snapshot` | 获取页面可访问性快照 |  
| `evaluate_script` | 执行JavaScript验证样式和API状态 |  
| `click` | 模拟用户交互 |  
| `wait_for` | 等待DOM变化 |  
| `list_console_messages` | 检查错误日志 |  
  
### 📋 测试覆盖范围  
  
✅ **CSS限制绕过测试**  
✅ **JavaScript事件移除测试**    
✅ **事件处理器重置测试**  
✅ **Selection API 保护测试**  
✅ **动态DOM处理测试**  
✅ **兼容性和性能测试**  
✅ **真实场景验证**  
  
### 🎯 验证状态  
  
```  
✓ 提案验证通过  
✓ 40个任务清单  
✓ 自动化测试完整覆盖  
```  
  
---  
  
## 下一步  
  
现在提案已经更新并验证通过！你可以：  
  
**选项 A：开始实施** 🚀  
说 "开始实施" 或 "开始开发"，我会帮你：  
1. 创建项目结构  
2. 配置 package.json 和 vite.config.js  
3. 实现核心功能  
4. 创建自动化测试  
  
**选项 B：查看完整任务清单**  
使用命令查看：  
```bash  
openspec show add-copy-bypass-core  
```  
  
**选项 C：继续修改提案**  
如果还想调整其他部分，告诉我！  
  
准备好开始了吗？💪
```

没啥疑问，直接开始实施，它这个挺好，直接给出下一步预设，让你选择，它自动去执行`command`添加新提案，一个功能完成之后会自动归档，如果你说出任务都完成的话，会将`- [ ]`都变成`- [x]`已完成状态。

它这有一个好处，你历史完成的功能都有归档，后续还能进行回看，以前我们AI编程的时候，你输入的一些`instructions`基本都随着`cursor`聊天记录沉浮了，你很难在翻看你以前做了什么，现在有这种归档功能，几乎都能防止AI扯皮的事情了。

如果你下次进来是一个新会话的话，还想继续使用白话文使用`cursor`来操作`openspec`的话，就必须先让它了解一下`openspec`的开发规范和规则，通过相关描述让`cursor`自己去调用`openspec/`的几个命令去创建提案、功能描述、任务拆分等`markdown`文件。

比如：

```
使用 openspec 添加功能，我需要一个chrome extension 的icon，帮我从开源图标网站里找一个适合的进行使用
```

它会自己去了解下当前项目目录里的`openspec`的指导文档，再自己调用`openspec`的几个`command`来实现对应的文档生成。

## 使用总结

如果按照刚安装完默认给出的提示去`cursor`里粘贴使用，会给你提供下一步的建议，这样都可以直接使用白话文进行交流，而没必要使用到`/openspec-xxx`的命令了。

### 添加提案

但是如果出现了新会话，没有上下文了，你要添加新功能

```
/openspec:proposal <任务，新需求/变更内容>  
  
  
# 官方案例(翻译后)  
  
创建一个OpenSpec变更提案，按角色和团队添加配置文件搜索过滤器  
  
相当于  
  
/openspec:proposal Add profile search filters  
  
这里的 `Add profile search filters` 就是你所要描述的任务或者需求，如果是在老项目里，可能你需要通过`@`的方式精准到哪些文件去规划需求，让AI知道你具体要改哪些内容  
  
## 注意  
  
在它添加完提案后，会有一个`openspec/changes/{生成的功能简洁的英文ID}  
这个英文ID在后续里使用命令的 后缀参数较多
```

### 检验和审查

```
$ openspec list                             # Confirm the change folder exists  
$ openspec validate add-profile-filters     # Validate spec formatting  
$ openspec show add-profile-filters         # Review proposal, tasks, and spec delta
```

注意：这3个是终端命令，不是在`cursor`或者`cc`里使用的，基本功能就是看你现在有哪些变更需求

### 完善需求

这时虽然有`proposal, tasks, and spec delta`等文件的生成，还没有开始实施/执行，所以这期间你要在检验和审查之后提出你觉得需要修改的地方，这里会针对你的要求去调整提案、任务以及需求描述。

### 实施/执行

正常在你确认完需求之后，`cursor`会有一个预设的提问，是否要开始"实施计划/执行计划?"，基本你只需输入“实施”即可开始自动触发以下命令

```
/openspec:apply add-profile-filters
```

最后等它将`tasks.md`里的所有待完成的任务都执行完成之后，基本就大功告成了，后面就是缝缝补补，你问我答环节，当然，如果你在需求整改期间，提出了一些测试上的要求，比如你要求使用`chrome devtools mcp`模拟测试，这一部分它不会帮你完成这个任务，你需要再次对话，执行，且你确认测试完成之后，这些在`tasks.md`里的测试任务才会变成已完成，最后才能进行归档。

### 归档

当你所有任务都完成了，且你自己也测试了，觉得没啥问题了，就可以进行功能归档

```
/openspec:archive add-profile-filters  
  
# 或者  
/openspec:archive add-profile-filters --yes  
  
# 再或者  
/openspec:archive add-profile-filters -y
```

每一次的需求变更的功能拆解、提案、任务对应的文档都进行归档，这个真的是方便后续需求变更或者产品失忆了之后可以与产品经理扯犊子的好东西。