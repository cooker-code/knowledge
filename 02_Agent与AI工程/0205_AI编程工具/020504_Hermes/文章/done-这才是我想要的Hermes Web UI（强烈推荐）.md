> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020504_Hermes/020504_核心知识点/Hermes协作与记忆治理边界|Hermes协作与记忆治理边界]]
---
title: 这才是我想要的Hermes Web UI（强烈推荐）
author: 云起泊言
date:
url: https://mp.weixin.qq.com/s?__biz=MzA5NjAxMTY1OA==&mid=2461868645&idx=1&sn=857b458123fe6bc2945862a0f61fe146&chksm=8624a5cca28f2d0e626a2259f1029001ad2ff6fb9aef9de848994891113af2905a9c99bb9af0&mpshare=1&scene=24&srcid=04174zpWZpDHvnuzCS7NKlym&sharer_shareinfo=9180bab6b199ac3a03343695d0fd6b52&sharer_shareinfo_first=9180bab6b199ac3a03343695d0fd6b52#rd
---

前几天，Hermes 官方出了 Web UI，我第一时间也体验了，之前也出了安装教程，虽然是官方出的，但是体验还是不太好，无法进行对话，而且真的欣赏不来老外的审美，页面丑字也丑，然后就没咋用了

详细可以看我这篇文章：

[Hermes Agent升级！网页端终于来了，附其他升级亮点](https://mp.weixin.qq.com/s?__biz=MzA5NjAxMTY1OA==&mid=2461868596&idx=1&sn=311a2ad056ca12cf0953c8285ce96b90&scene=21#wechat_redirect)

至于其他第三方的 Hermes Web UI 也体验了几个，比如 open webui，但是这类它只能对话，无法对 Hermes 的配置进行修改，如果单纯只是对话的话，用其他消息网关都能代替

这不，就在我反复折腾的时候，看到了一款国内大佬开发的Hermes Web UI，体验完我真的吹爆啊！

---

我先不吊大家胃口了，先上安装教程再给大家介绍

安装步骤很简单，就两条命令：

```
npm install -g hermes-web-ui 
hermes-web-ui start
```

执行完直接在浏览器访问：

```
http://127.0.0.1:8648/
```

如果你的设备没有安装nodejs，那么使用下方命令安装（支持Debian/Ubuntu/MacOS）：

```
bash <(curl -fsSL https://cdn.jsdelivr.net/gh/EKKOLearnAI/hermes-web-ui@main/scripts/setup.sh)
```

WSL:

```
bash <(curl -fsSL https://cdn.jsdelivr.net/gh/EKKOLearnAI/hermes-web-ui@main/scripts/setup.sh)
hermes-web-ui start
```

---

接下来给大家简单介绍一下为什么我会选择这个Web UI

#### UI 界面：

是不是感觉很好看，**并且支持对话**：

#### 手动创建定时任务：

#### 手动设置模型：

如果是要添加Oauth验证的，还是得靠Hermes CLI或者官方 Web UI

#### 频道设置：

并且支持微信扫码登录

#### 技能：

可以看到所有Hermes自带跟创建的skills，乍一看我的都已经上百个skills了

#### 记忆：

对`SOUL.md`、`MEMORY.md`、`USER.md` 直接进行修改

#### 用量：

可以直观的看到Token消耗统计

并且已经支持多Agent切换

看完介绍是不是觉得这套Web UI挺全能的，既支持对话，又支持微信扫码登录（官方的都不支持），还支持多Agent切换，真的是把能做到的功能都集成进来了

而不是那种：只能当“聊天壳子”的 UI

项目地址：

```
https://github.com/EKKOLearnAI/hermes-web-ui/
```

开源不易，觉得好用的，可以去点个 ⭐ 支持一下作者

---

## ✍️ 最后说一句

如果你之前对 Hermes 的印象是：

👉 只能在终端里折腾
👉 或者 UI 不太能打

那这个项目基本可以帮你改观：

👉 **Hermes 其实可以很好用，而且很好看**

---

**- 往期推荐 -**

**[Hermes 多 Agent 团队协作的正确打开方式](https://mp.weixin.qq.com/s?__biz=MzA5NjAxMTY1OA==&mid=2461868631&idx=1&sn=9b4210e367d4e69335bdb29839fe901d&scene=21#wechat_redirect)**

**[给大家推荐一个超级有趣的项目：TokenArena，Token消耗竞技场！](https://mp.weixin.qq.com/s?__biz=MzA5NjAxMTY1OA==&mid=2461868623&idx=1&sn=0ab6037761d33e60126287b65f0d6b2e&scene=21#wechat_redirect)**

**[飞牛NAS开启 Hermes 网页端（WebUI）详细教程](https://mp.weixin.qq.com/s?__biz=MzA5NjAxMTY1OA==&mid=2461868606&idx=1&sn=86d9c0470efee2ad54ad3fef0dd8e7a1&scene=21#wechat_redirect)**

**[飞牛NAS Docker方式部署Hermes Agent详细教程](https://mp.weixin.qq.com/s?__biz=MzA5NjAxMTY1OA==&mid=2461868596&idx=2&sn=04a4342ef540b22f4803b95bf7ed2f13&scene=21#wechat_redirect)**

**[飞牛NAS部署Hermes Agent教程](https://mp.weixin.qq.com/s?__biz=MzA5NjAxMTY1OA==&mid=2461868568&idx=1&sn=6f95a3ef52c29e8b3d4f3aaef9ef4140&scene=21#wechat_redirect)**