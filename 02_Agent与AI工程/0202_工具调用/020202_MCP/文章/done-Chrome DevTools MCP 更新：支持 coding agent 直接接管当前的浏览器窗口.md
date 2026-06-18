> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020202_MCP/020202_核心知识点/MCP生产接入与治理边界|MCP生产接入与治理边界]]
---
title: Chrome DevTools MCP 更新：支持 coding agent 直接接管当前的浏览器窗口
author: oh my x
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg5Mzk4MTc0Ng==&mid=2247494025&idx=1&sn=389d74ea49e90ca588a932555644b2c5&chksm=c1990b6541887b7b45a4df1963ee5c534705d1aa82ff30e1e474aa9085a44d6cbf0db7d9e75f&mpshare=1&scene=24&srcid=0316Sh5m25IVJcE0FnOu5SSy&sharer_shareinfo=1f76abcebd1f414aa9e1d38c454db586&sharer_shareinfo_first=1f76abcebd1f414aa9e1d38c454db586#rd
---

TLDR:

Chrome DevTools MCP 出了个“自动连接”功能，AI 现在能直接钻进你正开着的浏览器里修 Bug，不用再重复登录或者反复对齐上下文了。

刚刷到一个消息，Chrome DevTools MCP 更新了一个大家催了很久的功能：**自动连接活跃的浏览器会话**。

简单来说，就是你的 AI 编程助手现在可以“瞬移”到你正在用的浏览器窗口里，和你并肩作战了。这就很有意思了，咱们划几个重点看看它到底能省掉多少麻烦。

### 1. 告别尴尬的重复登录 🔑

以前让 AI 连浏览器，它往往是新开一个干干净净的窗口。如果你要调的是一个需要登录的页面（比如后台管理系统），你还得在 AI 开的那个窗口里重新扫码、输验证码，特别心累。

现在有了这个增强版功能，**它能直接复用你当前的会话**。你在浏览器里登录好了，AI 进来直接就能干活，那些挡在登录墙后面的 Bug 终于能让它直接去修了。

### 2. “你点哪，它修哪”的无缝衔接 🤖

这个是我觉得最爽的地方。以前你要让 AI 查个报错，得截图、复制日志、还得写一大堆描述告诉它在哪。

现在的逻辑变了：

* **网络面板：**

  你在 DevTools 里发现一个请求红了，直接选中它，然后告诉 AI：“去查查这个请求为啥挂了”。
* **元素面板：**

  你看到页面上某个组件样式歪了，选中那个 DOM 元素，AI 就能直接基于你选中的上下文开始分析。

这种从手动排查到 AI 接入的丝滑感，真的把“生产力”给拉满了。

### 3. 怎么用？🛠️

目前这招还在 **Chrome M144 (Beta版)** 里。想要尝鲜的话，操作逻辑大概是这样的：

* **开启权限：**

  去 `chrome://inspect/#remote-debugging` 把远程调试开关打开。官方为了安全，每次 AI 连进来时都会弹窗问你同不同意，而且浏览器顶部会有个“受自动化测试软件控制”的提示。
* **配置参数：**

  在 MCP 服务器的启动参数里加个 `--autoConnect` 就行。

我看了一下官方给出的 **gemini-cli** 配置示例，大概长这样：

```
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": [
        "chrome-devtools-mcp@latest",
        "--autoConnect",
        "--channel=beta"
      ]
    }
  }
}
```

说白了，Google 正在把 Chrome 变成 AI 的“眼睛”和“手”。目前还只是个开始，官方说后续会把更多面板的数据（比如 Application、Console 之类的）都喂给 AI。

大概就是这些，如果你平时经常跟 DevTools 打交道，这波更新绝对值得去瞅一眼。

原文链接：

https://developer.chrome.com/blog/chrome-devtools-mcp-debug-your-browser-session

---

---

🎉 **欢迎****加入我们的用户交流群** 🎉

在这里，你可以：
📢 第一时间了解最新动态和活动信息
🤝 结识同行，共享实战经验
📚 探讨行业趋势，拓展人脉圈子

📌 **如何加入？**📷 扫描添加小助手，完成验证即可入群！

期待你的加入，一起交流成长！🚀