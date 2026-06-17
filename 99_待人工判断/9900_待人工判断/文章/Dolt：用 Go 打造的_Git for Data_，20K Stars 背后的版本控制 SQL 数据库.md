---
title: Dolt：用 Go 打造的"Git for Data"，20K Stars 背后的版本控制 SQL 数据库
author: Go语言中文网
date: 站长 polarisxu站长 polarisxu
url: https://mp.weixin.qq.com/s?__biz=MzAxMTA4Njc0OQ==&mid=2651455909&idx=1&sn=af3acce433d14eb3bbf01b5f6b98254d&chksm=8185ec30c669b2e0462eed07bd30903bcc9ff23a684363773606e8359c51ea09e1914304394c&mpshare=1&scene=24&srcid=05132RPSubmaBEfuwq2YzPIJ&sharer_shareinfo=d066a841bc94d509f0f5e045ddd08295&sharer_shareinfo_first=d066a841bc94d509f0f5e045ddd08295#rd
---

点击上方蓝色“Go语言中文网”关注，每天一起学 Go

> 2026 年 2 月 23 日，Dolt 突破 20,000 GitHub Stars。DoltHub CEO Tim Sehn 发推庆祝，并预告：100,000 Stars，我们来了。

## 如果 Git 和 MySQL 生了一个孩子

这是 Tim Sehn 对 Dolt 的定义——**世界上第一个也是唯一一个版本控制的 SQL 数据库**。

听起来像是营销话术，但让我们看看它实际做了什么：

```
# 安装  
brew install dolthub/tap/dolt  
  
# 创建一个数据库，就像创建一个 Git 仓库  
dolt init  
# Initialized a new Dolt repository in the current directory  
  
# 像操作 MySQL 一样使用  
dolt sql-server  # 启动 SQL 服务器，默认端口 3306  
  
# 连接后执行 SQL  
CREATE TABLE users (  
    id INT PRIMARY KEY,  
    name TEXT,  
    email TEXT  
);  
  
INSERT INTO users VALUES (1, 'Alice', 'alice@example.com');  
  
# 现在，数据的"每一行"都是版本化的  
DOLT_COMMIT("Create users table and initial data", "alice@example.com")
```

关键的不同在后面：

```
# 查看历史  
dolt log  
# commit abc123 (HEAD -> main)  
# Author: Alice <alice@example.com>  
# Date:   2026-02-23 15:42:00 UTC  
#  
#     Create users table and initial data  
  
# 修改一行数据  
UPDATE users SET email='newalice@example.com' WHERE id=1;  
dolt add .  
dolt commit -m "Update Alice's email"  
  
# 对比两个版本的差异——不是 diff 工具，而是 SQL 查询  
dolt diff main HEAD~1  
# +----+----+-------------------+----------------------+  
# |    | id | name              | email                |  
# +----+----+-------------------+----------------------+  
# | -  | 1  | Alice             | alice@example.com    |  
# +----+----+-------------------+----------------------+  
# | +  | 1  | Alice             | newalice@example.com |  
# +----+----+-------------------+----------------------+  
  
# 回滚到任意历史版本  
dolt checkout HEAD~1
```

**这意味着什么？** 你可以对数据库的任意版本创建分支、合并、回滚、推送、拉取——就像 Git 对代码做的事一样。数据也有提交历史。

## 为什么现在值得关注：20K Stars + AI Agent 融合

Dolt 在 2026 年 2 月 23 日突破了 20,000 GitHub Stars。这个里程碑背后的驱动力尤其值得关注：

**Beads 的采用**。

Beads 是一个热门的 AI Agent 记忆工具，17,000 Stars。Beads 使用 Dolt 作为其底层存储引擎——因为 AI Agent 需要在"版本化的数据"上工作：对历史状态做推理、对比不同尝试的差异、回滚到某个"思考分支"。

Tim Sehn 在博客中写道：

> Dolt 目前正处于一个长期的 Stars 飙升期，比预期更早突破了 20K。Beads 的采用似乎是这次 Stars 加速的主要原因。两个社区的融合正在进行中。

这是技术选型影响生态的经典案例：当一个 AI 基础设施（Agent 记忆系统）找到了一个数据结构（版本化 SQL）上的契合点，两者互相促进。

## Prolly Tree：让"快速 Diff"成为可能的魔法数据结构

Dolt 的核心是一个叫做 **Prolly Tree**（Probabilistic B-tree，概率性 B 树）的数据结构。

这个名字是 Noms 团队发明的——Noms 是一个更早的、类似 Git 的文档数据库。**DoltHub 对 Noms 团队的工作抱有极大的敬意："没有他们，Dolt 不会存在。"** Noms 目前已停止维护，Dolt 在其基础上针对 SQL 场景进行了深度优化。

### B-Tree 的问题

传统的 B-Tree 是几乎所有 SQL 数据库（MySQL、PostgreSQL）的存储引擎基础。它在随机读写和有序扫描上表现优秀。

但 B-Tree 有两个致命缺陷：

1. **无法快速 Diff**：比较两个版本的 B-Tree 需要遍历两棵树并对比所有值，复杂度是 O(n)，随着数据量线性增长。
2. **无结构性共享**：对 B-Tree 的写入会改变树的内部结构，因此无法在两个版本之间共享存储。每个版本都要完整存储。

### Prolly Tree 的解决方案

Prolly Tree 是**内容寻址的 B-Tree 变体**。它的核心思想是：给 B-Tree 的每个内部节点都打上基于内容的哈希标签。

构建过程是这样的：

**第一步：排序**将键值对按 key 排序。

**第二步：确定 Chunk 边界**使用滚动哈希（rolling hash）算法决定在哪里"切割"数据块。每个 chunk 平均 4KB。切割位置由哈希函数和 chunk 大小共同决定——这个设计让结果与写入顺序无关。

**第三步：计算内容地址**对每个 chunk 计算 SHA-256 哈希，将 (最大 key, chunk 地址) 作为内部节点的键值，继续构建上一层。

**第四步：递归直到根**直到只剩一个根节点，构建完成。

结果：每个节点都有唯一的内容地址（content address），就像 Git 的每个对象都有 SHA-1 哈希一样。

### 为什么这让"快速 Diff"成为可能

假设你修改了某一行数据：

1. 从根节点开始，比较根节点哈希——**不同**，继续向下
2. 找到哈希不同的子节点——**不同**，继续向下
3. 一直追踪到叶子节点——**找到差异**

如果某个子树的根哈希相同，则整个子树相同，无需进一步遍历。

\*\*Diff 的复杂度从 O(n) 变成了 O(d)\*\*，其中 d 是差异的大小，而不是整棵树的大小。即使数据库有 10 亿行，只要修改了几行，Diff 只需要检查这几个变更点。

### 为什么这让"结构性共享"成为可能

因为存储是**内容寻址的**。相同的哈希意味着相同的内容，就只存储一份。

当你提交一个新版本时：

* 未修改的行：哈希不变，指向同一份存储
* 修改的行：生成新的哈希，创建新存储

这意味着 Dolt 可以**只存储变更的部分**，而不是整个数据库的副本。这与 Git 的对象存储完全一致。

### 历史无关性与 Chunk 大小控制

Prolly Tree 有一个优雅的特性：**历史无关性**（History Independence）。无论你以什么顺序插入、更新、删除数据，最终的树结构是确定的——因为 chunk 边界由内容和大小决定，与顺序无关。

但 Dolt 在 Noms 的基础上做了重要改进：**控制 Chunk 大小的分布**。

Noms 的原始实现会产生几何分布的 chunk 大小——大部分 chunk 很小，少数 chunk 非常大。这在读取时会导致性能问题：大 chunk 的二分查找更慢。

Dolt 使用 Weibull 分布来控制 chunk 大小，使分布更接近正态分布，从而在保证结构共享的同时维持 B-Tree 级别的读写性能。

## 架构全景：Prolly Tree + Commit Graph

Dolt 的存储引擎由两个核心组件构成：

```
Prolly Tree（存储数据）+ Commit Graph（管理版本）= Dolt 存储引擎
```

### Commit Graph

Commit Graph 是 Dolt 的版本控制层。它的工作方式与 Git 完全一致：

* 每个 commit 指向一个根哈希（数据库中所有表的 Prolly Tree 根的组合哈希）
* commits 形成一个有向无环图（DAG）
* 支持分支、合并、回滚
* 每个 commit 包含 author、message、parent commits

### 所有东西都是 Prolly Tree

Dolt 的设计哲学是**一切皆 Prolly Tree**：

* **表数据**：主键 → 行值，存储在 Prolly Tree 的叶子节点
* **表结构**（Schema）：列名、类型、约束，也存储在 Prolly Tree 中
* **数据库整体**：所有表的根哈希组合成一个根哈希

这种一致性保证意味着：即使表结构发生了变化（比如添加一列），新旧 schema 的变更也可以被追踪和 Diff。

## 实战：用 Go 客户端连接 Dolt

Dolt 支持 MySQL 协议，因此任何 MySQL 客户端或 ORM 都可以直接连接。官方也提供了专门的 Go 客户端：

```
import "github.com/dolthub/dolt/go/store/spec"  
import"github.com/dolthub/dolt/go/libraries/doltcore/sqle"  
  
func main() {  
    // 连接到本地 Dolt 数据库  
    db, _ := spec.ForDatabase("file://./my_database")  
    engine, _ := sqle.NewDoltServer(context.Background(), db)  
  
    // 或者连接到远程 DoltHub 数据库  
    // spec.ForRemote("dolthub/dolt-db-name")  
  
    // 执行标准 SQL  
    ctx := context.Background()  
    sqlCtx := sql.NewContext(ctx)  
  
    // 创建表  
    engine.Exec(sqlCtx, "CREATE TABLE users (id INT PRIMARY KEY, name TEXT)")  
  
    // 插入数据  
    engine.Exec(sqlCtx, "INSERT INTO users VALUES (1, 'Alice')")  
  
    // 提交变更  
    engine.Exec(sqlCtx, "SELECT DOLT_COMMIT('-m', 'Add users table')")  
  
    // 读取历史  
    engine.Exec(sqlCtx, "SELECT * FROM dolt_log")  
  
    // Diff 查询  
    engine.Exec(sqlCtx, `  
        SELECT * FROM users AS of 'HEAD~1'  
        UNION ALL  
        SELECT * FROM users  
    `)  
}
```

### 特别的数据表：Dolt 系统表

Dolt 提供了一系列系统表，让你能用 SQL 查询版本控制元数据：

```
-- 查看提交历史  
SELECT * FROM dolt_log;  
  
-- 查看所有分支  
SELECT * FROM doltBranches;  
  
-- 查看某个时间点的数据  
SELECT * FROMusersASOF'2026-02-23 15:00:00';  
  
-- 对比两个分支的差异  
SELECT * FROMusersFROMmain  
UNIONALL  
SELECT * FROMusersFROM feature_branch WHEREidNOTIN (SELECTidFROMmain);  
  
-- 查看未提交的变更  
SELECT * FROM dolt_status;
```

这些系统表的存在意味着：**你的版本控制操作完全可以通过 SQL 完成**，不需要记忆特定的 CLI 命令。

## 应用场景：谁在用 Dolt，为什么

### 场景一：AI Agent 记忆系统

这是目前最热门的应用场景。Beads（17K Stars）使用 Dolt 存储 AI Agent 的"记忆"：

* 每个 Agent 的每个"尝试"是一个 commit
* 可以回溯到任意历史状态
* 可以对不同策略的"记忆"做 Diff
* 多 Agent 协作时，可以 merge 不同的记忆分支

Tim Sehn 在 Data Engineering Podcast 中解释道：

> AI 工作流天然需要版本化的数据。Agent 会尝试不同的策略，产生不同的中间结果。当你想回到某个策略的某个版本时，版本控制数据库就是唯一的选择。

### 场景二：需要数据审计的企业应用

传统数据库做审计需要额外的审计表和触发器。Dolt 直接内置了完整的历史：

```
-- 查某行数据的所有历史变更  
SELECT * FROM users_history WHERE id = 1 ORDER BY commit_time;  
  
-- 查某个时间点谁改了数据  
SELECT commit_hash, author, message, changed_rows  
FROM dolt_log  
WHERE '2026-02-01' <= commit_time AND commit_time < '2026-03-01';
```

### 场景三：数据库 Schema 迁移测试

Dolt 的分支功能让 Schema 迁移变得可测试：

```
# 创建迁移分支  
dolt checkout -b schema-v2  
  
# 在分支上修改 Schema  
dolt sql -q "ALTER TABLE users ADD COLUMN phone TEXT"  
  
# 测试迁移  
# ... 测试验证 ...  
  
# 合并回 main  
dolt checkout main  
dolt merge schema-v2
```

### 场景四：数据众包与协作

DoltHub（dolt 的 GitHub）托管了数千个公开的数据集。任何人都可以 fork、分支、提交 PR：

```
# 克隆一个公开数据集  
dolt clone dolthub/country-codes  
  
# 创建一个改进分支  
dolt checkout -b fix-typo  
  
# 修复数据  
dolt sql -q "UPDATE country_codes SET name='Taiwan' WHERE code='TW'"  
  
# 提交并推送到 DoltHub  
dolt add .  
dolt commit -m "Fix country name"  
dolt push origin fix-typo  
  
# 在 DoltHub 上创建 Pull Request
```

这让"像代码一样协作数据"成为现实。

## Dolt vs 其他方案

| 特性 | Dolt | MySQL + Git Hooks | Liquibase/Flyway | PostgreSQL + pg\_audit |
| --- | --- | --- | --- | --- |
| **内置版本控制** | 是 | 否（需自行实现） | 是（Schema only） | 部分（Schema） |
| **数据历史查询** | SQL 直接查询 | 需额外表 | 需额外表 | 需扩展 |
| **分支与合并** | 原生 Git 风格 | 需额外工具 | 否 | 否 |
| **性能** | 接近 MySQL | N/A | N/A | 接近 PostgreSQL |
| **SQL 兼容性** | MySQL 兼容 | 是 | 是 | 是 |
| **生态集成** | GitHub 风格协作 | 不支持 | 部分 | 部分 |
| **开源** | 是（Apache 2.0） | N/A | 部分 | N/A |

## 技术细节：Go 实现的关键决策

Dolt 完全使用 Go 从零构建，从存储引擎到 SQL 引擎。这带来了几个有意思的技术决策：

### 1. 纯 Go 存储引擎

所有数据结构和 I/O 操作都用 Go 实现，不依赖任何外部 C 库。这让 Dolt 可以轻松在 Linux、macOS、Windows 上运行，也可以在 Docker 和 Kubernetes 中无缝部署。

### 2. 行级版本化

Dolt 实现了真正的**行级版本化**，而不是表级版本化。你可以查询某个时间点的特定行，而不是只能看整个表的历史：

```
SELECT * FROM users_history  
WHERE id = 1 AND commit_hash IN (  
    SELECT commit_hash FROM dolt_log  
    WHERE commit_time BETWEEN '2026-01-01' AND '2026-03-01'  
);
```

### 3. 冲突检测与合并

Dolt 的 Prolly Tree 结构让合并操作变得可预测。当两个分支修改了相同的行时，Dolt 会检测冲突并提供清晰的冲突报告：

```
-- 查看合并冲突  
SELECT * FROM dolt_conflicts.users;
```

### 4. DoltHub：数据的 GitHub

DoltHub 提供免费的公开数据库托管服务，类似 GitHub 托管代码仓库。目前托管了数千个数据集，包括：

* 国家代码（country-codes）
* 公司信息
* 地理数据
* 开源项目元数据

DoltHub 本身也使用 Dolt 存储所有数据——这是"dogfooding"的极致。

## 限制与权衡

没有完美的系统。Dolt 目前有以下限制：

1. **不支持事务的跨分支操作**：每个分支是独立的写上下文，不能在一个事务中同时写两个分支
2. **性能略低于原生 MySQL**：Prolly Tree 的内容寻址和结构性共享带来了一些开销，对于极致性能场景可能有影响
3. **相对年轻的项目**：2019 年发布，虽然 20K Stars 说明已被社区广泛验证，但相比 decades-old 的 MySQL/PostgreSQL，生态系统还在成长
4. **MySQL 兼容，不是 PostgreSQL**：如果你重度使用 PostgreSQL 特有功能，Dolt 可能不适用

## 2026 年的里程碑与未来

Tim Sehn 的目标是 **100,000 Stars**。从 2026 年 2 月的 20K 到 100K，需要 5 倍的增长。但 Beads 的采用已经证明了 AI Agent 场景与 Dolt 的契合度，这可能是一条加速增长的路。

值得关注的进展方向：

* **更完善的 PostgreSQL 兼容层**：扩大受众范围
* **性能优化**：接近甚至追上原生 MySQL/PostgreSQL
* **生态集成**：与主流 ORM、BI 工具、CDC 系统的更深度集成
* **AI Native 数据基础设施**：当 AI Agent 成为主流开发范式，版本化的数据库将成为标配

## 总结

Dolt 是一个真正独特的项目。它不是另一个 SQL 数据库——它是在 SQL 数据库上叠加了 Git 的版本控制能力，并用一个巧妙的 Prolly Tree 数据结构让这一切变得高效。

当 20K Stars 的里程碑与 AI Agent 的崛起交汇，我们看到了一个数据库项目在正确的时间找到了正确的场景。如果你正在构建需要"记忆"和"版本推理"的 AI 应用，或者你的业务需要内置的数据审计与回滚能力，Dolt 值得关注。

```
# 一行安装，开始探索  
brew install dolthub/tap/dolt  
  
# 或者  
curl -L https://github.com/dolthub/dolt/releases/latest/download/dolt-linux-amd64.tar.gz | tar xz
```

**Git for Data，不是口号，是现实。**

---

**参考链接：**

* dolthub/dolt — GitHub 仓库
* DoltHub — 数据托管平台
* Prolly Trees 原理详解 — DoltHub 官方博客
* Dolt 存储引擎架构 — 官方文档
* Branches, Diffs, and SQL: How Dolt Powers Agentic Workflows — Data Engineering Podcast
* Dolt 20K Stars 公告 — DoltHub 博客
* Beads — AI Agent 记忆工具 — 使用 Dolt 的 AI 项目
* Go Prolly Tree 实现 — Go Package 文档

---

**推荐阅读**

* [go-tui：Go 终端 UI 的 React 时刻——用 JSX 风格写声明式 TUI](https://mp.weixin.qq.com/s?__biz=MzAxMTA4Njc0OQ==&mid=2651455875&idx=1&sn=3850d3dad48c948f1853201db5967a11&scene=21#wechat_redirect)

**福利**

我为大家整理了一份从入门到进阶的Go学习资料礼包，包含学习建议：入门看什么，进阶看什么。关注公众号 「polarisxu」，回复 **ebook** 获取；还可以回复「**进群**」，和数万 Gopher 交流学习。