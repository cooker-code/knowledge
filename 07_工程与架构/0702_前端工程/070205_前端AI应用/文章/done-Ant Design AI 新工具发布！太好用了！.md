> 已吸收至：[[07_工程与架构/0702_前端工程/070205_前端AI应用/070205_核心知识点/前端AI生成式UI与工具调用边界|前端AI生成式UI与工具调用边界]]、[[07_工程与架构/0702_前端工程/070205_前端AI应用/070205_知识地图|070205_前端AI应用知识地图]]

---
title: Ant Design AI 新工具发布！太好用了！
author: 前端开发爱好者
date:
url: https://mp.weixin.qq.com/s?__biz=MzUzNTk3MjE2Ng==&mid=2247522915&idx=1&sn=e26000f8d47360dcf02abbd27d31726e&chksm=fbb04f065e72f33267983c6784cb43694529f8e9d98442197c020f016e103e3e4340affb32ac&mpshare=1&scene=24&srcid=0328m94cHGtAEfJRELl0hl8a&sharer_shareinfo=b60ece91e57d4a8035e7f2232fdb4559&sharer_shareinfo_first=b60ece91e57d4a8035e7f2232fdb4559#rd
---

**你还在为查文档疯狂切屏？AI 时代，这种方式太落后了！**

想象一下这个场景：

你正在用 **Cursor** 写代码，突然忘了 `DatePicker` 的 `format` 属性怎么传。于是你——

1. 切到浏览器
2. 打开 `antd` 官网
3. 搜索 `DatePicker`
4. 滚动半天找到 `API` 表格
5. 切回编辑器，发现刚才的思路断了...

**5 分钟就这么没了。一天重复 20 次，就是 100 分钟！**

如果有个工具，能在你敲代码的同时，毫秒级给出答案，甚至直接帮你生成代码，会怎样？

今天，Ant Design 官方放了个大招—— **`@ant-design/cli` 正式发布！**

## ant-design/cli 到底能做什么？

简单说：**把 Ant Design 的整个文档库，装进你的命令行，完全离线使用。**

不用联网、不用浏览器、不用等页面加载。

敲一行命令，答案直接甩脸上。

```
# 想知道 Button 组件有哪些属性？
antd info Button

# 想看 Select 的基础示例代码？
antd demo Select basic

# 想知道 v4 升 v5 时 Select 改了啥？
antd changelog 4.24.0 5.0.0 Select
```

**毫秒级响应。**比你切浏览器快 100 倍。

## 三大杀招，直击痛点

#### 完全离线，零延迟

所有数据打包在本地。飞机上的断网环境？照样查文档。**55+ 个版本快照**，从 v4 到 v6，精确到每个小版本。

比如你想知道 `antd@5.3.0` 的 `Button` 长啥样，它能给你准确的答案，而不是"大概是 v5 的样子"。

#### AI 智能体原生支持

这才是重头戏！

如果你用 **Claude Code、Cursor、Codex、Gemini CLI**，这个工具就是为你量身定制的。

```
# 一键添加为 Agent Skill
npx skills add ant-design/ant-design-cli
```

添加后，你的 AI 助手直接"开天眼"：

* 你写代码时，AI 自动查询正确的 API
* 你问"怎么改这个组件"，AI 直接给出准确的迁移方案
* 甚至能帮你自动修复废弃的 API

**AI 不再瞎编代码，因为它有了官方知识库。**

#### 项目诊断 + 自动迁移

老项目想升级 antd 版本？头疼吧？

```
# 一键诊断项目问题
antd doctor

# 检查代码里的废弃 API
antd lint ./src

# v4 升 v5，自动生成迁移脚本
antd migrate 4 5 --apply ./src
```

**25+ 个 v4→v5 迁移步骤，30+ 个 v5→v6 迁移步骤**，全部内置。AI 照着清单帮你改，不出错。

## 安装只需 10 秒

```
npm install -g @ant-design/cli
```

然后直接开用。零配置，零学习成本。

## 真实场景对比

| 场景 | 传统方式 | 用 @ant-design/cli |
| --- | --- | --- |
| 查 **Button** 属性 | 开浏览器 → 搜官网 → 找 API→2 分钟 | `antd info Button` →**2 秒** |
| 看 **Select** 示例 | 官网找 Demo→ 复制代码 →1 分钟 | `antd demo Select basic` →**3 秒** |
| **v4** 升 **v5** 迁移 | 看迁移文档 → 手动改 → 半天 | `antd migrate 4 5 --apply` →**AI 自动改** |
| 查废弃 **API** | 翻 Changelog→ 对比版本 →10 分钟 | `antd changelog 4.24.0 5.0.0` →**5 秒** |

**保守估计，效率提升 300% 起步。**

## 哪些人需要这个工具？

✅ **用 Cursor/Claude 写代码的开发者** — AI 有了官方知识库，代码质量直接起飞
✅ **维护老项目的全栈工程师** — 迁移版本不再痛苦
✅ **离线环境工作的程序员** — 飞机上、内网里照样查文档
✅ **团队技术负责人** — 统一团队代码规范，自动检查废弃 API

## 写在最后

**Ant Design** 这次发布的 **CLI** 工具，不只是一个"命令行查文档"的小玩具。

它是 **AI 编程时代的标准基础设施** —— 让 AI 真正理解你的组件库，写出靠谱代码。

如果你还在浏览器和编辑器之间疯狂切屏，如果你还在为 antd 版本迁移头疼，如果你希望 AI 助手不再瞎编 API...

**现在就去试试：**

```
npm install -g @ant-design/cli
antd info Button
```

**3 秒钟后，你会回来感谢我的。**

并且 **ant-design/cli** 工具已开源：`github.com/ant-design/ant-design-cli`
*支持 antd v4/v5/v6，完全离线，永久免费。*

* **更多使用方式**：`https://github.com/ant-design/ant-design-cli/blob/main/README.zh-CN.md`