---
title: 再见Cursor，玩转Claude Code 的50个实用小技巧，效率拉满！
author: 泽安的AI编程
date: 
url: https://mp.weixin.qq.com/s?__biz=MzYyMzU2MDk4MA==&mid=2247483810&idx=1&sn=cb9054381a961dcbdae15fe0e08b1831&chksm=fe47b4b6389ba61f4015b76bd7139e0722947f23c4479b7244ab3a10b1b7b2e7baed74ddf181&mpshare=1&scene=24&srcid=1028yZl1ypCJM3wxyQOMGe9v&sharer_shareinfo=f79772f31c25c086bff366b84f7c7c95&sharer_shareinfo_first=f79772f31c25c086bff366b84f7c7c95#rd
---

大家好，我是泽安！见字如面～

这是泽安的第五篇文章，关注泽安，泽安给你带来 Claude Code 的保姆级的实践教程

前三篇干货，手把手带你装好了 Claude Code！不管是国内还是国外都可以用，👇点下方链接，一键直达！

[用了 Claude Code 之后，我不再续费 Cursor 了！国内使用 Claude Code 教程【附案例实操】](https://mp.weixin.qq.com/s?__biz=MzYyMzU2MDk4MA==&mid=2247483716&idx=1&sn=8dd7b825a79111c7366f32c513751a16&scene=21#wechat_redirect)

[Claude Code + GLM-4.5，最强性价比编程组合保姆级教程](https://mp.weixin.qq.com/s?__biz=MzYyMzU2MDk4MA==&mid=2247483743&idx=1&sn=9f59da29290d0e84b548bd7916cbb80e&scene=21#wechat_redirect)

[太全了！Claude Code 一键接入国产四大模型: DeepSeek、GLM、Qwen、Kimi 超全攻略！](https://mp.weixin.qq.com/s?__biz=MzYyMzU2MDk4MA==&mid=2247483769&idx=1&sn=2367d4f81bc75cfeaa5e813b26468b51&scene=21#wechat_redirect)

工具备齐了，实战却懵了圈：快捷键记不住、指令敲不对、需求理不清、上下文喂不好……一顿输出猛如虎，生成的代码一看，哎——像极了“二百五”。

先别急着甩锅给 AI！说到底，不是 AI 不好用，是你缺了一份「使用说明书」。

别慌！我决定专门开一个系列连载，就用 5 篇文章，彻底给你讲明白，说清楚！

关注我，别错过任何一篇。很快让你的 Claude Code 将不再是你骂骂咧咧关掉的“黑窗口”，而是真正成为你触手可及、极速开发的——10 倍效能的编程外挂！

# 一。 检查 Claude Code 的安装情况

我们想要详细查看 Claude Code 的安装信息，直接使用“doctor” 即可

```
doctor
```

# 二。 Claude 更新

先说一个小技巧，Claude 第一次安装的时候需要 npm install 来，但是后面在需要更新的时候不需要每一次都用 npm 来做，直接输入"claude update"即可更新到最新版本啦

```
claude update
```

# 三。 建别名

我们如果每次都输入“claude --dangerously-skip-permissions” 这么长一大串，说实话也挺麻烦的，我们直接给他起一个别名；

windows:

mac:

# 四。 使用免授权的模式登录 Cluade Code

Claude 登录命令直接输入 Cluade 就好了，但是使用 Claude Code 最烦人的地方：它对每件事都要请求权限。

有一个解决方案，启动 Claude Code 的时候，使用

```
claude --dangerously-skip-permissions
```

这里有一个选择，我们直接接受就好了

接受之后，右下角会出现黄色的“Bypassing Permissions”

# 五。 初始化

当 Claude Code 打开一个现有项目的时候，我们先让他读取该项目的代码。输入 /init 命令

分析完成后，会生成一个 Claude.md

后面我们在继续提出需求和修复 bug，这样 claude code 的开发效果更佳！

# 六。 查看账户和系统状态

知其然必然要知其所以然，你用的哪家的 API，你用的什么模型，你的工作目录在哪里，当你开发之前，可以只使用 status 命令来检查下

```
status
```

# 七。 清除上下文历史会话

使用 /clear 命令可清空当前对话历史，释放上下文空间，但此操作无法恢复已清除内容

# 八。 指定某一个文件进行修改

我们知道要修改某一个文件的时候，直接指定文件操作，使用 @ 符号就可以指定文件；同时也是可以支持@多个文件的

# 九。 花费总结

使用 cost 可以查看消费情况，可以查看总花费、总使用时长等等信息。

# 十。 监控使用量

ccusage 比 cost 的监控更精准，更细致，如下：

更多用法：

```
基础用法  
ccusage          # 显示每日报告（默认）    
ccusage daily    # 每日 token 使用量及费用    
ccusage monthly  # 月度汇总报告    
ccusage session  # 按会话统计用量    
  
# 实时监控    
ccusage blocks --live  # 实时用量仪表盘    
  
# 筛选与选项    
ccusage daily --since 开始时间【20250710】 --until 结束时间【20250718】  # 指定日期范围    
ccusage daily --json      # 输出 JSON 格式    
ccusage daily --breakdown # 按模型细分费用
```

基础命令有很多，夯实基础，没有坏处！抓紧收藏，偷着去练习下吧！

我新建了一个 AI 编程商业化群，不仅可以围观泽安复利商业化路径，而且还有很多干货分享，限免进群

>/ 作者：泽安

能看到这里的都是真爱！

如果觉得不错，随手点个赞、小心心、转发，评论四连吧~

如果想第一时间收到推送，加上关注，给我个星标⭐

谢谢你耐心看完我的文章~