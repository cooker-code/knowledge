> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Claude Code 用户必装的 8 个 Hooks：从此告别"它又没听话"
author: AI的岔路口
date:
url: https://mp.weixin.qq.com/s?__biz=MzI5NTg2OTk2Ng==&mid=2247484469&idx=1&sn=62becd140e3f4d3e50f2e5c95c0055f5&chksm=ed30721799d66a56e6f0cefb8497c50ae3a099e8b6e4b2b1a439615841191a7521f0f772c84f&mpshare=1&scene=24&srcid=04131BI1KlMrYk90tvmxyFNE&sharer_shareinfo=a7f4877797f7a3606acd215b367f18e1&sharer_shareinfo_first=a7f4877797f7a3606acd215b367f18e1#rd
---

---

# Claude Code 用户必装的 8 个 Hooks：从此告别"它又没听话"

8个Claude Code Hooks

用 Claude Code 写代码，最让人抓狂的是什么？

不是它写不出来，而是它**不听话**。

你说"格式化一下"——它没格式化。你说"别碰那个文件"——它偏偏去改了。你说"跑一下测试再说"——它转头就忘。

问题出在哪？因为你写在 CLAUDE.md 里的规则，对它来说只是"建议"，大概 80% 的时候会遵守。剩下的 20% 就是你踩坑的时候。

但 Hooks 不一样。它是硬性规则，是自动触发的脚本——每次 Claude 编辑文件、执行命令、完成任务时都会强制运行，不存在"忘了"这回事。

下面这 8 个 Hooks，直接复制到 settings.json 里就能用。一次配好，永久生效。

PreToolUse vs PostToolUse

## Hooks 工作原理（30 秒搞懂）

Hooks 就是绑定在 Claude Code 操作上的自动化脚本。你设置一次，之后它们在后台默默执行，完全不需要你管。

最核心的两个时机：

**PreToolUse** —— 在 Claude 执行操作**之前**运行。你可以检查这个操作，返回退出码 2 就能直接拦截。相当于一个门卫。

**PostToolUse** —— 在 Claude 执行操作**之后**运行。用来做清理、格式化、跑测试、写日志。相当于流水线上的质检员。

```
Hooks 配置文件位置：

.claude/settings.json         项目级（通过 git 共享给团队）
~/.claude/settings.json       用户级（所有项目生效）
.claude/settings.local.json   本地（不提交到 git）
```

配置写在项目根目录的 **.claude/settings.json** 里，提交到 git 后团队所有人自动生效。

完整文档：https://code.claude.com/docs/en/hooks

图像

Hook 1-2: 自动格式化+文件保护

## 1. 自动格式化——再也不用嘴上说"跑一下 Prettier"

**痛点：** Claude 写的代码逻辑没问题，但格式乱七八糟。你在 CLAUDE.md 里写了"每次跑 Prettier"，它有时候照做，有时候选择性失忆。

**方案：** 每次文件写入后自动触发 Prettier，根本不给它"忘记"的机会。

```
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r &#x27;.tool_input.file_path&#x27; | xargs npx prettier --write 2>/dev/null; exit 0"
          }
        ]
      }
    ]
  }
}
```

Python 项目换成 **black**，Go 项目换成 **gofmt**，Rust 换成 **rustfmt**，套路一样。

这是我配的第一个 Hook，建议你也把它当成每个项目的标配。从此告别"Claude 又忘了格式化"的尴尬提交。

## 2. 拦截危险命令——别让 AI 碰你的生产环境

**痛点：** Claude 有能力执行 `rm -rf`、`git reset --hard`、`DROP TABLE`，甚至 `curl | sh`。它大概率不会这么干，但面对生产数据库，"大概率"三个字你敢赌吗？

**方案：** 在危险命令执行之前直接拦截。

创建 `.claude/hooks/block-dangerous.sh`：

```
#!/usr/bin/env bash
set -euo pipefail
cmd=$(jq -r &#x27;.tool_input.command // ""&#x27;)

dangerous_patterns=(
  "rm -rf"
  "git reset --hard"
  "git push.*--force"
  "DROP TABLE"
  "DROP DATABASE"
  "curl.*|.*sh"
  "wget.*|.*bash"
)

for pattern in "${dangerous_patterns[@]}"; do
  if echo "$cmd" | grep -qiE "$pattern"; then
    echo "Blocked: &#x27;$cmd&#x27; matches dangerous pattern &#x27;$pattern&#x27;. Propose a safer alternative." >&2
    exit 2
  fi
done
exit 0
```

然后在 settings.json 里挂上：

```
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/block-dangerous.sh"
          }
        ]
      }
    ]
  }
}
```

关键是**退出码 2**。它会阻止操作，并把错误信息发回给 Claude，让它换一种更安全的方式。退出码 0 表示放行，其他退出码只记录警告但不拦截。

Hook 3-4: 自动测试+Lint检查

## 3. 保护敏感文件——.env 和密钥文件碰都不许碰

**痛点：** Claude 能读写你项目里的任何文件，包括 `.env`、`package-lock.json`、`.pem` 密钥，这些文件一旦被改动，后果可能很严重。

**方案：** 对这些文件设置写保护，碰就拦截。

创建 `.claude/hooks/protect-files.sh`：

```
#!/usr/bin/env bash
set -euo pipefail
file=$(jq -r &#x27;.tool_input.file_path // .tool_input.path // ""&#x27;)

protected=(
  ".env*"
  ".git/*"
  "package-lock.json"
  "yarn.lock"
  "*.pem"
  "*.key"
  "secrets/*"
)

for pattern in "${protected[@]}"; do
  if echo "$file" | grep -qiE "^${pattern//\*/.*}$"; then
    echo "Blocked: &#x27;$file&#x27; is protected. Explain why this edit is necessary." >&2
    exit 2
  fi
done
exit 0
```

```
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/protect-files.sh"
          }
        ]
      }
    ]
  }
}
```

## 4. 改完代码自动跑测试——别再"搞定了"然后翻车

**痛点：** Claude 改完代码跟你说"搞定了"，你 20 分钟后准备提交才发现测试全挂了。

**方案：** 每次代码变更后自动跑测试。测试挂了，Claude 自己就能看到失败结果，当场修复。

```
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "npm run test --silent 2>&1 | tail -5; exit 0"
          }
        ]
      }
    ]
  }
}
```

`tail -5` 是为了只保留最后几行输出，避免撑爆 Claude 的上下文窗口。你要让它看到的是"3 个测试失败"，而不是 200 行完整日志。

这种即时反馈循环的效果非常显著：Claude 不再是写完代码就祈祷，而是写完、看到结果、自己修。代码质量能提升好几个档次。

Hook 5-6: 安全扫描+完成通知

## 5. 创建 PR 前必须测试通过——CI 红了丢人的事别再干了

**痛点：** Claude 写完功能就急着创建 PR，测试没跑。你的同事打开一看 CI 一片红，直接打回来。

**方案：** 测试不全绿，PR 就别想创建。

创建 `.claude/hooks/require-tests-for-pr.sh`：

```
#!/usr/bin/env bash
set -euo pipefail

if npm run test --silent; then
  exit 0
else
  echo "Tests are failing. Fix all test failures before creating a PR." >&2
  exit 2
fi
```

```
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "mcp__github__create_pull_request",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/require-tests-for-pr.sh"
          }
        ]
      }
    ]
  }
}
```

这是一道硬性关卡。测试不绿就没有 PR，Claude 会被迫先去修测试，因为退出码 2 会告诉它"操作被拦截了，原因是测试没过"。

## 6. 自动 Lint——代码风格问题在你看到之前就解决

**痛点：** Claude 写的代码能跑，但一堆 ESLint 警告、类型检查不过。你代码评审时才发现，又得打回去。

**方案：** 每次编辑后自动跑 Lint。有问题的话 Claude 自己看到就修了，根本轮不到你操心。

```
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "npx eslint --fix $(jq -r &#x27;.tool_input.file_path&#x27;) 2>&1 | tail -10; exit 0"
          }
        ]
      }
    ]
  }
}
```

这个可以和第 1 个 Hook（自动格式化）串联使用：先跑 Prettier 再跑 ESLint。等代码到你眼前的时候，格式和规范都已经处理好了。

Hook 7-8: 自动文档+依赖检查

## 7. 记录每一条命令——出了事能溯源

**痛点：** Claude 一次会话能执行大量 shell 命令。如果哪里搞炸了，你需要知道它到底做了什么、什么时候做的。

**方案：** 把每条 Bash 命令带上时间戳写入日志文件。

创建 `.claude/hooks/log-commands.sh`：

```
#!/usr/bin/env bash
set -euo pipefail
cmd=$(jq -r &#x27;.tool_input.command // ""&#x27;)
printf &#x27;%s %s\n&#x27; "$(date -Is)" "$cmd" >> .claude/command-log.txt
exit 0
```

```
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/log-commands.sh"
          }
        ]
      }
    ]
  }
}
```

记得把 `.claude/command-log.txt` 加到 `.gitignore` 里。

这在调试时特别好使：如果 Claude 三次会话之前搞坏了什么东西，翻一下日志就能精确定位是哪条命令惹的祸。

## 8. 任务完成自动提交——告别"混在一起的巨型 commit"

**痛点：** Claude 做完一个任务你忘了提交，它接着做下一个，两个毫不相关的改动混在同一个 commit 里。

**方案：** Claude 每完成一个回复，自动 commit 所有变更。

创建 `.claude/hooks/auto-commit.sh`：

```
#!/usr/bin/env bash
set -euo pipefail
git add -A
if ! git diff --cached --quiet; then
  git commit -m "chore(ai): apply Claude edit"
fi
exit 0
```

```
{
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/auto-commit.sh"
          }
        ]
      }
    ]
  }
}
```

每个任务一个原子提交，git 历史干干净净。配合 `claude -w feature-branch`（worktrees）使用，还能给每个任务搞一个隔离的功能分支，自动提交。

CLAUDE.md vs Hooks

## 完整 settings.json：复制即用

下面是所有 8 个 Hooks 合并在一个文件里的完整配置：

图像

把这个文件放到 `.claude/settings.json`，在 `.claude/hooks/` 下创建对应的脚本文件，用 `chmod +x .claude/hooks/*.sh` 加上执行权限，然后提交到 git。团队所有人自动生效。

```
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          { "type": "command", "command": ".claude/hooks/block-dangerous.sh" },
          { "type": "command", "command": ".claude/hooks/log-commands.sh" }
        ]
      },
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": ".claude/hooks/protect-files.sh" }
        ]
      },
      {
        "matcher": "mcp__github__create_pull_request",
        "hooks": [
          { "type": "command", "command": ".claude/hooks/require-tests-for-pr.sh" }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          { "type": "command", "command": "jq -r &#x27;.tool_input.file_path&#x27; | xargs npx prettier --write 2>/dev/null; exit 0" },
          { "type": "command", "command": "npx eslint --fix $(jq -r &#x27;.tool_input.file_path&#x27;) 2>&1 | tail -10; exit 0" }
        ]
      }
    ],
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          { "type": "command", "command": ".claude/hooks/auto-commit.sh" }
        ]
      }
    ]
  }
}
```

一个普通的 Claude Code 配置和一个顶级配置之间的差距，不在模型，不在提示词，而在 Hooks。它们是那些你不注意的时候默默帮你兜底的东西，帮你拦住那些本来要在代码评审甚至生产环境才会发现的问题。

建议先从 Hook #1（自动格式化）和 #2（拦截危险命令）开始配。光这两个就能挡住最常见的坑。其他的按需添加。