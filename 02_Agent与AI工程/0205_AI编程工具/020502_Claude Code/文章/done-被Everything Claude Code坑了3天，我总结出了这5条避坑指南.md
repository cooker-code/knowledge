> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: 被Everything Claude Code坑了3天，我总结出了这5条避坑指南
author: 汇数智通AI
date:
url: https://mp.weixin.qq.com/s?__biz=Mzk5MDQyODA5MA==&mid=2247483735&idx=1&sn=27f419fce45b875bd7f0415479f69f79&chksm=c4ce42a3a403f5311127faf950751e40134cd6593dc6ad0758cb6aa5bcab896e99cab8b044ef&mpshare=1&scene=24&srcid=0414xhLAIQJSth8OcXfiNrnI&sharer_shareinfo=2c13ad8968319713542f4658393f3d1d&sharer_shareinfo_first=2c13ad8968319713542f4658393f3d1d#rd
---

点击蓝字

关注我们

# 被Everything Claude Code坑了3天，我总结出了这5条避坑指南

上周末刷GitHub的时候，看到一个项目叫Everything Claude Code，8万多star，号称能把Claude Code的编码效率提升300%。作为一个天天跟AI编程助手打交道的程序员，我心想这不就是给我量身定制的吗？

结果，从周六早上9点开始折腾，到周一晚上11点才算真正跑通。中间踩的坑，说出来都是泪。这篇文章不聊那些虚的，就说说这3天我是怎么被坑的，以及怎么避免这些坑。

---

## 坑1：装了插件，rules没生效

**踩坑经过：**

按照README的说法，安装很简单：

```
/plugin marketplace add affaan-m/everything-claude-code
/plugin install everything-claude-code@everything-claude-code
```

装完之后，我兴冲冲地试了试 `/plan "添加用户认证"`，结果Claude一脸懵，完全不知道我在说啥。我心想不对啊，43个skills、31个commands不是应该自动生效吗？

翻了半天issues才发现，原来有个**超级重要的步骤被藏在了文档中间**：

> ⚠️ **重要提示：** Claude Code 插件无法自动分发 `rules`，需要手动安装

也就是说，我只做了一半，剩下的一半没做。

**解决方案：**

```
# 第一步：克隆仓库
git clone https://github.com/affaan-m/everything-claude-code.git

# 第二步：复制rules（这个步骤超级容易漏！）
cp -r everything-claude-code/rules/common/* ~/.claude/rules/
cp -r everything-claude-code/rules/typescript/* ~/.claude/rules/  # 根据你的技术栈选

# 第三步：再安装插件
/plugin install everything-claude-code@everything-claude-code
```

**教训：** 看到⚠️符号一定要停下来仔细看，别像我似的扫一眼就跳过。

---

## 坑2：Memory不同步，每次会话都要重新教Claude

**踩坑经过：**

好不容易把rules装好了，我试着让Claude帮我写一个Python脚本。写了一半我说"先这样吧，明天继续"。

第二天打开Claude Code，我说"接着昨天那个脚本"，结果它问我："什么脚本？"

我：？？？

Memory系统不是说好的"自动跨会话保存/加载上下文"吗？怎么跟鱼的记忆似的？

**排查过程：**

查了一下午，发现是SessionStart钩子没跑起来。用debug模式看了日志：

```
claude --debug
```

发现报错：`$CLAUDE_ENV_FILE` 未定义。原来环境变量没设置对。

**解决方案：**

在你的shell配置文件里加上：

```
# ~/.bashrc 或 ~/.zshrc
export CLAUDE_ENV_FILE="$HOME/.claude/env"
```

然后创建目录并给权限：

```
mkdir -p ~/.claude/env
chmod 755 ~/.claude/hooks/*.sh  # 给钩子脚本执行权限
```

最后重启Claude Code：

```
exit  # 退出当前会话
claude  # 重新进入
```

**教训：** 别信"开箱即用"这种鬼话，该配的环境变量一个都不能少。

---

## 坑3：Skills目录结构不对，Claude识别不了

**踩坑经过：**

ECC的一大卖点是可以自定义skills。我看了官方示例，想着自己也写一个skill来管理我们团队的代码规范。

我创建了个目录：

```
mkdir my-skill
touch my-skill/SKILL.md
```

写了几百字的说明，结果Claude完全没反应，好像我的skill不存在一样。

**问题所在：**

后来去翻了官方文档，发现这么一句话：

> "A common mistake in skill documentation is failing to reference additional resources."

原来我踩的是"常见错误第4号"。不只是要有SKILL.md，还得有完整的目录结构和前置元数据。

**解决方案：**

正确的skill目录结构应该是这样：

```
my-skill/
├── SKILL.md              # 核心文档，必须有YAML前置元数据
├── references/           # 参考资料目录
│   └── coding-standards.md
├── examples/             # 示例代码目录
│   └── good-example.py
└── scripts/              # 自动化脚本目录
    └── auto-check.sh
```

SKILL.md的开头必须是：

```
---
name: My Team Standards
description: 我们团队的代码规范
version: 1.0.0
---

## 参考资料
- [编码规范](./references/coding-standards.md)
```

**教训：** 看文档要看完整，别只看quick start那一节。

---

## 坑4：Hooks冲突，调试到怀疑人生

**踩坑经过：**

ECC自带的hooks挺多的，SessionStart、PreToolUse、Stop什么的。我自己也写了个PreToolUse钩子，想在写入文件前加个确认提示。

结果一跑起来，Claude直接卡死，不动了。按Ctrl+C都没反应，只能kill掉进程。

查了半天，发现是我写的钩子和ECC自带的钩子冲突了。两个PreToolUse钩子，一个想继续，一个想拦截，结果形成了死锁。

**调试方法：**

调试hooks是真的痛苦，官方给了个流程：

```
# 1. 先用linter检查脚本
./hook-linter.sh my-plugin/scripts/my-hook.sh

# 2. 创建测试输入
./test-hook.sh --create-sample PreToolUse > test-input.json

# 3. 测试执行并看详细输出
./test-hook.sh -v my-plugin/scripts/my-hook.sh test-input.json

# 4. 验证JSON schema
./validate-hook-schema.sh my-plugin/hooks/hooks.json
```

**解决方案：**

如果你要自定义hooks，最好先禁用ECC的对应钩子，或者合并成一个：

```
{
  "PreToolUse": [
    {
      "matcher": "Write|Edit",
      "hooks": [
        {
          "type": "prompt",
          "prompt": "先执行ECC的安全检查，再执行我的自定义逻辑"
        }
      ]
    }
  ]
}
```

或者用命令类型的钩子，直接调用bash脚本，在里面统一处理：

```
{
  "type": "command",
  "command": "bash ${CLAUDE_PLUGIN_ROOT}/scripts/combined-hook.sh",
  "timeout": 10
}
```

**教训：** 钩子这玩意儿，能不用就不用，非用不可的话一定要做好隔离。

---

## 坑5：包管理器检测抽风，yarn和pnpm来回跳

**踩坑经过：**

我们的项目用的是pnpm，但ECC有时候检测出来是yarn，有时候是npm，反正就是不对。

我查了一下ECC的检测优先级：

1. 1. 环境变量：`CLAUDE_PACKAGE_MANAGER`
2. 2. 项目配置：`.claude/package-manager.json`
3. 3. package.json里的`packageManager`字段
4. 4. 锁文件检测
5. 5. 全局配置

按理说我在package.json里写了`"packageManager": "pnpm@8.15.0"`，应该能检测对吧？但并没有，还是时不时就变回yarn。

**解决方案：**

最稳妥的方法是直接设置环境变量：

```
# ~/.bashrc 或 ~/.zshrc
export CLAUDE_PACKAGE_MANAGER=pnpm
```

或者在项目根目录创建配置文件：

```
# 使用ECC提供的脚本
node scripts/setup-package-manager.js --project pnpm

# 或者手动创建 .claude/package-manager.json
{
  "packageManager": "pnpm"
}
```

检测当前设置：

```
node scripts/setup-package-manager.js --detect
```

**教训：** 自动检测这种东西，看着很智能，实际用起来就是玄学。能手动指定的就别依赖自动检测。

---

## 总结：到底值不值得折腾？

被坑了3天，我得说句公道话：**如果你已经用顺了原生的Claude Code，其实没必要硬上ECC。**

ECC适合什么场景？

* • 大型项目，需要团队协作和代码规范统一
* • 复杂任务，需要多agents协作（planner + architect + reviewer）
* • 对AI编码有高要求，愿意花时间调优

ECC不适合什么场景？

* • 个人小项目，需求简单直接
* • 追求开箱即用，不想折腾配置
* • 偶尔用用Claude Code，不是重度用户

我现在的用法是**部分采用**：保留了ECC的rules和常用的几个commands（/plan、/code-review），但关掉了大部分hooks和自动memory功能。这样既享受了ECC的好处，又避免了被它绑架。

如果你也想试试，建议先从最简单的开始：装rules、试几个commands，觉得顺手了再慢慢加其他功能。千万别一股脑全开，不然就会像我一样，花3天时间在踩坑上。

你用过Everything Claude Code吗？踩过哪些坑？欢迎在评论区分享，大家一起避雷。

---

**参考链接：**

* • Everything Claude Code GitHub: https://github.com/affaan-m/everything-claude-code
* • 官方精简指南: https://x.com/affaanmustafa/status/2012378465664745795
* • 官方详细指南: https://x.com/affaanmustafa/status/2014040193557471352

### 往期回顾

* • [我实测了170个Claude Code技能插件，这10个最值得装](https://mp.weixin.qq.com/s?__biz=Mzk5MDQyODA5MA==&mid=2247483690&idx=1&sn=8f4d2e8d51c8957f1d7887d70c5ff979&scene=21#wechat_redirect)
* • [200行代码，从零搭建你的Claude Code克隆版](https://mp.weixin.qq.com/s?__biz=Mzk5MDQyODA5MA==&mid=2247483685&idx=1&sn=cb132fa95b4f2490886fa8d8428e0fc7&scene=21#wechat_redirect)
* • [我花3小时部署了OpenClaw，现在5个平台共用一个AI大脑](https://mp.weixin.qq.com/s?__biz=Mzk5MDQyODA5MA==&mid=2247483675&idx=1&sn=82d6566ac37e05cc3ee81fa94e912e86&scene=21#wechat_redirect)
* • [算力越便宜，软件死越快：揭秘AI产业链的“自我吞噬”死循环](https://mp.weixin.qq.com/s?__biz=Mzk5MDQyODA5MA==&mid=2247483660&idx=1&sn=c371c26783804ef13b2c09a5a47598c2&scene=21#wechat_redirect)
* • [OpenBB技术深潜：AI智能体如何重塑金融数据分析范式？](https://mp.weixin.qq.com/s?__biz=Mzk5MDQyODA5MA==&mid=2247483655&idx=1&sn=b9767fd57a86ae43f826f9b9456fdb53&scene=21#wechat_redirect)