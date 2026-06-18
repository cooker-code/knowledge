> 已吸收至：[[02_Agent与AI工程/0204_评估与观测/020402_Agent评估/020402_核心知识点/Agent评估体系与观测边界|Agent评估体系与观测边界]]
---
title: 测试视角的 Code Review：Agent 自动发现代码中的质量风险
author: AI测试圈
date: AI测试圈AI测试圈
url: https://mp.weixin.qq.com/s?__biz=MzYzMzg4MjE1NA==&mid=2247483815&idx=1&sn=7a1ad90d27fd0d6d447ebc6060e1edbb&chksm=f1d6c56b45363b23f2513074d8126e2470e173fac9bf934816d36320c2b4b15f4c9ec6209271&mpshare=1&scene=24&srcid=0512tM8PFvIUvaWdl2xmOkHJ&sharer_shareinfo=82c2094619f5abe58ab15f7fc1594a77&sharer_shareinfo_first=82c2094619f5abe58ab15f7fc1594a77#rd
---

开发提了 PR，你作为测试人去看代码，第一反应是什么？大概率是"这代码我也看不太懂"。但换个角度想——你不需要看懂业务逻辑的实现细节，你需要看的是：异常处理够不够、边界条件考虑了没、日志打了没、这段代码好不好测。这些恰恰是 Agent 擅长的。

## 测试人看代码，看什么？

传统 Code Review 是开发互审，关注实现逻辑、设计模式、性能。测试人参与 Code Review 的视角完全不同：

* **可测试性**

  ：函数是否可以被单独测试？依赖是否可注入？
* **异常处理**

  ：空值、超时、网络异常有没有处理？
* **边界条件**

  ：数组越界、整数溢出、空集合有没有考虑？
* **日志与可观测性**

  ：出错时能不能定位问题？关键操作有没有日志？
* **状态一致性**

  ：并发场景下数据会不会不一致？

这些检查项是有规律的，可以规则化，Agent 天然适合干这个。

## code-review-agent Skill 配置

**name:** code-review-agent
**description:** 从测试视角审查代码变更，识别可测试性问题和质量风险
**version:** 1.0.0
**author:** AI测试圈

**触发条件**

当用户提供代码变更（diff、PR链接、或文件路径）时激活。

**审查维度**

按以下 5 个维度逐一扫描，每个维度独立输出发现：

**1. 异常处理完整性**

* 外部调用（HTTP/RPC/DB）是否有 try-catch 或错误码处理
* 是否存在吞异常（catch 块为空或只打日志不处理）
* 超时场景是否设置了合理的 timeout

**2. 边界条件覆盖**

* 空值/null 检查是否到位
* 集合操作是否处理了空集合
* 数值运算是否考虑溢出和除零
* 字符串操作是否处理了空串和超长输入

**3. 可测试性评估**

* 函数参数是否过多（>5 个需拆分）
* 是否存在硬编码的外部依赖（直接 new 而非注入）
* 单个函数是否超过 50 行（难以单测覆盖）
* 是否有隐式依赖（全局变量、静态方法调用）

**4. 日志与可观测性**

* 关键业务操作是否有 INFO 级别日志
* 异常路径是否有 ERROR 级别日志且包含上下文
* 是否有足够信息用于问题排查（请求ID、用户ID、操作类型）

**5. 并发与状态安全**

* 共享资源访问是否有锁保护
* 数据库操作是否在事务内
* 缓存更新与 DB 更新的顺序是否正确

**输出格式**

对每个发现，输出：

* 风险等级：🔴 HIGH / 🟡 MEDIUM / 🟢 LOW
* 位置：文件名:行号
* 问题描述：一句话说明
* 建议修复：具体怎么改
* 测试建议：针对这个问题应补充什么测试

**约束**

* 只关注测试相关的质量风险，不评价业务逻辑设计
* 不建议大规模重构，只针对当前变更范围
* 输出按风险等级排序，HIGH 在前

## 实战演示：审查一个支付模块 PR

假设开发提了一个「余额扣款」功能的 PR，核心代码如下：

`public void deductBalance(String userId, BigDecimal amount) {
    User user = userDao.findById(userId);
    if (user.getBalance().compareTo(amount) >= 0) {
        user.setBalance(user.getBalance().subtract(amount));
        userDao.save(user);
    }
}`

把这段代码交给 code-review-agent，输出：

**🔴 HIGH — 并发安全问题**

* 位置：PaymentService.java:42
* 问题：余额扣减没有加锁，并发请求可能导致超扣
* 建议修复：使用 SELECT ... FOR UPDATE 或乐观锁版本号
* 测试建议：并发测试——10 个线程同时扣款，验证余额不为负

**🔴 HIGH — 异常处理缺失**

* 位置：PaymentService.java:40
* 问题：findById 返回 null 时直接 NPE，余额不足时静默失败无反馈
* 建议修复：增加 null 检查和余额不足的异常抛出
* 测试建议：用户不存在和余额不足的负向用例

**🟡 MEDIUM — 日志缺失**

* 位置：PaymentService.java:40-45
* 问题：扣款操作无任何日志，出问题无法追溯
* 建议修复：扣款前后各记一条 INFO 日志，包含 userId、amount、余额变化
* 测试建议：验证日志输出包含必要字段

**🟡 MEDIUM — 可测试性**

* 位置：PaymentService.java:40
* 问题：直接依赖 userDao 实例，无法 mock 测试
* 建议修复：通过构造器注入 UserDao
* 测试建议：确认单测可以注入 mock DAO

## 在 Claude Code 中使用

两种典型用法：

**方式一：直接对文件调用**

`# 对某个文件进行测试视角审查
claude`

`"用agent 审查 src/payment/PaymentService.java"`

**方式二：结合 Git diff**

`# 审查最近一次提交的变更
git diff HEAD~1 | claude "用 code-review-agent 审查这个 diff"`

**方式三：配合 Hook 自动触发**

在 .claude/settings.json 中配置 Hook，每次 git commit 前自动运行审查：

`{ "hooks": { "pre-commit": { "command": "git diff --cached | claude --skill code-review-agent --quiet" } } }`

这样开发每次提交代码，Agent 会自动跑一遍测试视角的审查，有 HIGH 级别问题就阻断提交。

## 3 个踩过的坑

**坑 1：审查范围太大 → 噪音爆炸**

把整个仓库丢给 Agent 审查，出来几百条 findings，根本看不完。正确做法：只审查本次 PR 的 diff，聚焦增量变更。

**坑 2：不区分风险等级 → 什么都改**

如果不分优先级，团队会把所有 findings 都当 Bug 提，开发怨声载道。正确做法：只强制修复 HIGH，MEDIUM 建议修复，LOW 知悉即可。

**坑 3：只扫描不跟进 → 形同虚设**

Agent 找到了问题但没人跟进验证，那和没扫一样。正确做法：HIGH 级别的 findings 自动创建 Jira Issue，并关联到对应测试用例。

## 总结

* **code-review-agent**

  从 5 个维度审查代码：异常处理、边界条件、可测试性、日志、并发安全
* 测试人不需要看懂实现逻辑，只需要关注"这段代码好不好测、容不容易出 Bug"
* 结合 Git Hook 可以实现提交时自动审查，质量门禁前置
* 审查范围控制在 PR 级别，按风险等级分类处理
* Agent 出 findings，人来决定哪些必须修、哪些可以接受