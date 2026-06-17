---
title: Claude Code Hooks 实战：5个配置让你少操心
author: 92year
date: 
url: https://mp.weixin.qq.com/s?__biz=MzY4MTE5ODQ2MQ==&mid=2247483846&idx=1&sn=6b96c6f2f5d6b9681e0d15c01510d6e2&chksm=f2ea6991b7372e83594ce81c7fea3123b2654f906de89717642397c78e3c8376742cda446904&mpshare=1&scene=24&srcid=0408CHN849kNLd9vEMga8Xgk&sharer_shareinfo=ab1d5c6b413d7606a1edb5a2ca37334e&sharer_shareinfo_first=ab1d5c6b413d7606a1edb5a2ca37334e#rd
---

# Claude Code Hooks 实战：5个配置让你少操心

Claude Code 在终端里干活挺靠谱，但有些事情你不想每次都手动盯——文件改完要格式化、某些文件不能碰、跑完了得提醒你。Hooks 就是干这个的：你写好规则，它自动执行，不靠AI"记得住"，靠配置强制生效。

今天分享5个我在用的 Hook 配置，复制粘贴就能跑。

## Hooks 是什么

Hooks 是写在 `~/.claude/settings.json` 里的 shell 命令，在特定时机自动触发。比如文件被编辑后、命令执行前、AI等你输入时。跟 git hooks 思路一样，不过触发点是 Claude Code 的操作。

所有配置都在 settings.json 的 hooks 字段里，格式长这样：

```
{
  "hooks": {
    "事件名": [{
      "matcher": "匹配条件",
      "hooks": [{
        "type": "command",
        "command": "要执行的shell命令"
      }]
    }]
  }
}
```

## 配置1：跑完了弹通知

Claude Code 跑长任务的时候你肯定想切去干别的。这个 hook 让它跑完自动弹桌面通知，不用一直盯着终端。

```
"Notification": [{
  "matcher": "",
  "hooks": [{
    "type": "command",
    "command": "osascript -e 'display notification \"干完了\" with title \"Claude Code\"'"
  }]
}]
```

macOS 用 osascript，Linux 换成 `notify-send`。

## 配置2：改完文件自动格式化

Claude 写代码缩进经常不统一。这个 hook 在每次文件被编辑后自动跑 Prettier。

```
"Edit|Write": [{
  "matcher": "\\.(js|ts|jsx|tsx|css|json|md)$",
  "hooks": [{
    "type": "command",
    "command": "npx prettier --write \"$CLAUDE_FILE_PATH\""
  }]
}]
```

matcher 用正则匹配文件类型。只对 js/ts/css/json/md 生效，其他文件不动。

## 配置3：禁止碰特定文件

有些文件不能让 AI 改——比如 .env、密钥文件、数据库迁移。这个 hook 直接拦截，不是靠 CLAUDE.md 里写"请不要修改"然后祈祷它听话。

```
"Edit|Write": [{
  "matcher": "(\\.env|secrets\\.json|migrations/)",
  "hooks": [{
    "type": "command",
    "command": "echo 'BLOCKED' && exit 1"
  }]
}]
```

exit 1 会让操作失败。比在 CLAUDE.md 里写规则靠谱得多——那个是建议，这个是强制。

## 配置4：压缩上下文后自动补回关键信息

Claude Code 上下文太长会自动压缩，压缩后它可能忘记你之前交代的规则。这个 hook 在压缩后自动把关键提示词塞回去。

```
"Compaction": [{
  "matcher": "",
  "hooks": [{
    "type": "command",
    "command": "cat ~/.claude/compaction-reminder.md"
  }]
}]
```

提前在 `~/.claude/compaction-reminder.md` 写好你最重要的几条规则，压缩后自动注入。

## 配置5：命令执行前记录日志

想知道 Claude Code 到底跑了什么命令？这个 hook 把每条命令记到日志文件里。

```
"PreToolUse": [{
  "matcher": "Bash",
  "hooks": [{
    "type": "command",
    "command": "echo \"$(date): $CLAUDE_TOOL_INPUT\" >> ~/.claude/command-audit.log"
  }]
}]
```

跑了一天之后翻翻 command-audit.log，你会发现 Claude 执行的命令比你想象的多得多。

## 使用提醒

在 Claude Code 里输入 `/hooks` 可以查看当前生效的所有 hook，但只能看不能改，改的话直接编辑 settings.json。

多个 hook 可以挂在同一个事件上，按数组顺序执行。一个 hook 返回 exit 1 会阻止后续操作。

Hooks 跑的是真正的 shell 命令，不是 AI 生成的。所以不会出现"有时候执行有时候忘了"的问题。确定性，是 hooks 跟 CLAUDE.md 规则最大的区别。

配置文件位置：`~/.claude/settings.json`（全局）或项目根目录 `.claude/settings.json`（项目级）。

官方文档：code.claude.com/docs/zh-CN/hooks-guide

关注我，每天一篇AI实战，带代码，看完能用。