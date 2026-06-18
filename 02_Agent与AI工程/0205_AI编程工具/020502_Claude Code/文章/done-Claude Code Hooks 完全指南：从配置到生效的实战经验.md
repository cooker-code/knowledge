> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/Claude Code Hooks权限与Skill治理|Claude Code Hooks权限与Skill治理]]、[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Claude Code Hooks 完全指南：从配置到生效的实战经验
author: AI贺贺
date:
url: https://mp.weixin.qq.com/s?__biz=MzAxNjIzOTkyOQ==&mid=2449866401&idx=1&sn=04c2d8a9d3cad7919352717d97062e7a&chksm=8dd6ff14b72ad8e3c68cda247abe0df83d6725a0a320bc3253f05e4923978db28ec27fcfae37&mpshare=1&scene=24&srcid=0104hjGcIUBA5PR25IpCQQeL&sharer_shareinfo=7d58d7aa9a8105ec228cb0d508a9b884&sharer_shareinfo_first=7d58d7aa9a8105ec228cb0d508a9b884#rd
---

> 本文记录了我在配置 Claude Code Hooks 时遇到的问题、排查过程和最终解决方案，希望能帮助遇到类似问题的开发者。

## 背景

作为一个重度使用 Claude Code 的开发者，我希望在 Claude 完成任务后能收到声音提醒，这样就不用一直盯着屏幕等待了。Claude Code 提供了强大的 Hooks 机制，理论上可以轻松实现这个需求，但实践中却遇到了不少坑。

## Hooks 是什么？

Claude Code Hooks 是一个强大的自动化机制，允许你在特定事件发生时自动执行自定义命令。这些事件包括：

* **SessionStart** - Claude Code 启动时
* **Stop** - Claude 完成任务时
* **PostToolUse** - 工具执行后（如执行 Bash 命令后）
* **UserPromptSubmit** - 用户提交 prompt 时
* **Notification** - 发送通知时
* 以及更多...

## 我的需求

很简单：**当 Claude 完成任务时，播放一个提示音**。

## 配置过程

### 第一步：基础配置

根据官方文档，我在 `.claude/settings.local.json` 中添加了配置：

```
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "afplay /System/Library/Sounds/Glass.aiff"
          }
        ]
      }
    ]
  }
}
```

配置看起来很简单，重启 Claude Code 后...什么都没发生。

### 第二步：排查之路

#### 尝试 1：检查配置格式

首先验证 JSON 格式是否正确：

```
python3 -m json.tool .claude/settings.local.json
```

格式正确 ✅

#### 尝试 2：测试命令

手动执行命令确认可用：

```
afplay /System/Library/Sounds/Glass.aiff
```

声音正常播放 ✅

#### 尝试 3：添加日志调试

修改配置，添加日志记录：

```
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "echo \"[$(date)] Hook triggered\" >> /tmp/claude-hook.log && afplay /System/Library/Sounds/Glass.aiff"
          }
        ]
      }
    ]
  }
}
```

重启后，日志文件依然不存在 ❌

#### 尝试 4：测试其他 Hook

为了确认 Hooks 系统本身是否工作，尝试了 `SessionStart`、`PostToolUse`、`UserPromptSubmit` 等多个 Hook...

**全部失败** ❌

### 第三步：Debug 日志救星

使用 `claude --debug` 启动后，查看调试日志：

```
cat ~/.claude/debug/[session-id].txt
```

**关键发现：**

```
[DEBUG] Skipping SessionStart:startup hook execution - workspace trust not accepted
[DEBUG] Skipping Stop hook execution - workspace trust not accepted
[DEBUG] Skipping UserPromptSubmit hook execution - workspace trust not accepted
```

所有 Hooks 都被跳过了，原因是：**workspace trust not accepted**（工作区信任未接受）！

## 问题根源

Claude Code 出于安全考虑，**不会在不受信任的工作区中执行 Hooks**。这是一个重要的安全机制，因为 Hooks 可以执行任意系统命令。

虽然全局配置文件 `~/.claude/settings.json` 中的 `hasTrustDialogAccepted` 是 `true`，但 Claude Code 为**每个工作区**维护独立的信任状态，存储在 `~/.claude.json` 文件中。

### 检查信任状态

```
cat ~/.claude.json | python3 -m json.tool | grep -B5 -A5 "hasTrustDialogAccepted"
```

果然发现：

```
{
  "/Users/bytedance": {
    "hasTrustDialogAccepted": false,  // 当前工作区未信任
    "hasTrustDialogHooksAccepted": false  // Hooks 未被授权
  }
}
```

## 解决方案

### 方法 1：手动修改配置文件（推荐）

编写脚本批量更新所有工作区的信任设置：

```
import json

# 读取配置
with open('/Users/bytedance/.claude.json', 'r') as f:
    config = json.load(f)

# 递归更新所有工作区的信任设置
def update_trust(obj):
    if isinstance(obj, dict):
        if'hasTrustDialogAccepted'in obj:
            obj['hasTrustDialogAccepted'] = True
        if'hasTrustDialogHooksAccepted'in obj:
            obj['hasTrustDialogHooksAccepted'] = True

        for value in obj.values():
            update_trust(value)

update_trust(config)

# 保存配置
with open('/Users/bytedance/.claude.json', 'w') as f:
    json.dump(config, f, indent=2)

print("✅ 配置已更新，请重启 Claude Code")
```

### 方法 2：通过 UI 接受信任（推荐新手）

1. 重启 Claude Code
2. 等待工作区信任对话框出现
3. 点击"信任此工作区"
4. 确认允许执行 Hooks

## 最终配置

经过优化，我的最终配置非常简洁：

```
{
  "permissions": {
    "allow": [
      "mcp__tavily__tavily-search",
      "Skill(skill-creator)",
      "mcp__lark-mcp__im_v1_chat_list",
      "mcp__lark-mcp__im_v1_message_list"
    ]
  },
"hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "/usr/bin/afplay /System/Library/Sounds/Glass.aiff"
          }
        ]
      }
    ]
  }
}
```

**为什么只保留 Stop Hook？**

* ❌ **SessionStart** - 每次启动都响，太频繁
* ❌ **PostToolUse** - 每次工具执行都响，非常干扰
* ✅ **Stop** - 只在任务完成时响一次，完美！

## Hooks 的实用场景

除了声音通知，Hooks 还有很多实用场景：

### 1. 代码自动格式化

```
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "prettier --write $FILE_PATH"
          }
        ]
      }
    ]
  }
}
```

### 2. 自动运行测试

```
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "npm test -- --related $FILE_PATH"
          }
        ]
      }
    ]
  }
}
```

### 3. Git 自动提交

```
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "git add . && git commit -m 'Auto-commit by Claude Code'"
          }
        ]
      }
    ]
  }
}
```

### 4. 任务完成通知（跨平台）

macOS：

```
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude has finished!\" with title \"Task Complete\"'"
          }
        ]
      }
    ]
  }
}
```

Linux（需要 libnotify）：

```
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "notify-send 'Claude Code' 'Task completed!'"
          }
        ]
      }
    ]
  }
}
```

### 5. 发送消息到飞书/钉钉

```
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "curl -X POST 'https://open.feishu.cn/open-apis/bot/v2/hook/YOUR_WEBHOOK' -H 'Content-Type: application/json' -d '{\"msg_type\":\"text\",\"content\":{\"text\":\"Claude 任务已完成\"}}'"
          }
        ]
      }
    ]
  }
}
```

### 6. 会话开始时加载上下文

```
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "cat project-context.md"
          }
        ]
      }
    ]
  }
}
```

## 关键要点总结

### ✅ 配置检查清单

1. **JSON 格式正确**

   ```
   python3 -m json.tool .claude/settings.local.json
   ```
2. **命令可执行**

   ```
   # 测试命令是否可用
   /usr/bin/afplay /System/Library/Sounds/Glass.aiff
   ```
3. **使用绝对路径**

* ✅ `/usr/bin/afplay`
* ❌ `afplay`（可能因 PATH 问题失败）

4. **工作区已信任** ⭐️ 最关键

   ```
   # 检查信任状态
   cat ~/.claude.json | grep -A5 "$(pwd)"
   ```
5. **重启 Claude Code**

* 配置修改后必须重启才能生效

### ⚠️ 常见陷阱

1. **`/hooks` UI 不显示所有 Hook 类型**

* UI 只显示部分 Hook（PreToolUse, PostToolUse 等）
* Stop、SessionStart 等不会显示在 UI 中
* **不代表配置无效**！

2. **配置文件优先级**

* `~/.claude/settings.json` - 用户全局设置
* `.claude/settings.json` - 项目设置
* `.claude/settings.local.json` - 本地项目设置（优先级最高）

3. **工作区信任是必须的**

* 即使全局设置已信任，每个工作区也需要单独信任
* Hooks 需要 `hasTrustDialogHooksAccepted: true`

4. **Stop Hook 的触发条件**

* ✅ Claude 主动完成任务并停止
* ❌ 用户中断（Ctrl+C）不会触发
* ❌ 等待用户输入时不会触发

### 🐛 调试技巧

1. **查看 Debug 日志**

   ```
   claude --debug
   # 日志位置：~/.claude/debug/[session-id].txt
   ```
2. **添加日志到命令**

   ```
   {
     "command": "echo \"$(date)\" >> /tmp/hook.log && your-command"
   }
   ```
3. **简化命令测试**

* 先用最简单的命令测试（如 `echo test`）
* 确认能执行后再添加复杂逻辑

4. **使用 SessionStart 快速测试**

* SessionStart 在启动时立即触发
* 最容易验证 Hooks 是否工作

## 最佳实践

### 1. 安全第一

```
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo \"$TOOL_INPUT\" | grep -q 'rm -rf' && exit 2 || exit 0"
          }
        ]
      }
    ]
  }
}
```

### 2. 使用环境变量

```
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/scripts/format.sh"
          }
        ]
      }
    ]
  }
}
```

### 3. 设置超时

```
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "long-running-task.sh",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

### 4. 条件执行

```
#!/bin/bash
# ~/.claude/hooks/smart-notify.sh

# 只在工作时间通知
hour=$(date +%H)
if [ $hour -ge 9 ] && [ $hour -le 18 ]; then
    afplay /System/Library/Sounds/Glass.aiff
fi
```

## 资源链接

* Claude Code Hooks 官方文档
* Claude Code Hooks 快速指南
* macOS 系统声音列表

## 结语

Claude Code Hooks 是一个强大但容易被忽视的功能。虽然初次配置可能会遇到一些坑（尤其是工作区信任的问题），但一旦配置成功，它能极大提升你的开发效率。

希望这篇文章能帮助你快速配置 Hooks，避免我踩过的坑。如果你有其他有趣的 Hooks 使用场景，欢迎在评论区分享！