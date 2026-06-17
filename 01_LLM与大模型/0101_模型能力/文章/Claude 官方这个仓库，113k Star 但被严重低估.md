---
title: Claude 官方这个仓库，113k Star 但被严重低估
author: Thea的AI折腾日记
date: 
url: https://mp.weixin.qq.com/s?__biz=MzYyMzAxNjEwMA==&mid=2247484072&idx=1&sn=e1bf6af72a13cc4b67e93b7263c0e100&chksm=fee3798a3d23408ef7e4592b900a818738639cc0f741e95b6ede806c424c7ef880144928eee1&mpshare=1&scene=24&srcid=0416TjyJAquQRylf9dSxE5lr&sharer_shareinfo=0e48feefc74842350951e9576bd848af&sharer_shareinfo_first=0e48feefc74842350951e9576bd848af#rd
---

嗨，我是**Thea**，打过ACM-ICPC世界总决赛，做过微软搜索算法工程师，去年手搓了月入万刀的AI应用，在这里记录我折腾的折腾我的AI✨

最近在翻 GitHub，发现一个项目看起来很普通，但越看越觉得捡到宝了。

anthropics/skills。113k star，Anthropic 官方维护。

大多数人 star 完就关掉了。以前以为这就是 Anthropic 放出来的几个 prompt 模板——原来里面放的是他们内部怎么让 Claude 做对事的答案。

---

## Skills 到底是什么

一个 Skill，就是一个文件夹，核心是一个 `SKILL.md`。

里面不只是"你是一个 XX 助手"这种笼统定义，而是：这类任务的决策树、该用什么工具、踩过什么坑、什么情况下不能这么做——是完整的作战手册。

Claude Code 里一行命令装上：

`/plugin install example-skills@anthropic-agent-skills`

仓库里 17 个 Skill，说说哪几个让我有点意外。

---

## mcp-builder：原来 MCP 做坏了比没有更糟

很多人现在在搞 MCP，让 Claude 接外部 API、读数据库、控制工具。

问题是：怎么设计工具接口？

大多数人的做法是：把 API 端点直接包一层，扔给 Claude 用。

然后发现 Claude 用起来很奇怪——要么反复调错工具，要么在奇怪的地方卡死。

**这个 Skill 给出了 Anthropic 内部的真实答案：MCP 的质量，由 Claude 能不能顺畅完成真实任务来衡量，而不是"有没有包到所有端点"。**

具体说：工具命名要一致且可发现（比如统一用 `github_create_issue`、`github_list_repos`，而不是 `create`、`list`）；错误信息要告诉 Claude 下一步怎么做，而不是只报个 400；不确定要覆盖 API 还是设计高层 workflow 工具时——优先覆盖 API，让 Claude 自己组合。

以前以为 MCP 只是个技术活，原来核心是站在 Claude 的视角想问题。

### frontend-design：你一直在用最烂的那个 prompt

让 Claude 做前端，大多数人说的是："帮我做一个 XX 页面，要好看一点。"

然后出来的东西：Inter 字体，浅紫色渐变，白底卡片，圆角按钮。

每次都一样。你懂那种感觉。

这个 Skill 里有一条我觉得很诚实的规定——

NEVER use Inter, Roboto, Arial. NEVER use purple gradients on white backgrounds.

它把 AI 惯用的滥调列成黑名单，然后强制 Claude 在动一行代码之前，先回答一个问题：**"这个界面让人记住的那一件事是什么？"**

不是"现代感"，不是"简洁"，是要选一个极端——brutally minimal、retro-futuristic、maximalist chaos……选定了，才能写代码。

**原来不是 Claude 没审美，是一直没被逼着做选择。**

Thea 试了几次，出来的东西开始有点个性了。做 side project 落地页、做工具界面，这个 Skill 直接装上。

### skill-creator：这个有点元，但很好玩

skill-creator 是专门帮你写新 Skill 的 Skill。

你有没有这种经历：某件事 Claude 第一次做得很好，第二次换了个 session 又开始犯傻。因为上次那套 prompt 没有沉淀下来，每次都要重新喂给它。

skill-creator 的做法是：

1.把你的工作流写成草稿 Skill

2.生成一批测试 prompt 跑一遍

3.用内置的 eval 脚本量化结果

4.根据数据修改，循环

还有一个细节——它有个独立的 `skill description improver`，专门优化 Skill 的 description 字段，让 Claude 在对的时机触发对的 Skill，而不是触发了错误的。

**以前以为写 prompt 靠感觉，原来可以 A/B 测试。**

### webapp-testing：Claude 终于不再假装测试了

以前让 Claude 测前端，它会做什么？

它会"推理"：按照代码逻辑，点击这个按钮应该会触发 XX 函数，所以应该显示 XX……

全是应该，全是假设。

这个 Skill 用 Playwright，强制 Claude 实际跑脚本——真正截图、真正读 console log、真正验证 UI 行为。Skill 里甚至明确写了：**先跑 `--help` 看工具参数，不要自己猜。**

就这一条约束，让输出可信了很多。做全栈项目的人，前端测试一直是个坑，这个 Skill 值得看看。

---

## 最后说一件更重要的事

比这几个 Skill 本身更值得学的，是搞懂这套设计模式。

一个 `SKILL.md` 能放什么：决策树、工具调用规范、禁止行为清单、评估标准……

**Claude 不缺能力，缺的是被正确约束的场合。** Skills 就是给它造这种场合的方式。

Anthropic 把自己内部用的那套放出来了，认真翻一遍，比读 100 篇"如何用好 Claude"的文章都值。

仓库：github.com/anthropics/skills