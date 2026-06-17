---
title: Claude Code hooks支持if条件，一天省下半小时
author: 知识药丸
date: 
url: https://mp.weixin.qq.com/s?__biz=MzIwMTM5MTM1NA==&mid=2649477557&idx=1&sn=d9438613c1612103cb25d16f45f64d88&chksm=8f218a12f32b8e5306d91c2c143d4d887194dbaa4df70975ca5e614cefc885cb04072be80586&mpshare=1&scene=24&srcid=0329MqVON7q53Fqqp4q1A1HR&sharer_shareinfo=170e9bc167abb05df1fa02bb7fb7892f&sharer_shareinfo_first=170e9bc167abb05df1fa02bb7fb7892f#rd
---

🌟星标 + 👆关注，第一时间知道最新、最有用的AI编程姿势

[《贾杰的AI编程秘籍》](https://mp.weixin.qq.com/s?__biz=MzIwMTM5MTM1NA==&mid=2649474437&idx=1&sn=4ae1516afc0ec71b7638dd79daaae2cb&scene=21#wechat_redirect)付费合集，共10篇，现已完结。30元交个朋友，学不到真东西找我退钱；）

 

以及[我开发的手机编程App，已经冲到了付费榜第一，欢迎围观](https://mp.weixin.qq.com/s?__biz=MzIwMTM5MTM1NA==&mid=2649477442&idx=1&sn=165edcddec85ffaff9db02890f9d5e55&scene=21#wechat_redirect)

---

### 写在前面

最近刷到 Jarred Sumner 分享的一个 Claude Code 新特性，挺有意思。

说的是 hooks 里现在支持 `"if"` 条件了。

听起来很简单？但解决的问题很实际。

### 痛点在哪

我们都知道，大项目里 hooks 是个好东西。

代码检查、类型校验、测试运行，该有的都得有。

但问题来了。

你只是想改个 README，结果触发了整套检查流程。Lint 跑一遍，测试跑一遍，类型检查再来一遍。

等了三分钟，才发现自己只是改了两行文档。

这种等待在长时间开发会话中会累积起来。一天下来，可能就浪费了半小时。

更糟的是，你明知道这些检查对当前操作没意义，却无能为力。

### 条件执行

现在 Claude Code 给了个解法：`"if"` 条件。

```
    {  
  "hooks": {  
    "pre-commit": {  
      "if": "git commit",  
      "run": "npm run lint"  
    }  
  }  
}
```

命令不匹配？跳过。

就这么简单。

Lint 只在真正 commit 时运行，其他时候安静待着。

### 为什么重要

这不是什么黑科技。

但它改变了一件事：hooks 从“全局开关”变成了“智能助手”。

以前你要么全开，要么全关。现在可以按需触发。

在大型仓库里，一次完整 hooks 可能要跑好几分钟。一天执行几十次命令，时间就这么没了。

更重要的是心理负担。你不用再想“我就改个配置，为什么要等这么久”。

工具该帮你，不该拖你后腿。

### 怎么用

根据场景设条件就行。

只在 push 时跑测试：

```
    {  
  "hooks": {  
    "pre-push": {  
      "if": "git push",  
      "run": "npm test"  
    }  
  }  
}
```

只在主分支跑严格检查：

```
    {  
  "hooks": {  
    "pre-commit": {  
      "if": "git branch --show-current | grep main",  
      "run": "npm run strict-check"  
    }  
  }  
}
```

灵活度上来了。

### 总结

小功能，真痛点。

Hooks 不再是“一刀切”，而是按需工作。

对 hooks 配置复杂的大项目来说，能省不少时间。

P.S. 功能已经可用了，有类似痛点的可以试试。

---

 

 

坚持创作不易，求个一键三连，谢谢你～❤️

以及「AI Coding技术交流群」，联系 ayqywx 我拉你进群，共同交流学习～

订阅链接 https://note.mowen.cn/detail/OLPEp7HzeB0EXJOLe7mM4