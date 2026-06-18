> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020503_Cursor/020503_核心知识点/Cursor工程使用与上下文边界|Cursor工程使用与上下文边界]]
---
title: 【Chat】即【Cursor规则】！只需2步，Cursor自动生成 Project Rules
author: 程序视点
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg2NTg5NDI5Mw==&mid=2247504795&idx=1&sn=28c51ad037cbdb3bcd55527f550c5001&chksm=cf7f573336da8bc24b145b12805465ac3e22c4bdc71802f1cf7479455dc4b137a23b3238f32e&mpshare=1&scene=24&srcid=1106PJPXvBmcXkoPGnTZVu9S&sharer_shareinfo=ef2553b7e5d60316ec9ff92c009278d9&sharer_shareinfo_first=ef2553b7e5d60316ec9ff92c009278d9#rd
---

**因公众号更改推送规则，部分小伙伴看不到文章推送，请将****程序视点****设为星标,精品文章第一时间阅读**

大家好！欢迎来到 `程序视点`，我是你们的老朋友.安戈。

### 前言

在之前的文章中，已经位大家分享了两篇关于Cursor Rules的文章：

[Cursor AI不听指挥？资深开发者都在用的Cursor Rules配置秘籍](https://mp.weixin.qq.com/s?__biz=Mzg2NTg5NDI5Mw==&mid=2247504456&idx=1&sn=2f47d535950ab6861059e6fd30225703&scene=21#wechat_redirect)

[3层架构揭秘！企业级Cursor Rules规范这样打造更高效](https://mp.weixin.qq.com/s?__biz=Mzg2NTg5NDI5Mw==&mid=2247504672&idx=1&sn=535c36b7fd867c3d1d5c56e396b637f7&scene=21#wechat_redirect)

今天我们就`Cursor自动生成 Project Rules`功能，更进一步分享Cursor Rules的实际应用经验！

现推出Cursor Pro叠加版，每月保障$100可用额度！有需要的小伙伴，请关注【程序视点】，回复：cursor，了解参与11.11 Cursor Pro激活福利！

### 功能概述：一键自动生成项目规则

Cursor 版本的自动生成 Project Rules 功能，极大简化了规则创建流程。特别适合希望将`“对话上下文”转化为可重用规则`的开发者。具体使用方法如下：

1. **对话命令生成：** 在聊天界面输入`/Generate Cursor Rules`命令，Cursor 会**自动分析当前对话内容，提取关键上下文，自动生成新的规则集**。
2. **文件位置与格式：** 规则文件会存储在项目的`.cursor/rules/`目录下，格式为`.mdc`，包含前置元数据和具体规则内容，同时根据类别智能命名。
3. **规则自动应用：**`Auto Attached` 状态下，Agent 无需人工操作就能自动识别并应用对应规则；`Always` 设置下，长对话中规则持续激活，弥补了旧版本在长对话场景中的局限。

### 实战案例：项目的规则生成流程

实践非常简单，直接`/Generate Cursor Rules`即可。

这里引用两个典型项目进行测试，感谢Eric大佬的分享。

> https://github.com/flyeric0212/wx-md
> https://github.com/flyeric0212/cursor-rules

#### 1. **在已有规则基础上自动补充**

* 测试项目：https://github.com/flyeric0212/wx-md
* 规则补充情况：

+ general.mdc（通用规范，整合所有引用）
+ react.mdc、document.mdc、git.mdc（已有规则）
+ 新增：project-structure.mdc、performance.mdc、accessibility-i18n.mdc、security.mdc、api-integration.mdc、state-management.mdc、testing.mdc 等

#### 2. **新项目规则全自动生成**

拷贝原有项目、删除所有规则后，通过`/Generate Cursor Rules`生成命令得到：

* project-structure.mdc（项目整体结构）
* core-components.mdc（核心组件）
* core-functionality.mdc（核心功能实现）
* workflow.mdc（主要工作流程）
* tech-stack.mdc（技术栈）

可见，**已有规则为主时（上述1）,内容更详细全面；全新生成时（上述2），则更聚焦项目核心。**

### 最佳实践建议

1. **手动 + 自动双管齐下：** 先通过企业级三层体系(通用+语言+框架)手动搭建基础框架，再用自动生成功能智能补充细节。
2. **场景应用持续优化：** 定期测试并收集团队反馈，优化和细化规则内容，适应项目持续演化。
3. **注意事项与提示：**

* `高质量的AI对话`可提升规则提取的准确度
* 生成后手动调整分类和结构，以免规则重叠或遗漏
* 控制规则粒度，根据项目复杂度灵活配置文件数量

### 结语：AI 带来开发规范管理新体验

Cursor 自动生成 Project Rules 功能，极大提升了团队协作和项目规范落地效率。它特别适合初创团队或需要敏捷创新的项目。

结合手动优化和系统化管理，人人都能做“标准化+自动化”的受益者，最大程度释放创造力。

**如果你对 AI 辅助开发还有更多实践心得，欢迎留言交流，让我们一起推动开发流程迭代升级！**

---

### 最后

`Cursor AI编辑器` = `传统编辑器的强大功能` + `AI辅助能力`，为开发者提供了一个AI辅助编程的高效开发环境。

经过前几天Cursor风控后，虽然我们已经率先恢复出号，但Cursor Pro可使用额度大幅下跌。现推出Cursor Pro叠加版，每月为大家保障至少$100可用额度！

有需要的小伙伴，请关注【程序视点】，回复：`cursor`，了解参与`11.11 Cursor Pro激活优惠`！

更多激活内容，也可以扫描下方二维码，按需备注，直接参与。

回复：`vip`，获取专属JetBrains全家桶IDE激活；
回复：`cursor`，获取Cursor激活；
回复：`copilot`，获取GitHub Copilot激活；
回复：`ai`，获取AI Assistant激活；
回复：`claude`，获取Claude Code激活；

【程序视点】助力打工人减负，从来不是说说而已！后续安戈会继续详细分享更多实用的工具和功能。

如果你觉得这篇教程有帮助，别忘了【点赞+分享+推荐】三连支持！

后续安戈会持续分享更多开发工具和技巧，敬请期待！如果有其他工具需求，欢迎留言讨论~  🚀