> 已吸收至：[[02_Agent与AI工程/0206_AI编码方式/020601_Workflow/020601_核心知识点/Workflow编排与验证闭环|Workflow编排与验证闭环]]
---
title: 优化Claude工作流：settings.json文件设置全解析
author: 代码麻辣烫
date: 一叶扁舟一叶扁舟
url: https://mp.weixin.qq.com/s?__biz=Mzk0NjY0MTc0NA==&mid=2247499996&idx=1&sn=64c6db005a83983b0323e63138ced08e&chksm=c29900a9ab88096e4953b7300df3783f693637fe7387de126efcb2593e243791704551ffc99d&mpshare=1&scene=24&srcid=0430Ck5kL6I5i9Gz7mJfFJBw&sharer_shareinfo=dbc74bf3f26effe1c8663090a90b9cba&sharer_shareinfo_first=dbc74bf3f26effe1c8663090a90b9cba#rd
---

封面图片

Claude Code 的用户们常常遇到这样的烦恼：

“我能编辑这个文件吗？”“我能运行这个测试吗？”

你一次又一次地点选“允许”，在每次会话中多达30次，每次都打断了你的工作流程。

解决方案就是 **「settings.json」**。

一个文件，几个规则，就能让 Claude 停止询问你每天执行的命令权限，同时保留对那些可能造成破坏的命令的限制！

大多数人并不知道这个文件的存在。那些知道的人，他们的版本往往只有三行，几乎起不到什么作用。以下是如何正确设置它的方法👇

## 文件位置

就像 CLAUDE.md 一样，有三个级别：

```
~/.claude/settings.json           → 全局（每个项目）
.claude/settings.json             → 项目（与团队共享，在 git 中）
.claude/settings.local.json       → 本地（个人，gitignored）
```

全局设置是你希望的权限。项目设置是团队共享的规则。本地设置是不应该进入 git 的个人覆盖。

规则会在不同级别间合并。如果你的全局设置允许 **「Bash(npm \*)」**，而你的项目设置拒绝 **「Bash(npm publish)」**，那么两者都会生效。

拒绝总是优先于允许。

## 60秒掌握权限系统

三个数组控制着一切：

```
{
  "permissions": {
    "allow": [],
    "deny": [],
    "ask": []
  }
}
```

**「allow」** - Claude 在使用这些工具时不会询问，不会出现确认对话框。

**「deny」** - Claude 绝对不能使用这些，完全被阻止。

**「ask」** - Claude 每次都会询问权限。

评估顺序是：先看拒绝，再看询问，最后看允许。第一个匹配的规则为准。对于同一工具，拒绝规则总是优先于允许规则。

规则格式：**「ToolName」** 或 **「ToolName」**(pattern)。

```
"Bash"              → 所有 bash 命令（危险）
"Bash(npm install)"  → 仅限 npm install
"Bash(npm run *)"    → 任何 npm run 脚本
"Bash(git *)"        → 任何 git 命令
"Write(src/**)"      → 仅限写入 src/ 下的文件
"Read(.env*)"        → 读取任何 .env 文件
```

星号前的空格很重要。Bash(ls \*) 匹配 ls -la，但不匹配 lsof。通配符是全局模式，不是正则表达式。

📸 图片 1：

## 5种权限模式

与其单独设置规则，你也可以设置默认模式：

```
{
  "permissions": {
    "defaultMode": "default"
  }
}
```

```
default          → 对所有危险事项进行询问
acceptEdits      → 自动批准文件编辑，但对 bash 命令仍会询问
plan             → 只读模式，不允许更改
dontAsk          → 拒绝所有未明确允许的操作
bypassPermissions → 批准所有操作（仅在容器/CI中使用）
```

在会话期间快速切换：按 **「Shift+Tab」** 在默认、acceptEdits 和 plan 模式之间循环，无需触碰任何配置。

## 允许什么（安全列表）

这些是你每天运行数十次的命令。让 Claude 在不询问的情况下运行它们，每个会话节省 5-10 分钟：

```
{
  "permissions": {
    "allow": [
      "Read",
      "Glob",
      "Grep",
      "LS",
      "Bash(npm run *)",
      "Bash(npm install *)",
      "Bash(npm test *)",
      "Bash(npx tsc *)",
      "Bash(npx vitest *)",
      "Bash(git status)",
      "Bash(git diff *)",
      "Bash(git log *)",
      "Bash(git add *)",
      "Bash(git commit *)",
      "Bash(git checkout *)",
      "Bash(git branch *)",
      "Write(src/**)",
      "Edit",
      "MultiEdit"
    ]
  }
}
```

注意模式：读取操作完全开放（Read, Glob, Grep, LS）。Bash 命令只允许特定工具（npm, git, 测试运行器）。写入访问权限仅限于 src/。

## 拒绝什么（安全网）

这些是可以造成真正损害的命令。无论如何都要阻止它们：

```
{
  "permissions": {
    "deny": [
      "Read(.env*)",
      "Read(**/secrets/**)",
      "Write(.env*)",
      "Write(production.*)",
      "Write(.github/workflows/*)",
      "Bash(rm -rf *)",
      "Bash(sudo *)",
      "Bash(git push *)",
      "Bash(git merge *)",
      "Bash(npm publish *)",
      "Bash(docker *)",
      "Bash(curl * | sh)",
      "Bash(wget *)"
    ]
  }
}
```

关键原则：Claude 可以读取你的代码，编写你的代码，运行你的测试，并提交更改。但它永远不能读取秘密，推送到远程，递归删除文件，或运行任何带有 sudo 的命令。

危险的操作被挡在拒绝墙后面。

## 在 settings.json 中添加钩子

设置和钩子位于同一个文件中。每次编辑后自动格式化代码，提交前自动检查：

```
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write(*.py)",
        "hooks": [
          {
            "type": "command",
            "command": "python -m black $file"
          }
        ]
      },
      {
        "matcher": "Write(*.ts)",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write $file"
          }
        ]
      }
    ]
  }
}
```

每个 .py 文件都自动使用 Black 格式化。

每个 .ts 文件都自动使用 Prettier 格式化。

无需提示，无需手动步骤。

## 团队共享设置

将项目级设置放在 **「.claude/settings.json」** 中，并提交到 git。你的整个团队将获得相同的权限：

```
{
  "permissions": {
    "allow": [
      "Read",
      "Glob",
      "Grep",
      "Bash(npm run *)",
      "Bash(npm test *)"
    ],
    "deny": [
      "Read(.env*)",
      "Bash(npm publish *)",
      "Bash(rm -rf *)",
      "Write(production.*)"
    ],
    "defaultMode": "acceptEdits"
  }
}
```

新团队成员克隆仓库，打开 Claude Code，一切都预配置好了。无需设置，无需“我应该允许哪些命令”的问题，也无需担心有人不小心允许 **「Bash(rm -rf \*)」**。

Anthropic 的 Boris Cherny 正是这样做的：团队共享 settings.json，预批日常命令和阻止风险命令。

## 完整的文件（复制粘贴就绪）

典型的 Node.js/TypeScript 项目的完整 settings.json。

将其复制到 **「~/.claude/settings.json」** 用于全局使用，或 **「.claude/settings.json」** 用于项目特定：

```
{
  "permissions": {
    "allow": [
      "Read",
      "Glob",
      "Grep",
      "LS",
      "Edit",
      "MultiEdit",
      "Write(src/**)",
      "Write(tests/**)",
      "Write(docs/**)",
      "Bash(npm run *)",
      "Bash(npm install *)",
      "Bash(npm test *)",
      "Bash(npx tsc *)",
      "Bash(npx vitest *)",
      "Bash(npx prettier *)",
      "Bash(npx eslint *)",
      "Bash(git status)",
      "Bash(git diff *)",
      "Bash(git log *)",
      "Bash(git add *)",
      "Bash(git commit *)",
      "Bash(git checkout *)",
      "Bash(git branch *)",
      "Bash(cat *)",
      "Bash(head *)",
      "Bash(tail *)",
      "Bash(wc *)",
      "Bash(find *)",
      "Bash(echo *)"
    ],
    "deny": [
      "Read(.env*)",
      "Read(**/secrets/**)",
      "Write(.env*)",
      "Write(production.*)",
      "Write(.github/workflows/*)",
      "Write(package-lock.json)",
      "Bash(rm -rf *)",
      "Bash(sudo *)",
      "Bash(git push *)",
      "Bash(git merge *)",
      "Bash(git rebase *)",
      "Bash(npm publish *)",
      "Bash(docker *)",
      "Bash(curl * | sh)",
      "Bash(wget *)",
      "Bash(chmod *)",
      "Bash(chown *)"
    ],
    "defaultMode": "acceptEdits"
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write(*.ts)",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write $file"
          }
        ]
      },
      {
        "matcher": "Write(*.tsx)",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write $file"
          }
        ]
      }
    ]
  }
}
```

复制它。根据你的文件夹结构调整 Write 范围。

添加或删除 Bash 规则以适应你的技术栈（将 npm 替换为 pnpm，添加 python 命令，等等）。

拒绝列表对每个项目来说大致相同。

## 前后对比

**「BEFORE settings.json：」**

* 每次会话30-40个权限提示 - 每次运行 npm install 时点击“允许” - 一不小心允许 rm -rf 一次，然后惊慌失措 - 新团队成员手动配置一切 - 每两分钟打断一次流程状态 **「AFTER settings.json：」** - 每次会话0-3个权限提示 - 常规命令立即运行 - 危险命令在配置级别被阻止 - 团队共享一个文件，每个人都预配置 - 流程状态保持不变

两分钟的设置。之后的每个会话都会更快。

感谢阅读 🙏🏼