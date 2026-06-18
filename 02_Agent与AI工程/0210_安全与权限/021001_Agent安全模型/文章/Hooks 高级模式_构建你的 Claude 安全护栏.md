---
title: Hooks 高级模式:构建你的 Claude 安全护栏
author: 匠心格物
date: 何谓第一等事何谓第一等事
url: https://mp.weixin.qq.com/s?__biz=MzkxNDczMDg0Nw==&mid=2247485350&idx=1&sn=2e11d00accd7af92820ca7d8cf010c16&chksm=c0ffce6ab01276c2d89bdf385441cea0fdbf34c1f4659d399b6743d81f77843c08191153535e&mpshare=1&scene=24&srcid=0530X3TRw9kvdXAM8J3UQMB6&sharer_shareinfo=3ca0749829b7311a4774bd68daaeaa90&sharer_shareinfo_first=3ca0749829b7311a4774bd68daaeaa90#rd
---

Hooks 高级模式封面

关键词：Claude Code、Hooks、AI 安全护栏、AI 编程、Claude Hooks 高级模式、on-demand hooks、SessionStart Hook、settings.json 权限、/careful 模式、/freeze 限制编辑、拦截 rm -rf、Claude 跑偏修改、生产环境 AI 安全、AI Agent 权限管理、Boris Cherny

---

[一期第 04 篇讲了 Hooks 的基本面:PreToolUse、PostToolUse、Stop,以及五个实用配方](https://mp.weixin.qq.com/s?__biz=MzkxNDczMDg0Nw==&mid=2247485251&idx=1&sn=dcb832be482d9a6ddfffa5e83359baf2&scene=21#wechat_redirect)。那篇回答的问题是"Hooks 能做什么"。

这篇回答另一个问题:Hooks 该拦住什么。

你给 Claude 的权限越大,它能帮你干的事越多。但权限大了,翻车的后果也大。`rm -rf /`、`DROP TABLE users`、`git push --force origin main`,这些命令你自己手抖都可能误敲,何况一个根据上下文推理的 AI。

Boris Cherny 在推特上讲过一条设计原则:

> ★
>
> use hooks to deterministically run logic as part of the agent lifecycle.

关键词是 deterministically。Hooks 不是"建议",是确定性的拦截。Claude 想做某件事,Hook 说不行,它就做不了。没有商量。

---

## On-demand Hooks:Skill 作用域内的钩子

[04 篇讲的 Hooks](https://mp.weixin.qq.com/s?__biz=MzkxNDczMDg0Nw==&mid=2247485251&idx=1&sn=dcb832be482d9a6ddfffa5e83359baf2&scene=21#wechat_redirect)都写在 `settings.json` 里,全局生效。问题是,不是所有拦截都该全局生效。

举个例子:你写了个 `/deploy` Skill,希望部署时强制检查测试是否通过。但平时写代码时不需要这个检查,加了反而烦人。

On-demand Hooks 就是把 Hook 写进 Skill 的 frontmatter 里。只有这个 Skill 运行时,Hook 才生效。

```
---  
name: deploy  
description: 部署到生产环境  
hooks:  
  PreToolUse:  
    - matcher: Bash  
      command: "npm test 2>&1 | tail -1 | grep -q 'passed' || (echo 'BLOCKED: 测试未通过,禁止部署' && exit 1)"  
---
```

Thariq 在推特上演示过两个 on-demand Hooks 的典型用法:

> ★
>
> use on-demand hooks in skills — /careful blocks destructive commands, /freeze blocks edits outside a directory.

这两个 Skill 值得单独说。

---

## /careful 模式:拦截破坏性命令

  

/careful vs /freeze 两种安全模式对比

`/careful` 是一个安全护栏 Skill。激活它之后,Claude 在执行任何 Bash 命令前,都会先过一遍"黑名单检查"。

它拦截的东西包括:

* `rm -rf` — 递归删除
* `DROP TABLE` / `DELETE FROM` — 数据库破坏
* `git push --force` — 强制推送覆盖远程历史
* `git reset --hard` — 丢弃本地所有修改
* `kubectl delete` — 删除 K8s 资源
* `chmod 777` — 开放所有权限

实现原理很简单,就是一个 PreToolUse Hook:

```
---  
name:careful  
description:安全模式,拦截破坏性命令。当用户说"小心点"、  
"be careful"、"safetymode"、"prodmode"时激活。  
hooks:  
PreToolUse:  
    -matcher:Bash  
      command:|  
        echo "$CLAUDE_TOOL_INPUT_COMMAND" | grep -qiE \  
          '(rm\s+-rf|DROP\s+TABLE|DELETE\s+FROM|git\s+push\s+--force|git\s+reset\s+--hard|kubectl\s+delete|chmod\s+777)' \  
        && echo "BLOCKED: 检测到破坏性命令,已拦截。如需执行请退出 /careful 模式。" \  
        && exit 1 \  
        || true  
---
```

用法:在 Claude Code 里输入 `/careful`,从这一刻起,所有破坏性命令都被拦截。想恢复正常模式,开一个新会话就行。

什么时候该用它?两个场景:

1. 你在调试生产环境的数据。
2. 你让 Claude 跑一个你不太确定会做什么的长任务。

别觉得"Claude 应该不会乱删东西"。它大部分时候不会。但"大部分时候"和"永远不会"之间,差一个 `rm -rf`。

---

## /freeze 模式:限制编辑范围

`/freeze` 解决另一个常见问题:调试的时候,Claude 改着改着就跑偏了。

你让它修 `src/api/auth.ts` 里的一个 bug,它发现 `src/utils/crypto.ts` 也有问题,顺手改了。然后发现 `src/config/index.ts` 的类型定义跟 crypto 不兼容,又改了。三个文件都改了,你只想修一个 bug。

`/freeze` 让你指定一个目录,Claude 只能在这个目录里编辑文件。出了这个范围,Edit 和 Write 工具直接报错。

```
> /freeze src/api/
```

从此刻起,Claude 只能改 `src/api/` 下的文件。它可以读任何文件(毕竟调试需要看上下文),但写操作被锁死在 `src/api/` 里。

实现方式也是 on-demand Hook,拦截 Edit 和 Write 工具:

```
---  
name:freeze  
description:限制编辑范围到指定目录。防止调试时误改其他模块。  
argument-hint:"[directory]"  
hooks:  
PreToolUse:  
    -matcher:Edit  
      command:|  
        ALLOWED_DIR="$1"  
        echo "$CLAUDE_TOOL_INPUT_FILE_PATH" | grep -q "^$ALLOWED_DIR" \  
        || (echo "BLOCKED: /freeze 模式下只能编辑 $ALLOWED_DIR 内的文件" && exit 1)  
    -matcher:Write  
      command:|  
        ALLOWED_DIR="$1"  
        echo "$CLAUDE_TOOL_INPUT_FILE_PATH" | grep -q "^$ALLOWED_DIR" \  
        || (echo "BLOCKED: /freeze 模式下只能编辑 $ALLOWED_DIR 内的文件" && exit 1)  
---
```

想解除限制?输入 `/unfreeze` 就行。

---

## SessionStart Hook:启动时自动加载上下文

每次打开 Claude Code,你是不是都要先说一遍"我在做什么项目、上次做到哪了、今天要干什么"?

SessionStart Hook 在 Claude 启动时自动运行。你可以用它动态加载上下文。

```
{  
  "hooks": {  
    "SessionStart": [  
      {  
        "matcher": "*",  
        "hooks": [  
          {  
            "type": "command",  
            "command": "cat ~/.claude/daily-context.md 2>/dev/null || echo '无上下文'"  
          }  
        ]  
      }  
    ]  
  }  
}
```

每天早上开工前,你往 `~/.claude/daily-context.md` 写一句"今天要做 X 功能",Claude 一启动就知道你要干什么。

更高级的玩法:让 SessionStart Hook 去查你的项目管理工具(Jira、Linear、GitHub Issues),自动拉取今天分配给你的任务。

```
{  
  "hooks": {  
    "SessionStart": [  
      {  
        "matcher": "*",  
        "hooks": [  
          {  
            "type": "command",  
            "command": "gh issue list --assignee @me --state open --limit 5 --json title,url | jq -r '.[] | \"- [\\(.title)](\\(.url))\"'"  
          }  
        ]  
      }  
    ]  
  }  
}
```

Claude 一开机就看到你的待办清单,不用你说一个字。

---

## PermissionRequest Hook:把审批推到手机上

Claude Code 遇到需要权限的操作时,默认在终端里弹一个 y/n 确认。你得盯着终端看。

PermissionRequest Hook 让你把这个确认推到别的地方,WhatsApp、Slack、手机通知,随你。

```
{  
  "hooks": {  
    "PermissionRequest": [  
      {  
        "matcher": "*",  
        "hooks": [  
          {  
            "type": "command",  
            "command": "curl -s -X POST 'https://your-webhook.com/notify' -d '{\"text\": \"Claude 请求执行: $CLAUDE_TOOL_NAME\"}'"  
          }  
        ]  
      }  
    ]  
  }  
}
```

场景:你让 Claude 跑一个长任务,然后去吃午饭。它中途需要权限确认,给你手机发条消息。你在手机上点"允许",它继续跑。

不用在电脑前干等着了。

---

## Stop Hook:Claude 停了?自动 poke 它继续

Claude 有时候会"自认为做完了"就停下来,但其实还差几步。

Stop Hook 在 Claude 停止时触发。你可以用它检查任务是否真的完成,如果没完成就 poke 它继续。

```
{  
  "hooks": {  
    "Stop": [  
      {  
        "matcher": "*",  
        "hooks": [  
          {  
            "type": "command",  
            "command": "test -f .claude/task-checklist.md && grep -c '\\[ \\]' .claude/task-checklist.md | grep -q '^0$' || echo 'CONTINUE: 还有未完成的检查项,请继续'"  
          }  
        ]  
      }  
    ]  
  }  
}
```

逻辑:检查 `task-checklist.md` 里还有没有未勾选的项。有的话,输出 CONTINUE 让 Claude 继续干。

这个 Hook 配合 Skill 使用效果更好。在 Skill 里定义 checklist,在 Stop Hook 里检查 checklist 是否完成。Claude 想偷懒都偷不了。

---

## Hooks + settings.json:两层护栏各管各的

  

双层安全护栏架构：settings.json + Hooks

很多人分不清 Hooks 和 `settings.json` 的权限配置有什么区别。

一句话:`settings.json` 管"能不能做",Hooks 管"做的时候怎么拦"。

### settings.json 的权限控制

```
{  
  "permissions": {  
    "allow": [  
      "Bash(npm test)",  
      "Bash(npm run lint)",  
      "Read",  
      "Glob"  
    ],  
    "deny": [  
      "Bash(curl *)",  
      "Bash(wget *)"  
    ]  
  }  
}
```

这是白名单 / 黑名单机制。`allow` 里的操作不需要确认,`deny` 里的操作直接禁止。

### Hooks 的运行时拦截

```
{  
  "hooks": {  
    "PreToolUse": [  
      {  
        "matcher": "Bash",  
        "hooks": [  
          {  
            "type": "command",  
            "command": "echo \"$CLAUDE_TOOL_INPUT_COMMAND\" | python3 safety_check.py"  
          }  
        ]  
      }  
    ]  
  }  
}
```

Hooks 在运行时检查具体内容,可以做更细粒度的判断。

两者的关系:

| 层级 | 机制 | 管什么 | 例子 |
| --- | --- | --- | --- |
| settings.json | 权限白名单/黑名单 | 工具级别的开关 | 禁止所有 curl 命令 |
| Hooks | 运行时拦截 | 命令内容级别的检查 | 允许 curl 但禁止 curl 到生产域名 |

settings 是大门,Hooks 是安检。大门说"你不能进",你就进不了。大门放你进了,安检还会检查你带了什么。

---

## 防 Prompt Injection:用 Hooks 检查命令安全性

  

Hooks 防 Prompt Injection 攻击链

这个场景不太常见,但值得知道。

如果你让 Claude 处理用户输入(比如读用户提交的 issue、处理用户上传的文件),恶意用户可能在内容里塞 prompt injection,试图让 Claude 执行危险命令。

Hooks 能在最后一道防线上拦住它:

```
{  
  "hooks": {  
    "PreToolUse": [  
      {  
        "matcher": "Bash",  
        "hooks": [  
          {  
            "type": "command",  
            "command": "python3 -c \"\nimport sys, os, re\ncmd = os.environ.get('CLAUDE_TOOL_INPUT_COMMAND', '')\npatterns = [\n    r'curl.*\\|.*sh',\n    r'eval\\s+\\$',\n    r'base64.*decode',\n    r'\\bsudo\\b',\n    r'>\\/etc\\/',\n]\nfor p in patterns:\n    if re.search(p, cmd):\n        print(f'BLOCKED: 检测到可疑命令模式: {p}')\n        sys.exit(1)\n\""  
          }  
        ]  
      }  
    ]  
  }  
}
```

这个 Hook 检查 Claude 即将执行的命令里有没有可疑模式:`curl | sh`(远程执行)、`eval $`(动态执行)、`base64 decode`(混淆)、`sudo`(提权)、写 `/etc/`(系统文件修改)。

不是银弹,但它是确定性的。不管 Claude 被什么 prompt 忽悠了,只要命令匹配模式,就被拦住。

---

## 你的安全护栏搭对了吗

  

Hooks 生命周期:5 个拦截时机

三个问题:

1. 你有没有在操作生产数据时开 `/careful`?如果没有,你在裸奔。
2. 你调试 bug 时有没有用 `/freeze` 限制编辑范围?如果没有,Claude 很可能在"顺手"帮你改别的文件。
3. 你的 settings.json 和 Hooks 是不是在各管各的事?如果你把所有安全逻辑都堆在 Hooks 里,却没配 settings 的权限白名单,那安检形同虚设,因为大门根本没锁。

全对的话,你的 Claude Code 安全性已经比大多数个人开发者强了。

---

## 一句话收尾

权限越大,护栏越要硬。Hooks 不是"建议 Claude 别乱来",是确定性地拦住它。你信任 Claude 的能力,但你锁死它的边界。这才是正确的人机协作姿势。

---

## 下一篇预告

Hooks 解决了"Claude 在干活的时候怎么拦"。但如果你希望 Claude 在你不在的时候也能干活呢?

下一篇我们讲 Scheduled Tasks:用 `/loop` 做本地定时循环,用 `/schedule` 做云端 cron,让 Claude 在你下班后继续交付。Boris 用 `/loop 5m /babysit` 自动处理 code review、自动 rebase、送 PR 进生产。一觉醒来,3 个 PR 已经合并了。

14 篇:Scheduled Tasks 与 Routines,让 Claude 在你下班后继续交付

---

## 本文引用的推文

* use hooks to deterministically run logic as part of the agent lifecycle · @bcherny
* use on-demand hooks in skills — /careful blocks destructive commands, /freeze blocks edits outside a directory · @trq212

---