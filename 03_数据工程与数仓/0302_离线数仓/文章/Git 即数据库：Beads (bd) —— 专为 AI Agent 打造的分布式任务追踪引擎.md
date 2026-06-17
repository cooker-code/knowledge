---
title: Git 即数据库：Beads (bd) —— 专为 AI Agent 打造的分布式任务追踪引擎
author: TonyBai
date: 
url: https://mp.weixin.qq.com/s?__biz=MzIyNzM0MDk0Mg==&mid=2247504974&idx=1&sn=354bb47496abfa1f7987060068aff715&chksm=e992793c73d300ae060d72471bda0727925a37e154721ec4efd88eeec10105f41337e2686653&mpshare=1&scene=24&srcid=0204LO4toA6gWp9B6sGXupUW&sharer_shareinfo=9f6cf80e2465854af9a2175dcaae86ac&sharer_shareinfo_first=9f6cf80e2465854af9a2175dcaae86ac#rd
---

大家好，我是Tony Bai。

在 AI 编码智能体（如 [Claude Code](https://mp.weixin.qq.com/s?__biz=MzIyNzM0MDk0Mg==&mid=2247503448&idx=1&sn=16c0e8fff144d70d0cbe5b5d1859833e&scene=21#wechat_redirect)、[Gemini CLI](https://mp.weixin.qq.com/s?__biz=MzIyNzM0MDk0Mg==&mid=2247500370&idx=2&sn=e44d9dda16dffdce1e0ecc7e12a10576&scene=21#wechat_redirect) 等）日益普及的今天，我们面临着一个棘手的工程难题：AI Coding Agent 虽然极其聪明，但它们通常是"健忘"的。

它们在处理一个长期、复杂的重构任务时，往往会在海量的上下文切换中迷失方向。传统的 Issue Tracker（如 Jira）对 AI 来说太重、太慢且难以集成；而简单的 Markdown 文件又缺乏结构化和版本控制。

于是，**Beads**（命令行工具 `bd`）应运而生。它是 [Gas Town](https://mp.weixin.qq.com/s?__biz=MzIyNzM0MDk0Mg==&mid=2247504858&idx=1&sn=031ff6f85643affc2887e97e535175fb&scene=21#wechat_redirect) —— 那个被誉为 AI Coding 领域"[Kubernetes](https://mp.weixin.qq.com/s?__biz=MzIyNzM0MDk0Mg==&mid=2247503365&idx=3&sn=f02cf4ad80d65c2982214c35540a5bfc&scene=21#wechat_redirect)"的宏大愿景 —— 的底层记忆基石。它巧妙地利用 Git 作为分布式数据库，为 AI Agent 提供了一个持久化、可协作、依赖感知的任务追踪系统。

## 为什么 AI Agent 需要 Beads？

传统的软件工程工具是为**人类**设计的，而 Beads 是为**AI 智能体**设计的。

### 上下文的持久化

AI 模型的上下文窗口（Context Window）虽然越来越大，但依然昂贵且有限。当一个 Agent 需要处理跨越数周、涉及数百个文件的任务时，它不能一直把所有信息都塞进 Prompt 里。

Beads 提供了一个外部的、结构化的存储，让 Agent 可以随时"卸载"和"重载"任务状态，就像人类使用笔记本一样。

### 原生的依赖管理

复杂的编码任务往往是一张有向无环图（DAG）：“先重构数据库 Schema，再更新 API，最后修复前端。”

Beads 原生支持任务依赖（Dependency Graph）。它能自动计算出当前的 **Ready Work（就绪工作）** ，告诉 Agent："别瞎忙，现在你只能做这个，其他的都还被阻塞着。"

### 分布式协作

如果是多个 Agent（或者人类与 Agent）同时工作怎么办？

Beads 将任务数据存储为 `.beads/` 目录下的 JSONL 文件。 这意味着：**任务即代码**。你可以像合并代码一样，通过 Git 分支、合并、解决冲突来管理任务。

## 核心架构：Git as a Database

Beads 的设计哲学极其硬核：它不想引入任何外部的中心化服务，它只想利用你现有的 Git 仓库。

### 三层分层架构：清晰的职责边界

Beads 采用了经典的**三层架构**，但在每一层都做了针对性的优化：

* **CLI Layer**：基于 `spf13/cobra` 构建，负责命令解析和用户交互。 它不直接操作数据，而是优先通过 RPC 与守护进程通信，失败时才降级到直接数据库访问。
* **Daemon Process**：每个工作区运行独立的后台守护进程，处理 RPC 请求、协调自动同步时机，并持有数据库连接以加速查询。通过 Unix domain socket（Windows 上为 named pipes）进行通信。
* **Storage Layer**：这是核心。它通过**接口隔离原则**定义了 `Storage` 接口，支持 SQLite、Dolt 甚至内存等多种后端。这种设计使得底层的存储实现可以被轻松替换，而不影响上层逻辑。

更为完整的架构参考下图：

来自deepwiki.com对beads源码的分析结果

### 双存储写屏障：SQLite 与 JSONL 的完美同步

Beads 最精妙的设计之一是它的双存储写屏障 (Dual-Storage Write Barrier)。它是如何解决 SQLite（高性能查询）与 JSONL（版本控制）之间的数据一致性的呢？

* **写入路径**：当用户创建任务时，数据**首先写入 SQLite**，保证了毫秒级的操作反馈。
* 防抖刷新 (Debounced Flush)：为了避免频繁的磁盘 I/O 和 Git 操作，Beads 实现了一个基于 Go Channel 的事件驱动 FlushManager。所有 flush 状态（isDirty、needsFullExport、debounceTimer）由单个后台 goroutine 拥有，通过 channel 通信消除了竞态条件。

系统默认配置 30 秒的防抖窗口（可通过 `flush-debounce` 配置调整），这为批量操作提供了"事务窗口"——30 秒内的多次修改会被合并成一次 JSONL 导出，避免了频繁的 Git commit。

这种机制确保了在高频操作下（如批量导入），系统不会因为频繁的 Git Commit 而卡顿。

### 并发安全与锁机制

在分布式和多进程环境下，数据竞争是最大的敌人。Beads 采取了多重防御：

* **进程级互斥**：使用文件锁（Unix 上为 `flock`，Windows 上为 `LockFileEx`）对守护进程（`daemon.lock`）和同步操作（`.sync.lock`）加锁，确保同一时间只有一个守护进程在运行，且不会有并发的 sync 操作导致数据损坏。
* **数据库连接池优化**：SQLite 连接池根据数据库类型进行智能配置：

+ 内存数据库：`SetMaxOpenConns(1)` 强制单连接，因为 SQLite 的内存数据库在连接间是隔离的。
+ 文件数据库：`SetMaxOpenConns(runtime.NumCPU() + 1)` 利用 SQLite WAL 模式的"1 writer + N readers"特性，同时防止写锁竞争导致的 goroutine 堆积。

* **Context 传播**：所有的存储操作都强制要求传递 `context.Context`，确保了超时控制和优雅退出的能力，这对于一个长期运行的后台守护进程至关重要。

### 自适应哈希 ID：算法的胜利

为了在"简短易读"和"全局唯一"之间取得平衡，Beads 没有使用 UUID，而是设计了一套**自适应哈希算法**：

* **综合哈希源**：ID 并非简单的标题哈希，而是综合了 title、description、creator、timestamp 和 nonce 的 SHA256 哈希，确保了即使标题相同，不同时间创建的 issue 也有不同的 ID。
* **Base36 编码**：使用 base36（0-9, a-z）而非 hex 编码，提供了更高的信息密度，让 ID 更短。
* **动态长度**：系统根据当前数据库的规模，使用生日悖论数学计算碰撞概率，自动调整 ID 的长度：

+ 小型数据库：`bd-a1b2` (4 字符)
+ 中型数据库：`bd-a1b2c3` (6 字符)
+ 大型数据库：最多 8 字符

* **碰撞处理**：在生成 ID 时，如果检测到碰撞，系统会尝试最多 10 个 nonce 值，如果仍然碰撞，则增加哈希长度。这是一种典型的用**计算换取协作体验**的策略。

### Beads Issue 状态

Beads 定义了 **8 个核心状态**：

* **`open`** - 可开始的工作
* **`in_progress`** - 正在进行中
* **`blocked`** - 被依赖阻塞
* **`deferred`** - 暂时延期
* **`closed`** - 已完成
* **`tombstone`** - 软删除（30天后清理）
* **`pinned`** - 永久保持开放
* **`hooked`** - AI智能体钩子工作

它们的状态机转换流程如下图所示：

这个状态机设计确保了数据一致性、合并安全性和自动化工作流。

### 依赖管理的多样性

Beads 支持多种依赖类型，不同类型有不同的语义：

* `blocks`：阻塞依赖，Issue X 必须关闭后 Y 才能开始，影响 `bd ready` 计算
* `parent-child`：层级关系，用于 Epic 和子任务，父节点被阻塞时子节点也被阻塞
* `related`：软链接，仅用于引用，不影响执行顺序
* `discovered-from`：记录在执行某任务时发现的新问题

系统使用递归 CTE(Common Table Expression) 检测循环依赖，确保依赖图始终是一个有向无环图（DAG）。

### Blocked Issues Cache：性能的飞跃

在大型项目中，实时计算哪些 issue 被阻塞可能非常耗时。Beads 引入了 **Blocked Issues Cache** 机制，这是一个关键的性能优化：

* **问题**：在 10K issue 的数据库上，使用递归 CTE 实时计算阻塞状态需要约 752ms。
* **解决方案**：使用 `blocked_issues_cache` 表物化阻塞计算结果。
* **效果**：查询时间降至约 29ms，**性能提升 25 倍**。

缓存在每次依赖变更或状态变更时完全重建（DELETE + INSERT），虽然重建需要 <50ms，但由于依赖变更相对读取操作非常罕见，这个权衡是值得的。

## 实战：Agent 的工作流

让我们看看在一个典型的 AI 编码场景中，Beads 是如何工作的。

**场景**：你需要重构一个遗留系统的用户认证模块。

1. **初始化与规划**： Agent 首先通过 `bd create` 创建主任务（Epic）。 注意，Beads 支持层级 ID，这对于 AI 拆解任务非常有帮助。

   ```
   # 创建 Epic  
   $ bd create "重构用户认证模块"  
   Created: bd-auth01  
     
   # 拆分子任务（注意：Beads 支持层级结构，或者我们可以手动关联）  
   $ bd create "设计新 User 表结构"  
   Created: bd-db002  
     
   $ bd create "迁移旧数据"  
   Created: bd-migr03  
     
   $ bd create "切换 API 逻辑"  
   Created: bd-api004
   ```
2. **建立依赖**： Agent 知道事情有轻重缓急，它会建立依赖关系。根据 `bd dep add <child> <parent>` 格式（child 依赖 parent，即 parent blocks child）：

   ```
   # bd-migr03 (Child) 依赖于 bd-db002 (Parent)  
   # 意味着：必须先设计完表结构，才能迁移数据  
   $ bd dep add bd-migr03 bd-db002  
     
   # bd-api004 (Child) 依赖于 bd-migr03 (Parent)  
   # 意味着：必须先迁移完数据，才能切换 API  
   $ bd dep add bd-api004 bd-migr03
   ```
3. **获取就绪工作**： Agent 不再迷茫，它只需要问 Beads："我现在能做什么？"

   ```
   $ bd ready --json  
   {  
     "issues": [  
       {  
         "id": "bd-db002",  
         "title": "设计新 User 表结构",  
         "status": "pending",  
         "blocks": ["bd-migr03"]  
         ...  
       }  
     ]  
   }
   ```

   Beads 会告诉它，只有"设计表结构"是 Ready 的，其他的都被阻塞了。
4. **执行与更新**： Agent 完成任务后，关闭 Issue。

   ```
   $ bd close bd-db002
   ```

   此时，后台的 blocked cache 自动重建，"迁移旧数据" (`bd-migr03`) 的任务状态瞬间变为 Ready。

## 高阶实战：Claude Code 与 Beads 的"双人舞"

如果说上面的命令是基本舞步，那么当 Claude Code 遇上 Beads，它们能跳出怎样的双人舞？让我们看一个\*\*"任务中断与恢复"\*\*的真实场景。

**0. 前置配置：教会 Claude 使用工具**

要让 Claude Code 懂得使用 Beads，我们首先需要在项目的根目录下创建一个`CLAUDE.md` 文件(其它Coding agent一般支持`AGENTS.md`)。这是 Claude Code 的"行动指南"。

创建beads-demo目录，使用git init初始化该目录：

```
$mkdir beads-demo  
$cd beads-demo  
$git init .
```

执行bd init初始化该目录：

```
bd init  
  Repository ID: 3f0bbad4  
  Clone ID: c15058f43e4678f2  
  ✓ Created AGENTS.md with landing-the-plane instructions  
  
✓ bd initialized successfully!  
  
  Backend: sqlite  
  Database: .beads/beads.db  
  Issue prefix: beads-demo  
  Issues will be named: beads-demo-<hash> (e.g., beads-demo-a3f2dd)  
  
Run bd quickstart to get started.  
  
⚠ Setup incomplete. Some issues were detected:  
  • Git Hooks: Missing 1 recommended hook(s)  
  • Sync Divergence: 1 sync divergence issue(s) detected  
  • Claude Integration: Not configured  
  • Git Working Tree: Uncommitted changes present  
  • Git Upstream: No upstream configured for master  
  • Version Tracking: Version tracking not initialized  
  
Run bd doctor --fix to see details and fix these issues.
```

beads创建了AGENTS.md文件，我们将之改名为CLAUDE.md，并写入下面这行文字：

```
$mv AGENTS.md CLAUDE.md  
$echo "Use 'bd' for task tracking" >> CLAUDE.md
```

有了CLAUDE.md 这份"说明书"，Claude Code 就会在启动时自动扫描并理解这套规则。

**1. 记忆的建立**

**场景**：你让 Claude Code (在plan模式下，disable一些不必要的插件或skill)使用 Go 开发一个"简单的并发下载器"。它会创建了任务，并建立任务间的依赖关系：

```
● Bash(bd create --type feature --title "项目初始化" --description "初始化Go项目结构和基础配置…)                                    
  ⎿  ✓ Created issue: beads-demo-dee                                                                                                
       Title: 项目初始化                                                                                                            
       Priority: P2                                                                                                                 
       Status: open                                                                                                                 
                                                                                                                                    
● Bash(bd create --type feature --title "核心下载逻辑实现" --description "实现下载器的核心功能…)                                    
  ⎿  ✓ Created issue: beads-demo-9uq                                                                                                
       Title: 核心下载逻辑实现                                                                                                      
       Priority: P2                                                                                                                 
       Status: open                                                                                                                 
                                                                                                                                    
● Bash(bd create --type feature --title "进度显示实现" --description "实现终端进度条功能…)                                          
  ⎿  ✓ Created issue: beads-demo-8pf                                                                                                
       Title: 进度显示实现                                                                                                          
       Priority: P2                                                                                                                 
       Status: open                                                                                                                 
                                                                                                                                    
● Bash(bd create --type feature --title "断点续传实现" --description "实现状态持久化和恢复功能…)                                    
  ⎿  ✓ Created issue: beads-demo-fhv                                                                                                
       Title: 断点续传实现                                                                                                          
       Priority: P2                                                                                                                 
       Status: open                                                                                                                 
                                                                                                                                    
● Bash(bd create --type feature --title "CLI入口实现" --description "实现命令行工具入口…)                                           
  ⎿  ✓ Created issue: beads-demo-4bb                                                                                                
       Title: CLI入口实现                                                                                                           
       Priority: P2                                                                                                                 
       Status: open   
  
... ...  
  
  
● 现在设置依赖关系：                                                                                     09:24 AM  glm-4.7          
                                                                                                                                    
● Bash(bd dep add beads-demo-9uq --blocked-by beads-demo-dee)                                                                       
  ⎿  ✓ Added dependency: beads-demo-9uq depends on beads-demo-dee (blocks)                                                          
                                                                                                                                    
● Bash(bd dep add beads-demo-8pf --blocked-by beads-demo-9uq)                                                                       
  ⎿  ✓ Added dependency: beads-demo-8pf depends on beads-demo-9uq (blocks)                                                          
                                                                                                                                    
● Bash(bd dep add beads-demo-fhv --blocked-by beads-demo-9uq)                                                                       
  ⎿  ✓ Added dependency: beads-demo-fhv depends on beads-demo-9uq (blocks)                                                          
                                                                                                                                    
● Bash(bd dep add beads-demo-4bb --blocked-by beads-demo-9uq --blocked-by beads-demo-8pf --blocked-by beads-demo-fhv)               
  ⎿  ✓ Added dependency: beads-demo-4bb depends on beads-demo-fhv (blocks)                                                          
                                                                                                                                    
● Bash(bd dep add beads-demo-4id --blocked-by beads-demo-4bb)                                                                       
  ⎿  ✓ Added dependency: beads-demo-4id depends on beads-demo-4bb (blocks)                                                          
                                                                                                                                    
● Bash(bd list)                                                                                                                     
  ⎿  ○ beads-demo-4id [● P2] [task] - 编写测试                                                                                      
     ○ beads-demo-4bb [● P2] [feature] - CLI入口实现                                                                                
     ○ beads-demo-fhv [● P2] [feature] - 断点续传实现                                                                               
     ○ beads-demo-8pf [● P2] [feature] - 进度显示实现                                                                               
     ○ beads-demo-9uq [● P2] [feature] - 核心下载逻辑实现                                                                           
     ○ beads-demo-dee [● P2] [feature] - 项目初始化                                                                                 
                                                                                   
● Bash(bd graph --all)                                                                                                              
  ⎿  📊 Dependency graph for beads-demo-4id:                                                                                        
                                                                                                                                    
       Status: ○ open  ◐ in_progress  ● blocked  ✓ closed                                                                           
                                                                                                                                    
       Layer 0 (ready)                                                                                                              
       ┌────────────────────────────┐                                                                                               
       │ ○ 项目初始化                    │                                                                                          
       │ beads-demo-dee             │                                                                                               
       │ blocks:1                   │                                                                                               
       └────────────────────────────┘                                                                                               
           │                                                                                                                        
           ▼                                                                                                                        
                                                                                                                                    
... ...  
∴ Thinking…                                                                                                                         
                                                                                                                                    
  现在bd issues已经创建完成，依赖关系也设置好了。我可以看到：- Layer 0: 项目初始化 (ready)                                          
  - Layer 1: 核心下载逻辑实现                                                                                                       
  - Layer 2: 进度显示实现、断点续传实现                                                                                             
  - Layer 3: CLI入口实现                                                                                                            
  - Layer 4: 编写测试                                                                                                               
                                                                                                                                    
  现在我需要调用ExitPlanMode来请求用户批准计划。                                                                                    
                                                                                                                                    
● 完美！bd issues已创建完成，依赖关系如下：                                                              09:25 AM  glm-4.7          
                                                                                                                                    
  Layer 0 (ready):     项目初始化                                                                                                   
             │                                                                                                                      
             ▼                                                                                                                      
  Layer 1:            核心下载逻辑实现                                                                                              
             │                                                                                                                      
             ├──────────────┐                                                                                                       
             ▼              ▼                                                                                                       
  Layer 2:      进度显示实现    断点续传实现                                                                                        
             │              │                                                                                                       
             └──────┬───────┘                                                                                                       
                    ▼                                                                                                               
  Layer 3:            CLI入口实现                                                                                                   
                    │                                                                                                               
                    ▼                                                                                                               
  Layer 4:            编写测试                                                                                                      
                                                                                                                                    
  现在退出计划模式，准备开始实现：  
```

之后让Claude Code基于bd中的任务逐个进行实现即可。

```
❯ 使用bd获取要完成的任务并实现任务。                                                                                                
                                                                                                                                    
● Bash(bd ready)                                                                                                                    
  ⎿  📋 Ready work (1 issues with no blockers):                                                                                     
                                                                                                                                    
     1. [● P2] [feature] beads-demo-dee: 项目初始化                                                                                 
                                                                                                                                    
● Bash(bd list)                                                                                                                     
  ⎿  ○ beads-demo-4id [● P2] [task] - 编写测试                                                                                      
     ○ beads-demo-4bb [● P2] [feature] - CLI入口实现                                                                                
     ○ beads-demo-fhv [● P2] [feature] - 断点续传实现                                                                               
     … +3 lines (ctrl+o to expand)                                                                                                  
                                                                                                                                    
● Bash(bd show beads-demo-dee)                                                                                                      
  ⎿  ○ beads-demo-dee · 项目初始化   [● P2 · OPEN]                                                                                  
     Owner: Tony Bai · Type: feature                                                                                                
     Created: 2026-01-27 · Updated: 2026-01-27                                                                                      
     … +11 lines (ctrl+o to expand)                                                                                                 
                                                                                                                                    
● Bash(bd update beads-demo-dee --status in_progress)                                                                               
  ⎿  ✓ Updated issue: beads-demo-dee                                                                                                
                                                                                                                                    
● 开始执行项目初始化：                                                                                                              
                                                                                                                                    
● Bash(mkdir -p cmd/downloader pkg/downloader)                                                                                      
  ⎿  (No content)                                                                                                                   
                                                                                                                                    
● Bash(go mod init github.com/tonybai/beads-demo)                                                                                   
  ⎿  go: creating new go.mod: module github.com/tonybai/beads-demo                                                                  
     go: to add module requirements and sums:                                                                                       
        go mod tidy                                                                                                                 
                                                                                                                                    
● Write(README.md)                                                                                                                  
  ⎿  Wrote 43 lines to README.md                                                                                                    
       1 # beads-downloader                                                                                                         
       2                                                                                                                            
       3 A simple concurrent download tool written in Go.                                                                           
       4                                                                                                                            
       5 ## Features                                                                                                                
       6                                                                                                                            
       7 - Concurrent chunked download with automatic HTTP Range detection                                                          
       8 - Resumable downloads (breakpoint resume)                                                                                  
       9 - Progress bar display                                                                                                     
      10 - Graceful exit on Ctrl-C                                                                                                  
     … +33 lines (ctrl+o to expand)                                                                                                 
                                                                                                                                    
● Bash(ls -la && find . -type f -name "*.md" -o -name "go.mod" | head -10)                                                          
  ⎿  total 44                                                                                                                       
     drwxr-xr-x 7 root root 4096 Jan 27 09:34 .                                                                                     
     drwxr-xr-x 3 root root 4096 Jan 27 09:10 ..                                                                                    
     … +14 lines (ctrl+o to expand)                                                                                                 
                                                                                                                                    
● Bash(bd close beads-demo-dee)                                                                                                     
  ⎿  ✓ Closed beads-demo-dee: Closed                                                                                                
                                         
```

如果中途退出再继续会话，可以告诉Claude Code基于bd中的任务继续进行实现即可：

```
❯ 使用bd获取未完成的任务，继续完成                                                                                                  
                                                                                                                                    
● 我来使用 bd 获取未完成的任务。                                                                                                    
                                                                                                                                    
● Bash(bd ready)                                                                                                                    
  ⎿  📋 Ready work (2 issues with no blockers):                                                                                     
                                                                                                                                    
     1. [● P2] [feature] beads-demo-8pf: 进度显示实现                                                                               
     2. [● P2] [feature] beads-demo-fhv: 断点续传实现                                                                               
                                                                                                                                    
● 有2个未完成的任务。让我查看详细信息：                                                                                             
                                                                                                                                    
● Bash(bd show beads-demo-8pf)                                                                                                      
  ⎿  ○ beads-demo-8pf · 进度显示实现   [● P2 · OPEN]                                                                                
     Owner: Tony Bai · Type: feature                                                                                                
     Created: 2026-01-27 · Updated: 2026-01-27                                                                                      
     … +11 lines (ctrl+o to expand)                                                                                                 
                                                                                                                                    
● Bash(bd show beads-demo-fhv)                                                                                                      
  ⎿  Running…                                            
```

**这就是 Beads 的魔力**：它将 AI 的短期记忆（Context Window）转化为项目的长期记忆（Git Database），让 AI 能够像人类一样，跨越时间，有条不紊地推进复杂工程。

## 小结与展望

Beads 不仅仅是一个工具，它代表了一种\*\*"任务即代码" (Tasks as Code)\*\* 的新范式。

在 Gas Town 的宏大构想中，未来的软件开发将是由无数个 AI Agent 协作完成的。而 Beads，正是连接这些 Agent 的神经网络。它让任务的状态、依赖和历史，像代码一样被版本控制、被分发、被协同。

对于正在构建 AI Coding Agent 的开发者来说，集成 Beads 或许是让你的 Agent 拥有"长期记忆"和"战略规划能力"的最短路径。

**项目地址**：github.com/steveyegge/beads

## 附录

为了便于开发者查看当前beads中的issue状态，社区开源了多款图形化的Beads viewer工具，包括网页版的beads-ui、终端TUI版的beads\_viewer等。

这里以TUI版的beads\_viewer为例，简单看看这些viewer的用法。

**安装beads\_viewer：**

```
curl -fsSL "https://raw.githubusercontent.com/Dicklesworthstone/beads_viewer/main/install.sh?$(date +%s)" | bash  
==> Installing bv...  
==> Detected platform: linux_amd64  
==> Checking for pre-built binary...  
==> Latest release: v0.13.0  
==> Selected asset: bv_0.13.0_linux_amd64.tar.gz  
==> Downloading https://github.com/Dicklesworthstone/beads_viewer/releases/download/v0.13.0/bv_0.13.0_linux_amd64.tar.gz...  
==> Extracting...  
==> Installed bv v0.13.0 to /root/.local/bin/bv  
==> Run 'bv' in any beads project to view issues.  
  
Tip: You can also install via Homebrew:  
  brew install dicklesworthstone/tap/bv
```

**使用beads\_viewer查看beads中的issue列表和状态：**

进入beads-demo目录，执行bv命令即可，你将看到类似下面的输出：

从图中，我们可以看到issue列表、优先级、状态，以及处于选择状态下的issue详情（包括依赖图）。

---

**你的 Agent 记忆法**

Beads 用 Git 解决了 Agent 的“长期记忆”问题。你在使用 AI 编程时，是如何管理任务上下文的？是靠手动复制粘贴，还是有什么独门秘籍？你 觉得“任务即代码”这种理念会成为未来的主流吗？

欢迎在评论区分享你的工作流或对 Beads 的看法！让我们一起探索 AI 协作的最佳实践。👇

如果这篇文章为你打开了 AI 任务管理的新视界，别忘了点个【赞】和【在看】，并转发给你的极客朋友，邀请他们一起体验 Beads！

---

**点击下面标题，阅读更多干货！**

-  [Gas Town 启示录：多智能体编排开启 AI 编程工业革命](https://mp.weixin.qq.com/s?__biz=MzIyNzM0MDk0Mg==&mid=2247504858&idx=1&sn=031ff6f85643affc2887e97e535175fb&scene=21#wechat_redirect)

- [【Gemini CLI：重新定义命令行AI开发】01 入门篇 - 为什么你需要一个命令行 AI？](https://mp.weixin.qq.com/s?__biz=MzIyNzM0MDk0Mg==&mid=2247500370&idx=2&sn=e44d9dda16dffdce1e0ecc7e12a10576&scene=21#wechat_redirect)

- [地球上第一个“硅基生命”社交网络moltbook上线：人类禁止发帖，只能围观！](https://mp.weixin.qq.com/s?__biz=MzIyNzM0MDk0Mg==&mid=2247504944&idx=1&sn=932f621499ab4516687d1ed2793b4cab&scene=21#wechat_redirect)

- [Rust 输了？在 AI Agent 的战场上，TypeScript 才是唯一的“神”](https://mp.weixin.qq.com/s?__biz=MzIyNzM0MDk0Mg==&mid=2247504930&idx=1&sn=5d905fd4a77af3642759bae1de9d6f09&scene=21#wechat_redirect)

- [还在当“上下文搬运工”？我写了一门课，帮你重塑AI开发工作流](https://mp.weixin.qq.com/s?__biz=MzIyNzM0MDk0Mg==&mid=2247503448&idx=1&sn=16c0e8fff144d70d0cbe5b5d1859833e&scene=21#wechat_redirect)

- [【AI应用开发第一课】05 赋予 AI “记忆”：Go 中的多轮对话上下文管理](https://mp.weixin.qq.com/s?__biz=MzIyNzM0MDk0Mg==&mid=2247501081&idx=2&sn=507016442d206827ca83d7e4cedf2148&scene=21#wechat_redirect)

- [像构建 Claude Code 一样构建应用：揭秘 Agent-native 架构的 5 大核心原则](https://mp.weixin.qq.com/s?__biz=MzIyNzM0MDk0Mg==&mid=2247504587&idx=1&sn=6274d3b6ff792a74ecf7d4b4abe7db51&scene=21#wechat_redirect)

---

🔥 还在为“复制粘贴喂AI”而烦恼？我的新极客时间专栏 **《[AI原生开发工作流实战](https://mp.weixin.qq.com/s?__biz=MzIyNzM0MDk0Mg==&mid=2247503448&idx=1&sn=16c0e8fff144d70d0cbe5b5d1859833e&scene=21#wechat_redirect)》** 将带你：

* 告别低效，重塑开发范式
* 驾驭AI Agent(Claude Code)，实现工作流自动化
* 从“AI使用者”进化为规范驱动开发的“工作流指挥家”

扫描下方二维码👇，开启你的AI原生开发之旅。