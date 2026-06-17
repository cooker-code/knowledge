---
title: OpenSpec只支持英文？用这一招立马变成“全语言通吃”
author: AI喵小课
date: 
url: https://mp.weixin.qq.com/s?__biz=MzE5ODM3NjM4NQ==&mid=2247484467&idx=1&sn=80fe61ac8e6038ee2a81881dc4bbaaff&chksm=97d40087f3ab08065edef390f55bd7aaa52a6b4363e5cf1aa0aba68b1365c05ea3d65073e1f6&mpshare=1&scene=24&srcid=1206sb6i4iGupXgD3OWdUCo0&sharer_shareinfo=3bc000d8d9152f9888a57132178c48e9&sharer_shareinfo_first=3bc000d8d9152f9888a57132178c48e9#rd
---

上一篇文章《搭建AI开发流水线：Cursor + Qwen Code + OpenSpec》伙伴们普遍反映很实用，但对于OpenSpec只支持英文的问题很是头大，问我有没有解决办法。

> 说明：本文用到了Claude Code Cli工具，伙伴们如果不知道如何使用，可以参考我之前的教程：
>
> 命令行党狂喜：Claude Code才是最划算的AI编程工具

还别说，真有解决办法，但是得借助Claude Code的自定义命令能力，我们可以把关键文档安全、可控地翻成中文（或其它语言），真正做到团队友好上手。

原理：

在Claude Code里写一个受限的翻译斜杠命令：它只翻译指定OpenSpec 文档中的自然语言段落，保护代码块/路径/命令示例/frontmatter，不改动translate命令本身；翻译完成后额外把“Always respond in <语言>”之类的指令加进相关agent文件，保证后续交互也用目标语言。

```
用户输入命令 -> Claude 读取命令文件 -> 解析语言参数 -> 读取目标文件 -> 执行翻译 -> 写回文件 -> 更新配置
```

如何实现？

进入项目目录，先用OpenSpec初始化一下规范文档（如果没有初始化过），初始化的时候，在Natively supported providers列表中选择Claude Code：

初始化后，会在项目目录中找到.claude/commands/，这里就是Claude Code放置命令的地方：

```
.claude/└── commands/    ├── openspec/        # OpenSpec相关    │   ├── proposal.md    │   └── apply.md
```

我们的OpenSpec自定义命令就放在上面的目录下：

.claude/commands/openspec/translate.md，下面是创建自定义翻译命令的完整代码，我先贴上了，伙伴们可以直接拿去用：

```
---name: OpenSpec: Translatedescription: Translate OpenSpec documentation to any target languagecategory: OpenSpectags: [openspec, translation, i18n]---<!-- 这里写命令的详细执行指令 --><!-- OPENSPEC:START -->**Overview**This command translates OpenSpec-related documentation from English to a specified target language while preserving technical accuracy and document structure.**Usage**`/openspec:translate <language>`**Examples**- `/openspec:translate chinese` or `/openspec:translate 中文`- `/openspec:translate japanese` or `/openspec:translate 日本語`- `/openspec:translate spanish` or `/openspec:translate español`- `/openspec:translate korean` or `/openspec:translate 한국어`- `/openspec:translate french` or `/openspec:translate français`**Target Files**The following files will be translated:- `openspec/project.md`- `openspec/AGENTS.md`- `.claude/commands/openspec/proposal.md`- `.claude/commands/openspec/apply.md`- `.claude/commands/openspec/archive.md`- `AGENTS.md` (OpenSpec section only)- `CLAUDE.md` (OpenSpec section only)**Excluded Files**The following files will NOT be translated and always remain in English:- `.claude/commands/openspec/translate.md` (this command file itself)**Language Name Mapping**The system maps English language names to localized names for the language instruction:- `chinese` or `中文` → "中文"- `japanese` or `日本語` → "日本語"- `spanish` or `español` → "español"- `korean` or `한국어` → "한국어"- `french` or `français` → "français"**Process**1. Extract target language from command arguments2. Determine the localized language name for the language instruction3. Read each target file using the Read tool (skip `.claude/commands/openspec/translate.md`)4. Translate text content to the specified language while preserving:   - Frontmatter (YAML headers)   - Code blocks and command examples   - Markdown structure (headings, lists, links, tables)   - Special markers (e.g., `<!-- OPENSPEC:START -->`)   - English commands, file paths, and technical identifiers5. Write translated content back to original files using the Write tool6. After translation, append or update language instruction in `CLAUDE.md` and `AGENTS.md`:   - Check if file ends with "**Always respond in [language]**"   - If exists, replace with new target language   - If not exists, append new instruction: "Always respond in [localized language name]"7. Provide translation progress and completion feedback**Translation Principles**- Maintain technical terminology accuracy- Use natural, idiomatic expressions in the target language- Preserve all English commands, code, and file paths- Keep the original document's tone and style- Support both English language names (e.g., "chinese") and native names (e.g., "中文")**WARNING**- Do not translate the command file itself `.claude/commands/openspec/translate.md`<!-- OPENSPEC:END -->
```

命令创建好后，在控制台输入：claude启动的Claude Code，然后在命令框输入“/”就能看到我们刚创建的命令了：

这样就好办了，我们输入刚才创建的命令：

```
/openspec:translate 中文
```

这个时候，Claude就会将我们指定的OpenSpec文档翻译成中文，怎么样？挺方便的吧！

上面，我们针对translate讲解了如何用Claude对OpenSpec做命令扩展，至于更多的命令扩展，大伙可以参考Claude的斜杠命令文档：

> https://code.claude.com/docs/zh-cn/slash-commands

这里详细讲解了如何扩展Claude，在此我就不多做说明了！

好了，今天的内容就分享到这里了，希望能对伙伴们有帮助！如果还有更多疑惑，欢迎在评论区留言！

往期文章：

[搭建AI开发流水线：Cursor + Qwen Code + OpenSpec 免费撸代码](https://mp.weixin.qq.com/s?__biz=MzE5ODM3NjM4NQ==&mid=2247484458&idx=1&sn=df3423f76f2e443874b3c061d4d247e1&scene=21#wechat_redirect)

[Coding Agent必修课：用Mini-Kode搭建你的第一个可控AI工程师](https://mp.weixin.qq.com/s?__biz=MzE5ODM3NjM4NQ==&mid=2247484427&idx=1&sn=4265dfa990ee9e1ace81338a70a6a49d&scene=21#wechat_redirect)

[Qoder vs Trae vs CodeBuddy谁才是深得人心的国产AI编程助手？](https://mp.weixin.qq.com/s?__biz=MzE5ODM3NjM4NQ==&mid=2247484395&idx=1&sn=57a733c8382709d81cbdd590bf4f777b&scene=21#wechat_redirect)

[安装chrome-devtools-mcp：让AI看懂网页，自动帮你Debug](https://mp.weixin.qq.com/s?__biz=MzE5ODM3NjM4NQ==&mid=2247484354&idx=1&sn=ba28341b1781b2446a56d544ab848f0e&scene=21#wechat_redirect)

[四大AI编程神器测评：Qoder、Trae、CodeBuddy、Cursor哪款最适合你？](https://mp.weixin.qq.com/s?__biz=MzE5ODM3NjM4NQ==&mid=2247484154&idx=1&sn=60565fc81568a9a44be551831482787d&scene=21#wechat_redirect)