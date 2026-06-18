> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Claude Code 2.1.116 发布：大 session 恢复更快，终端和插件细节继续补强
author: 阿俊聊AI
date:
url: https://mp.weixin.qq.com/s?__biz=MzYzODc1NzU3Ng==&mid=2247483945&idx=1&sn=0cbbe5601970ccab9b2ba88ca92ba873&chksm=f15e9336a412224c1b9b93978624a318f288382cc471e2dc607d379585975a42599f11e27930&mpshare=1&scene=24&srcid=0422GkznnvBpuX6AeKnLaWbu&sharer_shareinfo=56d09372e3b5798d18af9770fe8a1340&sharer_shareinfo_first=56d09372e3b5798d18af9770fe8a1340#rd
---

一句话看这版：**它没有堆很多新入口，重点都落在高频工作流上——大体量会话恢复更快了，MCP 启动更轻了，终端交互、插件更新和安全边界也都继续往前推了一步。**

---

## 这次更新最值得关注的几个点

### 1. `/resume` 在大体量会话里明显更快了

这版最实在的一项改进，就是 **`/resume` 对大 session 的恢复速度明显提升**。

官方给出的信息很直接：

* 在 40MB 以上的大会话里，恢复速度最高可提升 **67%**
* 对包含大量 dead-fork entries 的会话，处理效率也更高

**价值：** 如果你平时会长时间连续使用 Claude Code，或者会话里积累了很多上下文，这次更新最直接的意义就是等待时间更短，大会话重新接上工作的过程会顺很多。

---

### 2. MCP 启动更快了，多 stdio server 场景更省时间

2.1.116 也优化了 MCP 启动流程。

官方说明是：当配置了多个 stdio server 时，MCP 启动会更快；同时 `resources/templates/list` 现在会延迟到第一次 `@` mention 时再加载。

**价值：** 这会直接减少启动阶段的额外等待。尤其是 MCP 配置比较重、server 比较多的用户，会更容易感受到进入工作状态的速度提升。

---

### 3. 终端和编辑器里的交互又顺了一些

这一版在终端交互细节上也做了不少调整：

* VS Code、Cursor、Windsurf 终端里的全屏滚动更顺滑了
* `/terminal-setup` 现在会配置编辑器的滚动灵敏度
* thinking spinner 改成行内显示进度，例如 “still thinking”“thinking more”“almost done thinking”
* `/doctor` 现在可以在 Claude 正在回复时打开，不用等当前轮次结束
* `/config` 搜索现在也会匹配配置项的值，比如搜索 `vim` 能找到 Editor mode

**价值：** 这些变化看起来都不算大，但都跟高频使用时的流畅度有关。对天天开着 Claude Code 工作的人来说，这些小改动是能真实感受到的。

---

### 4. 插件和 Agent 工作流继续补齐

插件和 Agent 相关，这次也有几项很实用的增强：

* `/reload-plugins` 和后台插件自动更新，现在会自动安装你已添加 marketplace 中缺失的插件依赖
* Agent frontmatter hooks 现在在通过 `--agent` 作为主线程 agent 运行时也会触发
* slash command 菜单在筛选结果为空时，现在会明确显示 “No commands match”，而不是直接消失

**价值：** 这类更新的重点，不是“多了一个功能”，而是减少工作流被打断的次数。插件依赖缺失、命令菜单反馈不清楚、agent 行为和预期不一致，这些问题虽然零碎，但都挺影响日常使用；2.1.116 把这些环节又补齐了一些。

---

### 5. 安全提示和用量显示也更稳了

除了体验层面的优化，这版还有几项很实用的补强：

* Bash 工具在 `gh` 命令遇到 GitHub API rate limit 时，会给出提示，方便 agent 回退而不是反复重试
* Settings 里的 Usage 标签页，现在会立刻显示 5 小时和每周用量；即使用量接口被限流，也不会直接失败
* 安全方面，sandbox auto-allow 不再绕过对 `rm` / `rmdir` 指向 `/`、`$HOME` 或其他关键系统目录时的危险路径检查

**价值：** 一部分提升的是可见性，另一部分提升的是安全边界。前者让你更早知道发生了什么，后者则减少了危险操作被错误放行的可能。

---

## 这次还修了哪些值得注意的问题？

### 终端显示与键盘交互

* 修复 Devanagari 等 Indic scripts 在终端 UI 中列对齐异常的问题
* 修复使用 Kitty keyboard protocol 的终端里，`Ctrl+-` 不能触发撤销的问题
* 修复使用 Kitty keyboard protocol 的终端里，`Cmd+Left/Right` 不能跳到行首/行尾的问题
* 修复通过 wrapper process 启动 Claude Code 时，`Ctrl+Z` 可能导致终端挂起的问题
* 修复 inline mode 下，终端 resize 或大段输出时 scrollback 重复的问题
* 修复终端高度较小时，模态搜索框溢出屏幕、搜索框和键盘提示被遮挡的问题
* 修复 VS Code 集成终端滚动时，零散空白单元格和 composer chrome 消失的问题

---

### 大会话、分支与工作树相关问题

* 修复与 cache control TTL 排序有关的间歇性 API 400 错误，该问题可能在并行请求于请求初始化过程中完成时触发
* 修复 `/branch` 拒绝处理超过 50MB transcript 会话的问题
* 修复 `/resume` 在大型 session 文件上静默显示空白对话，而不是报告加载错误的问题
* 修复在 session 中途进入 worktree 后，`/update` 和 `/tui` 不可用的问题

---

### 插件与列表状态问题

* 修复 `/plugin` Installed 标签页里，同一个项目在 Needs attention 或 Favorites 下重复显示的问题

---

## 这版更新最大的用户价值是什么？

### 1. 大会话终于更好用了

`/resume` 提速和大 session 相关修复，直接改善了重度用户最容易遇到的等待和失败问题。

### 2. 启动到可用的路径更短了

MCP 启动优化、延迟加载 `resources/templates/list`，都在缩短进入工作状态前的准备时间。

### 3. 高频交互更顺手了

全屏滚动、thinking spinner、`/doctor`、`/config` 搜索这些变化，都是日常高频动作里的体验优化。

### 4. 插件和 Agent 行为更可预期了

依赖自动安装、hooks 生效范围补齐、空结果提示更明确，都在减少“为什么这次不对劲”的情况。

### 5. 边界更稳，提示也更明确了

无论是 GitHub API rate limit 提示，还是危险路径保护加强，本质上都在降低误操作和误判断的成本。

---

## 怎么看 Claude Code 2.1.116？

如果说有些版本更强调“加了什么新能力”，那 **2.1.116 更像是在把已经形成的工作流继续磨顺、磨稳。**

它没有推出特别抢眼的新入口，但围绕下面这些真实使用场景做了持续补强：

* 大会话恢复
* MCP 启动性能
* IDE 终端滚动和输入体验
* 插件依赖补全
* Agent hooks 一致性
* 安全保护与状态提示

对高频用户来说，这种版本的价值通常很直接：不一定更花哨，但更省时间，也更少打断。