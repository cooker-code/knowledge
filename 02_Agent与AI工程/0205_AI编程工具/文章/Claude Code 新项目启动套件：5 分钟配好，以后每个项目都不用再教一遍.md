---
title: Claude Code 新项目启动套件：5 分钟配好，以后每个项目都不用再教一遍
author: Import AI
date: Import AIImport AI
url: https://mp.weixin.qq.com/s?__biz=MzYzMTc4MjgyNA==&mid=2247484454&idx=1&sn=98015237a637784f7acce653aa5f0aa2&chksm=f150b4fc27240597ebc06d324bfd7ecdecf4866a166331d4ca0cd6947c4539f10ce914012edd&mpshare=1&scene=24&srcid=0509vDrng6Y2S20MVKWukd3d&sharer_shareinfo=14ee1855d568b88897001125567ab181&sharer_shareinfo_first=14ee1855d568b88897001125567ab181#rd
---

周一那篇 Claude code15 个功能的文章发出去之后，私信里最多的一类问题是：配哪些、按什么顺序配、有没有现成的模板可以直接用。

[别把Claude当豆包了，Claude 有 15 个功能你大概从没点开过](https://mp.weixin.qq.com/s?__biz=MzYzMTc4MjgyNA==&mid=2247484441&idx=1&sn=b08f9ab3b777cafcfa554cd1fe8d964d&scene=21#wechat_redirect)

说实话，我自己的配置是半年里踩坑踩出来的。每犯一次错加一条规则，每次被弹窗烦到加一条 allow。要是有人一开始就给我一套模板，能少走很多弯路——所以现在我把这套整理出来给你。

一套 4 个配置文件 + 9 个 slash command。放到任何项目里，Claude Code 在你打终端后就已经知道你的技术栈、哪些文件不能碰、提交格式是什么、出 bug 了怎么查。

---

## 你每次开新项目，Claude 都跟失忆了一样

举两个场景你就明白了。

改一个 API，Claude 直接在组件文件里写 SQL。你去 review 的时候已经晚了——它不知道你的数据库操作全在 `src/lib/services/` 里，没人告诉过它。

每执行一个命令弹一次权限确认。`npm install` 弹一次，`git status` 弹一次，写个文件又弹一次。一个下午点了 30 次 Allow，你以为自己在写代码，其实在点按钮。

# 

* 群里的小伙伴吐槽 🤣🤣🤣

你不配置，它就默认。默认的 Claude 是什么都能做、什么都不知道的通用助手。

但通用不是好事。

这套配置做的事情就是把"通用"变成"你的项目的专属"。4 个文件，9 个命令，一次配置。

---

## 第一个文件：CLAUDE.md

这个老生常谈了，这是 Claude Code 启动时第一个读的文件。意思就是：**你写在这里的东西，Claude 会在每一次对话里自动遵守。**

我自己用的版本，精简摘录：

```
## 沟通方式  
  
- 默认中文回复；代码、命令、变量名、文件路径保持英文。  
- 结论先行，简洁直接，不先铺垫背景。  
- 不谄媚，不夸我的想法好，不说"这是个很好的问题"，不以"当然可以"开头。  
- 给真实判断。方案有问题直接指出；发现更好做法主动说明。  
  
## Git  
  
- 不自动 `git commit` 或 `git push`，除非我明确要求。  
- 提交前先展示将要提交的变更摘要。  
- commit message 使用简洁英文，并优先遵循项目历史风格。  
  
## 红线操作  
  
以下操作即使在 auto-accept 模式下也必须先问我：  
  
- 删除文件、目录或 git 历史  
- 修改 `.env`、密钥、token、证书、CI/CD 配置  
- `git push`、`git rebase`、`git reset --hard`、强制推送  
- 公开发布，例如 `npm publish`、生产部署、公开发文
```

这是全局 CLAUDE.md 的思路——跨所有项目生效的协作规则。项目级的再加一层：技术栈、目录结构、commit 格式、禁区。两个文件叠在一起，Claude 第一次打开你的项目就已经知道——跑测试是 `pnpm vitest run` 而不是 `npm test`，数据库操作全在 `src/lib/services/` 里，`migrations/` 目录别碰那是 ORM 管的。

**这个东西的力量不在"写了什么"，在于"从此不用再说"。**

我自己的习惯是，每被 Claude 坑一次，立刻加一条到 CLAUDE.md。三个月下来，这个文件成了"这个项目 Claude 犯过的所有错误的预防清单"。

但也不是只加不删。过时的规则留着只会占上下文，让 Claude 在没用的信息里找重点。同一个错误出现第二次就写进 CLAUDE.md，过时的规则就删掉，内容要精炼。

---

## 第二个文件：settings.json

用 Claude Code 最烦什么？

弹窗。每干一件事弹一次。Allow？Deny？一直 Allow？

这个文件把"该放行的放行、该锁住的锁住"写死：

**allow 白名单**——Read、Edit、写源码、写测试、跑日常命令（npm/pnpm/git）。这些都是安全操作，不应该每次都问。

**deny 黑名单**——读 .env、读密钥、写 CI/CD 配置、执行 `rm -rf`、`sudo`、`git push`、`docker`。这些要么是安全红线，要么是需要你亲自确认的重操作。

配完之后的体验：**日常操作零弹窗，危险操作自动封堵。** Claude 不会再问你"能读这个文件吗"，因为它知道哪些能读。也不会不小心把 `package-lock.json` 写坏，因为压根没权限写。

还有一个很容易被忽略的功能：**hooks**——写完文件自动格式化、类型检查自动跑、敏感文件自动拦截，这些不在 permissions 里，要另外配。代码就不放了，有会配的大佬可以在评论区说下写法。

精简版放这里，直接抄：

```
{  
  "permissions": {  
    "allow": [  
      "Read",  
      "Glob",  
      "Grep",  
      "LS",  
      "Edit",  
      "MultiEdit",  
      "Write(src/**)",  
      "Write(tests/**)",  
      "Write(docs/**)",  
      "Bash(npm run *)",  
      "Bash(pnpm *)",  
      "Bash(npm install *)",  
      "Bash(npm test *)",  
      "Bash(npx tsc *)",  
      "Bash(npx vitest *)",  
      "Bash(npx prettier *)",  
      "Bash(npx eslint *)",  
      "Bash(git status)",  
      "Bash(git diff *)",  
      "Bash(git log *)",  
      "Bash(git add *)",  
      "Bash(git commit *)",  
      "Bash(git checkout *)",  
      "Bash(git branch *)",  
      "Bash(cat *)",  
      "Bash(head *)",  
      "Bash(tail *)",  
      "Bash(wc *)",  
      "Bash(find *)",  
      "Bash(echo *)"  
    ],  
    "deny": [  
      "Read(**/.env*)",  
      "Read(**/.dev.vars*)",  
      "Read(**/*.pem)",  
      "Read(**/*.key)",  
      "Read(**/secrets/**)",  
      "Read(**/credentials/**)",  
      "Read(**/.aws/**)",  
      "Read(**/.ssh/**)",  
      "Read(**/.npmrc)",  
      "Write(**/.env*)",  
      "Write(**/secrets/**)",  
      "Write(**/.ssh/**)",  
      "Write(.github/workflows/*)",  
      "Write(package-lock.json)",  
      "Bash(rm -rf *)",  
      "Bash(sudo *)",  
      "Bash(git push *)",  
      "Bash(git merge *)",  
      "Bash(git rebase *)",  
      "Bash(npm publish *)",  
      "Bash(docker *)",  
      "Bash(curl * | sh)",  
      "Bash(wget *)",  
      "Bash(chmod *)"  
    ],  
    "defaultMode": "acceptEdits"  
  }  
}
```

allow 按你的工具链改——用 yarn 就加 Bash(yarn \*)，用 bun 就加 Bash(bun \*)。deny 那几行建议原样留着，它们是安全底线。

---

## 第三个文件：.gitignore

这个文件大家都有。但我的版本多了几行你很可能忘了加的：

```
# AI 工具本地配置  
.claude/settings.local.json  
.cursor/  
.aider*  
.continue/  
.cody/  
  
# 密钥和凭证  
*.pem  
*.key  
credentials.json  
.npmrc  
.aws/  
.ssh/
```

换句话说：**不只保护你的代码，也保护你的 AI 工具配置和密钥不泄露到 git。**

有个细节容易被忽略：`.claude/settings.local.json` 被 gitignore 了，但 `.claude/settings.json` 和 `.claude/skills/` 没有。意思是——项目配置团队共享，个人偏好自己留着。你团队里的人不用猜你的 Claude 是怎么配的，但你的 API key 也不会不小心传上去。

---

## 第四个：9 个 Slash Command（Skills）

这是最重要的部分。

你知道那种感觉吗？每次上线前，你心里有一个检查清单——类型有没有过、测试有没有跑、有没有人忘了删 console.log。但你不会每次都记得逐条过一遍。有时候赶时间，跳了一步，上线之后才发现。

Skills 就是把这个清单变成命令。你不用再临时想"还有什么没查"——敲个 `/`，它按你定的规矩跑完。

网上有现成的 skill 可以直接装，如果追求个性化也可以自己写。自己写也不复杂，就是一个 markdown 文件。

9 个 `.claude/skills/[名字]/SKILL.md` 文件，等于你自己定制了 9 个 Claude Code 命令。拿最核心的 3 个拆开看：

**`/review`** —— 审代码。不看风格问题，直接按严重程度排：CRITICAL（逻辑错误、空指针、竞态条件）→ WARNING（安全问题、N+1 查询）→ INFO（命名、垃圾代码）。输出是一张 checklist，不是一篇作文。

拿 `/review` 举个完整的例子，你放到 `.claude/skills/review/SKILL.md` 里就能用：

```
---  
name: review  
description: 适用于完成任务、实现主要功能或合并代码前，验证工作是否符合要求  
tags: 代码评审, Git工作流, 代码验证, 开发流程  
allowed-tools: Read, Grep, Glob, Bash(git *)  
---  
  
# 发起代码评审  
  
**核心原则：尽早评审，经常评审。**  
  
## 何时发起  
- 完成任务或主要功能后（强制）  
- 合并到 main 前（强制）  
- 遇到瓶颈需要新视角时（可选）  
  
## 操作步骤  
  
1. 获取变更范围：`git diff --stat` 了解改了哪些文件  
2. 按严重程度审查：  
   - CRITICAL：逻辑错误、空指针、竞态条件、安全漏洞  
   - WARNING：N+1 查询、缺少错误处理、性能隐患  
   - INFO：命名、垃圾代码、TODO  
3. 输出 checklist，按严重程度分组  
4. 结尾总结："X critical, Y warnings, Z info"  
  
## 注意事项  
- 不因"代码很简单"跳过评审  
- 严重问题立即修复后再继续  
- 次要问题记录留待后续
```

**`/commit`** —— 提交代码。自动跑 `git status` 和 `git diff`，把改动按逻辑分组，每个组一个 commit，格式严格遵循 `type(scope): description`。提交完给你一个总结，而不是让你自己翻 diff。

**`/deploy-check`** —— 上线前检查。按顺序跑：类型检查 → 测试 → lint → 构建 → 搜 `console.log` → 检查 .env 引用 → 确认没有未提交的改动。每一步 ✅ 或 ❌，全绿才敢上线。

另外 6 个是 `/test``/pr``/debug``/refactor``/docs``/security`。套路和 `/review` 一样——前面用 frontmatter 声明 name、description、allowed-tools，后面写检查步骤，照着上面的模板改就行。

**写一次，以后每次都是它替你查。**

---

## 配之前 vs 配之后

配之前，新项目打开 Claude Code 的第一分钟：手动打技术栈、点三十次 Allow、聊了几轮发现 .env 早就被读了、commit 信息格式每次都靠编。折腾完你放弃了——以后每次对话都在空白基础上开始。

配之后：打开 Claude Code，它知道你的技术栈、命令、规则，权限确认降到两三次，.env 系统级封堵想读都读不到，`/review``/commit``/deploy-check` 随时可用——你直接开始干活。

这不是"AI 变聪明了"。

**是你把一次性的解释成本，变成了可复用的配置文件。**

---

## 三种安装方式

三种装法，看你的场景。

从零开始的新项目最简单，直接把模板复制进去，填入技术栈，第一次 commit 就带着完整配置。

已有项目麻烦一点——把 CLAUDE.md 和 .gitignore 加到根目录，settings.json 合并到现有权限配置里，skills 文件夹拖进去就行。

如果你是所有项目都写代码的人，settings.json 和 skills 直接放 `~/.claude/` 全局生效。但 CLAUDE.md 还是每个项目单独写——毕竟不同项目技术栈不一样。

我自己的做法是：settings.json 和 skills 放全局，CLAUDE.md 每个项目单独写。这样权限和命令不用每个项目配一遍，但项目上下文是单独的。

---

## 别当一锤子买卖

这套模板最大的意义不是"直接用"，是"以此为起点，让它越长越像你的项目"。

CLAUDE.md 是活的。Claude 每犯一次错就加一条，"更新 CLAUDE.md，让这件事不再发生"——这是 Claude Code 里最有生产力的一句话。settings.json 随着项目长大，新工具来了加 allow，发现新的危险操作加 deny。Skills 要长出自己的版本，`/review` 里加你代码库特有的常见问题，`/commit` 里写上团队的 scope 命名规范。.gitignore 也一样，每用一个新工具，看它会不会在本地生成配置文件。

三个月后回头看，你最初的模板已经被使用习惯磨成了完全不同的形状。**这个变化本身就是价值：你的项目越来越像你，Claude 也越来越懂你。**

---

这套配置放在任何项目里，都是直接就能用的。但它真正值钱的地方，是之后你加进去的每一条规则——那都是你在这个项目里踩过的坑。

最后说一句：别觉得用不了 Anthropic 的模型就跟 Claude Code 无缘。Claude Code 本身就是顶级的 Agent 框架，接上国产大模型一样能打。

现在Deepseek-v4pro官网api折扣才2.5折。截止到月底，羊毛趁早薅！！！（这不是广告哈，毕竟咱还是小卡拉米）只有用上最顶尖的，你才算摸到了 AI 编程的门。不用，永远在门外。

抓紧行动起来！

以上

如果这篇内容对你有帮助，欢迎点个赞、点个在看，也欢迎转发给更多有需要的朋友。 你的每一次互动，都是我持续更新的动力。 想第一时间收到后续内容，记得点个关注。

感谢阅读，我们下篇见。