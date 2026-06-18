---
title: 让Claude Code帮你精准提交代码：中文可读的 feat/fix/docs 规范一键生成
author: 刘小排r
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI1MTUxNzgxMA==&mid=2247499429&idx=1&sn=b24f05eed52ab30573228c210bcd9a6d&chksm=e8b7381bd64589a7769b15ea299720cc7990186860c7e5cc70ce215d7e1acfdb6057f0a13a94&mpshare=1&scene=24&srcid=1105FjSeRcOjw322JSf9zn6T&sharer_shareinfo=3e8f7f323b730286fca23e01cabbe1d7&sharer_shareinfo_first=3e8f7f323b730286fca23e01cabbe1d7#rd
---

哈喽，大家好，我是刘小排。

你希望看到自己的Git代码提交记录，是下图这样的？

还是下图这样的？

都喜欢后者，对吧？

好的Git提交记录，我认为至少有这些特点：

* 每条提交记录，都用人能看懂的话，说明需求改动内容
* 提交记录的开头，有分类标签。例如， feat = 新功能 (feature)、 fix = bug修复、pref = 性能优化、 docs = 文档 ……
* 每个功能，只有一条提交记录
* 每条提交记录，只对应一个功能

弄得这么规范，会不会很麻烦？

不会。

下面讲两个小技巧，你也可以在AI的帮助下快速做到，又快又规范的提交。

无论是VS Code、还是Cursor、或者其他由VS Code代码分叉打包的IDE产品，都可以使用。

第一步：精准选择待提交文件

无即便是我们并行开了很多个Claude Code/Codex、分别在编辑多个文件，我们也可以精准选择本次需要提交哪些内容。

方法是：在已更改文件文件区域(Changes)，找到具体文件，点击文件名后面的加号(+)， 可把修改记录放到暂存区(Staged Changes)

示例： 下图编号1的红圈。

如果点歪了，可以到Staged Changes区域找到文件，点文件后面的减号 ( - )， 重新把一个文件放回到Changes区域

第二步：点击AI自动生成代码提交信息

确保需要提交的文件在Staged Changes区域后，点击 VS Code / Cursor 提供的自动生成Git记录按钮。

示例：下图编号2的红圈。

由于在第一步，我们已经精准选定了所有当前需求相关的文件，此时生成的提交记录，会非常精确。

就这样。

---

这还嫌麻烦？？

还有一招全自动：  让Claude Code帮你提交。

（为了进一步偷懒，请打开Bypass permission，提前授予Claude Code所有权限）

> 帮我提交代码，只提交和本次修改内容有关的文件，提交记录写规范点，用中文

注意观察它的操过过程： Claude Code 会自动把本次功能所涉及的文件，放到Staged Changes区域。 （当然，它是通过命令行的形式，而不是点击按钮）

然后，再进行了代码提交。请看下图，Claude Code只提交了本次功能相关的部分、写了规范的提交记录； 把和本次修改无关的内容，继续留到了Changes区域。

---

学会了吗？期待你的反馈！