---
title: 还没配 Codex 的 AGENTS.md？先抄这 65 行
author: 硅潮
date: 硅潮硅潮
url: https://mp.weixin.qq.com/s?__biz=MzI0MDEyNzY0NQ==&mid=2648540951&idx=1&sn=02bc0f782b9899e43b8769d78b241854&chksm=f0a1f76fc944af0312e61adf6b34d9d80b9c78d5b97b987edf8b28631154542f27a15b853b59&mpshare=1&scene=24&srcid=05266OdMinzxqyAAHL4umMy0&sharer_shareinfo=520ccf82a240a237b4f71d4eb7b1cc31&sharer_shareinfo_first=520ccf82a240a237b4f71d4eb7b1cc31#rd
---

如果你已经开始用 Codex 写代码，但还没给它配过全局规则，大概率会遇到这种场面：

你只是让它修一个小 bug。

它先猜了一堆你没说过的需求，又顺手重构了旁边三个文件，最后补了一层“为了未来扩展”的抽象。

等你打开 diff，问题已经变成：**我到底是在 review 这次需求，还是在审一场 AI 自主装修？**

这时候，不要急着写一篇 3000 字的“Codex 使用圣经”。先抄一份极简作业就够。

仓库在这里：

https://github.com/multica-ai/andrej-karpathy-skills

里面最关键的是一个 65 行的 `CLAUDE.md`。虽然名字叫 `CLAUDE.md`，但它的内容更像一套给编码 Agent 的通用行为规范。放到 Codex App 的全局自定义指令里，或者放到本地 `~/.codex/AGENTS.md`，都很适合当起点。

先说清楚：这个仓库不是 Karpathy 本人仓库，而是 Multica 作者把 Karpathy 对 LLM 编码问题的观察整理成了可复用规则。标题里说“抄 Karpathy 作业”，抄的是这套工程判断。

## 这 65 行真正值钱的地方

我把它拆开看了一遍，核心其实就四件事：

第一，编码前先别急着动手。

LLM 最常见的问题不是不会写代码，而是太会自圆其说。需求有歧义时，它会默默选一种解释，然后一路执行下去。

这份规则第一条就卡住它：

* 不要暗自假设；
* 有多种解释就说出来；
* 有更简单做法就提醒；
* 真不清楚时停下来问。

这条对 Codex 很关键。因为 Codex 一旦进了执行循环，成本就从“多问一句”变成“多改 20 个文件”。

第二，能 50 行解决，就不要写 200 行。

很多 Agent 有一种本能：把普通需求产品化，把一次性脚本框架化，把局部修复平台化。

这份规则直接压住这个倾向：

* 不加需求之外的功能；
* 不为一次性代码做抽象；
* 不为了“未来可能用到”加配置；
* 200 行能压到 50 行，就重写。

对个人项目和科研代码尤其有用。很多代码库不是被 bug 拖垮的，而是被“顺手加一点可扩展性”拖垮的。

第三，只碰必须碰的地方。

这是我最建议放进 Codex 全局规则的一条。

很多时候你让 Agent 修 A，它会顺手美化 B、改名 C、重排 D。每一处单看都“有道理”，合在一起就变成不可 review 的 diff。

这 65 行里有一个非常硬的验收标准：

**每一行修改都应该能直接追溯到用户的请求。**

这个标准比“写得优雅”重要。Agent 写代码时，最先要学会克制。

第四，别给任务，给成功标准。

不要只说：

```
修一下这个 bug。
```

要让 Codex 把任务变成可验证目标：

```
先写一个能复现 bug 的测试，确认失败； 再修改实现； 最后跑测试确认通过。
```

这也是 Karpathy 一直强调的工程味：Agent 擅长循环执行，但你得给它一个能判断“到站了”的标准。

## 直接怎么配

你可以按自己的入口选一种。

### 方式一：Codex App 里直接粘贴

如果你用的是 Codex App，打开全局自定义指令区域，把仓库里的 `CLAUDE.md` 内容复制进去。

最小动作就是：

1. 打开仓库：`multica-ai/andrej-karpathy-skills`；
2. 找到 `CLAUDE.md`；
3. 复制全文；
4. 粘贴到 Codex App 的全局自定义指令；
5. 保存后新开一个任务，让 Codex 总结当前指令，确认它读到了。

如果你已经有自己的全局规则，不要直接覆盖。先把这 65 行追加进去，再删掉和你原规则冲突的部分。

### 方式二：Codex CLI 全局 AGENTS.md

如果你用 Codex CLI，更直接。

```
mkdir -p ~/.codex curl -L https://raw.githubusercontent.com/multica-ai/andrej-karpathy-skills/main/CLAUDE.md \   -o ~/.codex/AGENTS.md
```

然后验证：

```
codex --ask-for-approval never "Summarize the current instructions."
```

如果 Codex 能复述出“Think Before Coding / Simplicity First / Surgical Changes / Goal-Driven Execution”这几条，就说明全局规则已经加载。

注意：如果你本来就有 `~/.codex/AGENTS.md`，先备份。

```
cp ~/.codex/AGENTS.md ~/.codex/AGENTS.md.bak.$(date +%Y%m%d-%H%M%S)
```

再手动合并，不要让一条命令把你原来的工作习惯清空。

### 方式三：放进项目根目录

如果你只想让某个项目使用这套规则，可以把内容放到项目根目录的 `AGENTS.md`。

官方机制里，Codex 会从项目根目录一路读到当前工作目录。越靠近当前目录的规则越晚出现，也就更适合覆盖前面的通用规则。

一个简单分层可以这样：

```
~/.codex/AGENTS.md          # 全局习惯：少改、先测、问清楚 my-project/AGENTS.md        # 项目习惯：怎么安装、怎么测试、怎么提交 my-project/api/AGENTS.md    # 子目录习惯：API 层的特殊约定
```

临时强覆盖可以用 `AGENTS.override.md`。但不要滥用。override 用多了，规则系统很快会变成另一种混乱。

## 我会在这 65 行后面补什么

原版已经够稳，但如果你是 Codex 重度用户，我建议再加一个“本地化补丁区”。

不要写抽象愿望，比如：

```
请保持代码高质量。
```

这句话没用。Codex 不知道什么叫高质量。

更好的写法是：

```
当修改 TypeScript API handler 时： 1. 先定位现有 handler 的输入校验模式； 2. 只复用项目已有错误返回结构； 3. 修改后运行 npm test -- --runInBand； 4. 如果测试不能运行，报告命令、失败原因和未验证风险。
```

一条好规则最好有三个部件：

```
触发场景 → 行为约束 → 验证方式
```

比如：

```
当任务涉及数据库 migration 时： 不要自动执行 destructive migration； 先生成 migration 文件和回滚说明； 验证命令是 npm run db:migrate:check。
```

再比如：

```
当任务只要求修复一个 UI 文案时： 不要改组件结构、状态管理或样式系统； 只修改对应 copy； 最后用 git diff 说明每个文件为什么被改。
```

这才是能沉淀的 Codex 规则。

## 规则不是越多越好

很多人第一次配 Agent 指令，会把所有焦虑都写进去：安全、测试、架构、命名、提交、文档、性能、审美、沟通风格……最后写成一堵墙。

问题是，规则太长以后，Agent 反而抓不住重点。

OpenAI 的 Codex 文档里也提到，项目指令有默认大小限制，默认合并上限是 32 KiB。这个限制本身就是提醒：**规则要分层，不要把所有东西塞进全局。**

我的建议是：

* 全局规则只放“所有项目都成立”的工作协议；
* 项目根目录放安装、测试、提交、架构约定；
* 子目录放局部例外；
* 每次 Codex 犯错，只补一条能复现、能验证的新规则。

这 65 行适合做全局规则，因为它管的是 Agent 的底层坏习惯：假设、复杂化、乱改、缺验证。

项目里那些更具体的东西，比如 `pnpm` 还是 `npm`、用 `pytest` 还是 `vitest`、PR 前跑哪条命令，应该留给项目级 `AGENTS.md`。

## 复制前，先过一遍这个检查表

你可以照这个顺序来：

1. **先备份**

   ：如果已有全局指令，先复制一份。
2. **再粘贴**

   ：把 65 行放到全局自定义指令或 `~/.codex/AGENTS.md`。
3. **做一次验证**

   ：让 Codex 总结当前指令，确认它读到了。
4. **跑一个小任务**

   ：找一个低风险 bug，让它按“先计划、后修改、再验证”执行。
5. **看 diff**

   ：有没有无关文件被碰？有没有过度抽象？有没有验证结果？
6. **只补一条规则**

   ：如果这次翻车，把翻车点写成“触发场景 → 行为约束 → 验证方式”。

不要一上来追求完美配置。

先让 Codex 从“会写代码”变成“知道什么时候该停”。

## 这一份规则适合谁

适合：

* 刚开始认真用 Codex App / CLI 的人；
* 经常被 Agent 的大 diff 折磨的人；
* 希望把个人编码习惯沉淀成全局规则的人；
* 做科研小工具、内部脚本、个人项目的人。

不太适合：

* 已经有成熟团队级规范的人；
* 想要一套覆盖安全、发布、CI、review、架构的完整工程手册的人；
* 把 Agent 当完全自动程序员、不想做人工验收的人。

对大多数个人用户，它最好的位置就是“第一层刹车”。

先把 Agent 拉回一个朴素状态：问清楚，少写点，只改该改的，跑完验证再说完成。

这四件事做到了，你的 Codex 使用体验会立刻稳一截。

## 往期推荐 / 延伸阅读

* [别急着接 API：先把 AI Studio 当成科研模型风洞](https://mp.weixin.qq.com/s?__biz=MzI0MDEyNzY0NQ==&mid=2648539824&idx=1&sn=5db650025a52427f57fac47ceef1f607&scene=21#wechat_redirect)
* [AI 编程 Agent 第一份全栈榜单：Cursor 第一，Gemini 垫底，真正的战争变了](https://mp.weixin.qq.com/s?__biz=MzI0MDEyNzY0NQ==&mid=2648540847&idx=1&sn=49026cb68d9fef497fdae157b64ed8f3&scene=21#wechat_redirect)
* [ChatGPT Images 2.0 做科研配图，关键不在好看，在可控](https://mp.weixin.qq.com/s?__biz=MzI0MDEyNzY0NQ==&mid=2648540048&idx=1&sn=f76323e70997d204f67a59638d67369c&scene=21#wechat_redirect)

参考来源：

1. `multica-ai/andrej-karpathy-skills`

   GitHub 仓库：`https://github.com/multica-ai/andrej-karpathy-skills`
2. OpenAI Codex 文档：Custom instructions with AGENTS.md：`https://developers.openai.com/codex/guides/agents-md`