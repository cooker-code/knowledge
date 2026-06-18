> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020203_Skill/020203_核心知识点/Skill能力封装与治理边界|Skill能力封装与治理边界]]
---
title: 实用工具推荐 | Skill Manager —— Agent Skill 管理面板
author: 小藕同学
date:
url: https://mp.weixin.qq.com/s?__biz=MzIzOTU0OTIzMw==&mid=2247495400&idx=1&sn=db2ff0747809c99dcff5f7c623a8d8d5&chksm=e8d3e2e570446bf97c7ea6ee8cafd2f582ff547b126038703d8211f1c4e355137019b324cb61&mpshare=1&scene=24&srcid=0320PuydoSHMuTOWWfEj2h1J&sharer_shareinfo=8caa0cb1c35730da886b5612b3ad0089&sharer_shareinfo_first=8caa0cb1c35730da886b5612b3ad0089#rd
---

🔥 GitHub上又出爆款了！让我带你们一探究竟！

---

**项目地址：** https://github.com/ryderme/skill-manager[1]

Claude Code、Codex、Cursor 等 AI 编程工具通过扫描本地目录加载 "skill"（slash 命令、agent、工作流）。同时使用多个 AI

工具时，需要为每个工具单独维护软链接，繁琐且容易出错。

**Skill Manager** 是一个轻量级的本地 Web 面板，解决这个问题：

✦ 扫描你的 skill 仓库，按项目分组展示所有 skill

✦ 直观显示每个 skill 在各工具中的链接状态（已链接 / 未链接 / 已屏蔽 / 链接错误）

✦ 点击即可切换链接，右键打开上下文菜单，支持批量操作

✦ 三级屏蔽规则：工具级 / 分组级 / Skill 级，支持白名单例外

✦ 全量同步、清理失效链接、名称冲突检测

✦ 单文件前端，无需构建；纯 Node.js 后端，无数据库

```
git clone https://github.com/ryderme/skill-manager

cd skill-manager && npm install

cp tools.example.json tools.json  # 填入你的工具路径

npm start  # 访问 http://localhost:3456

适合同时使用多个 AI 编程工具的开发者。


---

🤔 **灵魂拷问**：类似的工具你用过哪些？哪个更好用？

🔄 欢迎评论区交流，觉得不错记得转发支持一下！
```