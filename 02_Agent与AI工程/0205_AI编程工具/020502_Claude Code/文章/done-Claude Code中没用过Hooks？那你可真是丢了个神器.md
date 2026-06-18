> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Claude Code中没用过Hooks？那你可真是丢了个神器
author: 老金带你玩AI
date:
url: https://mp.weixin.qq.com/s?__biz=MzI0NzU2MDgyNA==&mid=2247489918&idx=1&sn=bd5458b3e600d696a7381a28598ea1ec&chksm=e8b91ccce4b4f8033b2020f6fcc3a3b8d85020dc5f67f960b0a878b2af6463971af3dbb45eb3&mpshare=1&scene=24&srcid=1128pj5ei1Rps0VXqsXtpoHB&sharer_shareinfo=88db3a630501e3f5f569627d22eb7e6a&sharer_shareinfo_first=88db3a630501e3f5f569627d22eb7e6a#rd
---

加我进AI讨论学习群，公众号右下角“联系方式”

文末有老金的 **开源知识库地址·全免费**

---

用Claude Code Hooks快一年了,上周在群里随口问了句:

"你们用Hooks吗?"

20多个人,17个回:啥是Hooks?

剩下3个说听说过,不知道咋配。

我当时就懵了。

这就好比你买了辆车,天天当板车推,从来不知道能打火开。

我接着问:"那你们平时用Claude Code干啥?"

基本都是:让它改改代码,问问bug在哪儿。

我用Hooks用了大半年了,自动commit、自动格式化、保护敏感文件,这些都是日常操作。

周五凌晨刷GitHub,看到disler的claude-code-hooks-mastery,1.9k星。

点开一看,把8种Hook全实现了,代码写得比我自己的好太多。

这个项目值得推荐,比我自己摸索的那套完善多了。

---

## 我这大半年用的Hook

先说说我日常在用的几个,都是被坑出来的。

去年11月的坑:代码丢了3次

去年11月开始用Claude Code,有次让它重构一个300行的文件。

改了半天,突然断网,再打开,全没了。

那天改的逻辑,全都要重新想。

那时候就配了PostToolUse Hook,Claude动了文件就自动git commit。

大半年过来,再没丢过代码。

去年12月的坑:.env被删了

有次让Claude"清理一下项目里的无用文件"。

它把.env直接删了。

数据库密钥、API key,全没了。

找了半天备份才恢复。

从那以后配了PreToolUse Hook,碰到.env、.git这些文件,直接拦住。

今年开始的坑:格式化手动跑

以前写完代码,得手动npx prettier --write xxx。

一天下来得手动跑50次。

后来PostToolUse Hook配上,改完JS/TS文件,自动格式化。

这三个Hook,我每天都在用,已经成习惯了。

---

看了disler的claude-code-hooks-mastery,发现人家把8种Hook都实现了。

我自己这大半年,其实只用了3种:PostToolUse、PreToolUse、偶尔用用Stop。

其他5种,听说过但没深入用。

UserPromptSubmit是按回车时触发的,Notification是通知时用的,SubagentStop是子任务结束用的,PreCompact是压缩对话前触发,SessionStart是启动时加载。

disler项目里,这8种Hook都有完整实现,代码质量比我自己写的好太多。

特别是Stop Hook那块,他用了stop\_hook\_active标志防无限循环,我之前没想到这个细节,吃过亏。

还有个新东西我没玩过:Prompt-based Hooks。

不用写脚本,直接写提示词让LLM判断,这个思路挺骚。

所以这次专门写篇文章推荐,顺便把我自己这大半年踩的坑也说说。

关键是,这8个Hook里,有4个能"拦住"Claude,4个不能。

能拦住的是UserPromptSubmit(拦提示词)、PreToolUse(拦工具执行)、Stop(拦停止)、SubagentStop(拦子任务停止)。

不能拦住的是PostToolUse(事后检查,已经晚了)、Notification(纯提醒)、PreCompact(压缩前备份)、SessionStart(启动时加载)。

这个区别很重要。

比如你想防止Claude删.env,必须用PreToolUse,在它动手之前拦住。

PostToolUse不行,文件已经删了,来不及了。

---

## 最实用的3个配置

说了这么多理论,直接给配置。

### 配置1:自动git commit

这个救过我无数次。

在项目根目录创建.claude/settings.json:

```
{
  "hooks":{
    "PostToolUse":[{
      "matcher":"Edit|Write",
      "hooks":[{
        "type":"command",
        "command":"git add -A && git commit -m 'auto: claude changes'"
      }]
    }]
}
}
```

只要Claude改了文件,自动commit。

再也不怕代码丢。

### 配置2:保护敏感文件

这个防删库跑路。

```
{
  "hooks":{
    "PreToolUse":[{
      "matcher":"Edit|Write",
      "hooks":[{
        "type":"command",
        "command":"python3 -c \"import json,sys;d=json.load(sys.stdin);p=d.get('tool_input',{}).get('file_path','');sys.exit(2 if '.env' in p or '.git/' in p else 0)\""
      }]
    }]
}
}
```

这个python脚本的意思:

检查文件路径,如果是.env或.git,退出码返回2。

退出码2在Claude Code里是"阻止操作"的意思。

Claude想删.env,被Hook拦下来,删不了。

### 配置3:自动格式化

强迫症必备。

```
{
  "hooks":{
    "PostToolUse":[{
      "matcher":"Edit .*\\.(js|ts|jsx|tsx)$",
      "hooks":[{
        "type":"command",
        "command":"npx prettier --write $FILE_PATH"
      }]
    }]
}
}
```

改完JS/TS文件,自动prettier。

代码永远美美的。

把这3个配置合在一起:

```
{
  "hooks":{
    "PostToolUse":[
      {
        "matcher":"Edit|Write",
        "hooks":[{
          "type":"command",
          "command":"git add -A && git commit -m 'auto: claude changes'"
        }]
      },
      {
        "matcher":"Edit .*\\.(js|ts|jsx|tsx)$",
        "hooks":[{
          "type":"command",
          "command":"npx prettier --write $FILE_PATH"
        }]
      }
    ],
    "PreToolUse":[{
      "matcher":"Edit|Write",
      "hooks":[{
        "type":"command",
        "command":"python3 -c \"import json,sys;d=json.load(sys.stdin);p=d.get('tool_input',{}).get('file_path','');sys.exit(2 if '.env' in p or '.git/' in p else 0)\""
      }]
    }]
}
}
```

复制到.claude/settings.json,重启Claude Code,就生效了。

如果对你有帮助，记得关注一波~

---

## Exit Code这个东西

这里有个关键点,Exit Code。

你的Hook脚本退出的时候,返回的数字:0是成功继续,2是阻止操作把错误信息喂给Claude,其他数字是报错但继续。

所以保护文件那个脚本,用的sys.exit(2)。

如果检测到.env,退出码2,Claude就停下来,不会继续删文件。

这个机制,PreToolUse、UserPromptSubmit、Stop、SubagentStop都支持。

PostToolUse不支持,因为工具已经执行完了。

---

## 进阶玩法:Stop Hook强制继续

有时候Claude干完活想停,但你觉得还没完成。

比如跑测试,有3个失败了,Claude说"我完成了"。

你想让它继续修到全部通过。

用Stop Hook:

```
{
  "hooks":{
    "Stop":[{
      "hooks":[{
        "type":"command",
        "command":"python ~/.claude/hooks/check_tests.py"
      }]
    }]
}
}
```

check\_tests.py:

```
import json, sys

# 检查测试是否全部通过
# 如果有失败,返回exit code 2

data = json.load(sys.stdin)

# 检查stop_hook_active,防止无限循环
if data.get("stop_hook_active"):
    sys.exit(0)  # 已经在继续了,让它停

# 这里写你的测试检查逻辑
# 如果测试失败:
print("还有3个测试失败,继续修!", file=sys.stderr)
sys.exit(2)  # 阻止停止
```

注意那个`stop\_hook\_active`检查,很重要。 不然会无限循环,Claude永远停不下来。

## 高级控制:JSON输出

除了Exit Code,Hook还能返回JSON控制。 比如PreToolUse可以返回:

```
{
  "decision": "approve",
  "reason": "这个文件可以改"
}
```

或者:

```
{
  "decision": "block",
  "reason": "不许碰.env文件"
}
```

decision有3种:"approve"是批准绕过权限检查,"block"是阻止把reason喂给Claude,undefined是正常流程。

Stop Hook也能用:

```
{
  "decision": "block",
  "reason": "测试还没全通过,继续修"
}
```

这个JSON输出,比Exit Code更灵活,可以给Claude更详细的反馈。

---

## 新特性:Prompt-based Hooks

从最近版本开始,Stop和SubagentStop支持一种新模式。

不用写脚本,直接写个提示词,让LLM判断:

```
{
  "hooks":{
    "Stop":[{
      "hooks":[{
        "type":"prompt",
        "prompt":"Check if all tests passed. If any failed, block and tell Claude to fix them."
      }]
    }]
}
}
```

Claude Code会把Hook的输入和你的提示词,发给Haiku(快速LLM)。

Haiku返回决策,告诉Claude Code该咋办。

这个比写脚本灵活多了,但要消耗API。

---

## 真实数据

用了一个月,统计了一下:

自动commit:

以前每天手动commit 15-20次。

现在自动,省30分钟。

自动格式化:

以前手动prettier一天50次。

现在自动,省8分钟。

误删事故:

以前误删过3次敏感文件。

配了保护Hook,0次。

Anthropic官方研究说,Hooks能让开发效率提升50-100%。

TELUS给57000员工部署后,PR周转快了30%。

IG Group的分析团队,每周省70小时。

---

## 也有坑

Hook写错了,会影响Claude的正常工作。

比如你的Hook超时(默认60秒),会拖慢Claude。

调试也不方便,Hook报错信息有时候看不到。

建议先在测试项目试,Hook里加日志方便调试,执行时间控制在1秒内。

我的做法是在Hook里加这一行:

```
echo "$(date) Hook执行: $TOOL_NAME" >> ~/.claude/hook-debug.log
```

出问题了,去这个日志文件里找。

---

## 现在该你了

如果你经常让Claude改代码,手动commit太麻烦,担心Claude误删重要文件,代码格式化要手动跑,那你应该用Hook。

如果你只是偶尔问问题,不让Claude动文件,项目没有git,那Hook对你用处不大。

GitHub上那个教程,1.9k星了:

https://github.com/disler/claude-code-hooks-mastery

里面有完整的8种Hook实现,代码都是现成的。

我给的3个配置,复制到.claude/settings.json,重启就能用。

用完你会发现,Claude Code不只是个聊天工具。

配上Hook,它是个可以深度定制的自动化平台。

以前我们和Claude对话,是一问一答。

现在配上Hook,它改代码、自动检查、自动提交,一气呵成。

这才是Claude Code的正确打开方式。

---

**往期推荐：**

[提示词工工程（Prompt Engineering）](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI0NzU2MDgyNA==&action=getalbum&album_id=4120385726238392327#wechat_redirect)

[LLMOPS(大语言模运维平台)](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI0NzU2MDgyNA==&action=getalbum&album_id=3171759118513111043#wechat_redirect)

[WX机器人教程列表](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI0NzU2MDgyNA==&action=getalbum&album_id=3502843007181520907#wechat_redirect)

[AI绘画教程列表](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI0NzU2MDgyNA==&action=getalbum&album_id=3192433076551843848#wechat_redirect)

[AI编程教程列表](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI0NzU2MDgyNA==&action=getalbum&album_id=3704202865347362819#wechat_redirect)

    

---

谢谢你读我的文章。

如果觉得不错，随手点个赞、在看、转发三连吧🙂

如果想第一时间收到推送，也可以给我个星标⭐～谢谢你看我的文章。

开源知识库地址：

https://tffyvtlai4.feishu.cn/wiki/OhQ8wqntFihcI1kWVDlcNdpznFf

扫码**添加下方微信（备注AI）**，拉你加入**AI学习交流群**。