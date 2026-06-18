> 已吸收至：[[04_OLAP与数据库/0402_关系数据库/040202_PostgreSQL/040202_核心知识点/PostgreSQL权限治理与PostgREST边界|PostgreSQL权限治理与PostgREST边界]]
---
title: PG数据库｜PostgreSQL权限管理深度解析：从授权到合规的全链路实践
author: 安呀智数据坊
date:
url: https://mp.weixin.qq.com/s?__biz=MzkzMDc1MzQ3Mg==&mid=2247489858&idx=1&sn=017d0707667cced9667354eb6b37fbab&chksm=c36ae47180f2717cb189cc0a34b840eed62d536bb9b7a91406fa14243463521d87fcba52d5ad&mpshare=1&scene=24&srcid=1015n0PINtT4wOL4BKSbPxnX&sharer_shareinfo=c221f990fc949f0db1ebdc40ffb35ba1&sharer_shareinfo_first=c221f990fc949f0db1ebdc40ffb35ba1#rd
---

注: 本文为安丫科技刘峰的原创，请尊重知识产权，转发请注明出处，不接受任何抄袭、演绎和未经注明出处的转载。

**导**

**语**

在企业级数据库安全体系中，权限管理是守护数据的第一道防线。

PostgreSQL不仅实现了标准的基于角色访问控制（RBAC），更提供了灵活的授权、撤销、默认权限与行级安全（RLS）机制。

本文将以**实验驱动 + 场景剖析**的方式，带你全面掌握从对象授权到合规落地的全过程，既有理论深度，也有一线实践价值。

**01**

**核心概念：角色、所有权与权限**

1

**角色（Role）**

在PostgreSQL中，没有独立的用户和组。一切皆为角色。

一个角色可以拥有登录权限（此时可视为“用户”），也可以不拥有登录权限，专门用于聚合一组权限（此时可视为“组”）。

角色可以成为其他角色的成员，从而继承其权限。

2

**所有权（Ownership）**

当一个数据库对象（如表、视图、函数）被创建时，创建该对象的角色即成为该对象的所有者。

所有者默认拥有对该对象的所有权限，并且有权授予或撤销其他角色的权限，甚至删除该对象。

3

**权限（Privileges）**

权限是允许一个角色对一个特定对象执行某种操作的许可。

常见的权限包括：

* 表/视图

  SELECT, INSERT, UPDATE, DELETE, TRUNCATE, REFERENCES, TRIGGER
* 模式（Schema）

  USAGE, CREATE
* 函数/过程

  EXECUTE
* 数据库

  CONNECT, CREATE, TEMPORARY

**02**

**基础授权与参数详解：**

**GRANT 与 REVOKE**

GRANT用于授予权限，REVOKE用于撤销权限。

这是权限管理最基础的两个命令。

1

**GRANT 命令详解**

基本语法结构：

```
GRANT { privilege | ALL } ON { object_type } TO { role_name | PUBLIC } [ WITH GRANT OPTION ];
```

* privilege

  SELECT, INSERT等具体权限，或使用ALL授予所有适用权限。
* ON {object\_type}

  作用对象，如TABLE table\_name, SCHEMA schema\_name, ALL TABLES IN SCHEMA schema\_name等。
* TO {role\_name}

  授权对象，可以是具体角色名，或是代表所有角色的PUBLIC。
* WITH GRANT OPTION

   级联授权选项。

  获得此选项的角色，有权将自己拥有的权限再次授予其他角色。

**【**实验一：基础授权与WITH GRANT OPTION

1）准备环境（以超级用户postgres执行）：

```
CREATE ROLE readonly_group;
CREATE ROLE app_user LOGIN PASSWORD 'your_password' IN ROLE readonly_group;
CREATE ROLE manager LOGIN PASSWORD 'manager_pw';
CREATE TABLE employees (id SERIAL PRIMARY KEY, name TEXT);
INSERT INTO employees (name) VALUES ('Alice'), ('Bob');
```

**2）授予权限：**

```
-- 将 SELECT 权限授予“组”角色
GRANT SELECT ON TABLE employees TO readonly_group;
-- 将 SELECT 权限及转授权限授予 manager
GRANT SELECT ON TABLE employees TO manager WITH GRANT OPTION;
```

**3）验证权限继承与转授**

* 使用app\_user连接数据库，执行SELECT \* FROM employees;应成功，因为app\_user是readonly\_group的成员。
* 使用manager连接数据库，执行GRANT SELECT ON TABLE employees TO app\_user;应成功，因为manager拥有GRANT OPTION。

2

**REVOKE 命令详解**

基本语法结构：

```
REVOKE [GRANT OPTION FOR] { privilege | ALL } ON { ... } FROM { ... } [ CASCADE | RESTRICT ];
```

* GRANT OPTION FOR

  仅撤销转授权的能力，保留其自身权限。
* CASCADE vs. RESTRICT (级联行为)

  **1）RESTRICT (默认)**

  如果被撤销权限的角色曾将此权限授予过他人，操作将失败。

  **2）CASCADE**

   不仅撤销目标角色的权限，还会递归地撤销所有由该角色授予出去的相同权限。

**【实验二：CASCADE撤销的实践】**

1）准备环境

确保manager已将SELECT权限授予了app\_user。

2）尝试RESTRICT撤销（将会失败）

```
-- 以超级用户执行
REVOKE SELECT ON TABLE employees FROM manager;
-- 预期输出: ERROR: cannot revoke ... because other objects depend on it
```

3） 使用CASCADE成功撤销

```
REVOKE SELECT ON TABLE employees FROM manager CASCADE;
```

4）验证级联效果

使用app\_user连接，执行SELECT \* FROM employees;现在应失败，因为其权限也被级联撤销了。

**5）解析器接管**

此时，bash解析器进程启动，它接收到脚本文件路径作为参数，然后开始逐行读取、解析并执行脚本文件中的内容。

**03**

**核心对象权限的实验验证**

本章节将通过一系列独立的实验，深入验证针对数据库（DATABASE）、模式（SCHEMA）和表（TABLE）这三个关键对象的权限授予与撤销效果。

##

1

**实验一：数据库（DATABASE）权限验证**

本实验将验证CONNECT权限如何作为访问数据库的第一道门槛。

1）准备环境（以超级用户postgres执行）

```
-- 创建一个新的测试数据库
CREATE DATABASE test_db;

-- 创建一个新角色，默认情况下它没有任何特定数据库的权限
CREATE ROLE db_tester LOGIN PASSWORD 'tester_pw';

-- 默认情况下，CONNECT权限被授予PUBLIC，我们先将其撤销，以建立一个严格受控的环境
REVOKE CONNECT ON DATABASE test_db FROM PUBLIC;
```

2）验证CONNECT权限缺失

现在，尝试使用db\_tester角色连接到test\_db数据库。

在psql命令行中：

```
# 尝试连接（将会失败）
psql -U db_tester -d test_db -h localhost
# 输入密码 'tester_pw'

# --- 预期输出 ---
# psql: error: FATAL:  permission denied for database "test_db"
# DETAIL:  User "db_tester" is not allowed to connect to database "test_db".
```

**实验证明**

没有CONNECT权限，用户甚至无法建立到数据库的连接。

3）授予CONNECT权限并再次验证

返回超级用户会话

```
-- 明确地将CONNECT权限授予db_tester
GRANT CONNECT ON DATABASE test_db TO db_tester;
```

再次尝试连接：

```
# 再次尝试连接
psql -U db_tester -d test_db -h localhost
# 输入密码 'tester_pw'

# --- 输出 ---
# psql (17.4)
# You are now connected to database "test_db" as user "db_tester".
# test_db=>
```

**实验证明**

CONNECT权限是控制数据库级访问的基础，通过REVOKE ... FROM PUBLIC并进行精细化授予，是实现数据库访问控制的第一步。

2

**实验二：模式（SCHEMA）权限验证**

本实验将验证USAGE和CREATE权限在模式级别的重要性。

1）准备环境（在test\_db数据库中，以超级用户执行）

```
-- 确保你已连接到 test_db
\c test_db

-- 创建一个新的模式
CREATE SCHEMA production;

-- 创建一张表并放入该模式
CREATE TABLE production.inventory (
    item_id INT PRIMARY KEY,
    quantity INT
);

-- 授予 db_tester 对该表的 SELECT 权限
GRANT SELECT ON TABLE production.inventory TO db_tester;
```

2）验证USAGE权限缺失

切换到db\_tester用户的psql会话（如果已断开，请重新连接）

```
\c test_db db_tester

-- 尝试查询 inventory 表（将会失败）
SELECT * FROM production.inventory;

# --- 输出 ---
# ERROR:  permission denied for schema production
```

**实验证明**

即使一个角色拥有对模式内某个表的直接权限（如SELECT），如果它没有该模式的USAGE权限，它甚至无法“看到”或访问到该表。USAGE是访问模式内任何对象的前提条件。

3）授予USAGE权限并验证

返回超级用户会话：

```
GRANT USAGE ON SCHEMA production TO db_tester;
```

再次切换到db\_tester会话：

```
-- 再次尝试查询（现在应该成功）
SELECT * FROM production.inventory;
# --- 输出 ---
# (0 rows)
```

4）验证CREATE权限

在db\_tester会话中，尝试在production模式下创建一张新表

```
-- 尝试创建表（将会失败）
CREATE TABLE production.new_table (id INT);

# --- 预期输出 ---
# ERROR:  permission denied for schema production
```

返回超级用户会话，授予CREATE权限：

```
GRANT CREATE ON SCHEMA production TO db_tester;
```

再次切换到db\_tester会话：

```
-- 再次尝试创建表（现在应该成功）
CREATE TABLE production.new_table (id INT);
# --- 预期输出 ---
# CREATE TABLE
```

**实验结论**

USAGE权限是访问模式内对象的“通行证”，而CREATE权限是进行模式内对象创建的“许可证”。

二者相互独立，需要分别授予。

3

**实验三：表（TABLE）权限验证**

本实验将演示针对表的SELECT, INSERT, UPDATE, DELETE等DML权限的精细化控制。

1）准备环境（在test\_db数据库中，以超级用户执行）

```
-- 创建一个只读角色和一个读写角色
CREATE ROLE report_user LOGIN PASSWORD 'report_pw';
CREATE ROLE app_writer LOGIN PASSWORD 'writer_pw';

-- 授予他们访问 production 模式的 USAGE 权限
GRANT USAGE ON SCHEMA production TO report_user, app_writer;

-- 授予 report_user 只读权限
GRANT SELECT ON TABLE production.inventory TO report_user;

-- 授予 app_writer 读写权限
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLE production.inventory TO app_writer;
```

2）验证report\_user（只读）的权限

连接为report\_user：

```
\c test_db report_user

-- SELECT 操作（应该成功）
SELECT * FROM production.inventory;

-- INSERT 操作（失败）
INSERT INTO production.inventory VALUES (1, 100);
# --- 输出 ---
# ERROR:  permission denied for table inventory
```

3）**验证app\_writer（读写）的权限**

连接为app\_writer：

```
\c test_db app_writer

-- SELECT 操作（成功）
SELECT * FROM production.inventory;

-- INSERT 操作（成功）
INSERT INTO production.inventory VALUES (1, 100);
# --- 预期输出 ---
# INSERT 01

-- UPDATE 操作（成功）
UPDATE production.inventory SET quantity = 150 WHERE item_id = 1;
# --- 预期输出 ---
# UPDATE 1

-- DELETE 操作（成功）
DELETE FROM production.inventory WHERE item_id = 1;
# --- 预期输出 ---
# DELETE 1
```

**实验结论**

针对表的DML权限可以被精确地授予不同的角色，从而实现业务层面的读写分离和职责划分。

4

**对象默认权限**

**(ALTER DEFAULT PRIVILEGES)**

GRANT命令只对已存在的对象生效。

为了解决新创建对象的权限自动授予问题，必须使用ALTER DEFAULT PRIVILEGES。

**【实验三：解决新对象的权限问题】**

1）准备环境

```
CREATE SCHEMA app_schema;
CREATE ROLE app_owner LOGIN PASSWORD 'owner_password';
GRANT CREATE, USAGE ON SCHEMA app_schema TO app_owner;
GRANT USAGE ON SCHEMA app_schema TO readonly_group;
```

2）暴露问题

app\_owner在app\_schema中创建新表products后，app\_user（readonly\_group成员）无法查询。

3）解决问题

确保manager已将SELECT权限授予了app\_user。

```
-- 关键步骤：为 app_owner 角色设置默认权限
ALTER DEFAULT PRIVILEGES FOR ROLE app_owner IN SCHEMA app_schema
   GRANT SELECT ON TABLES TO readonly_group;
```

4）验证解决方案

app\_owner再创建一张新表orders，此时app\_user应能自动获得对orders表的SELECT权限。

**实验证明**

ALTER DEFAULT PRIVILEGES是实现自动化权限授予、确保新对象权限一致性的关键工具，是企业级权限管理体系的必备组成部分。

**04**

**高级主题：行级别安全**

**（Row-Level Security, RLS）**

当需要控制用户只能访问表中的特定行时，RLS是最终解决方案。

**【实验四：实现销售人员只能看到自己的销售**

**记录】**

1）准备环境

```
CREATE TABLE sales (id SERIAL PRIMARY KEY, salesperson TEXT, amount NUMERIC);
INSERT INTO sales (salesperson, amount) VALUES ('alice', 100), ('bob', 150), ('alice', 200);
CREATE ROLE alice LOGIN PASSWORD 'alice_pw';
CREATE ROLE bob LOGIN PASSWORD 'bob_pw';
GRANT SELECT ON TABLE sales TO alice, bob;
```

2）创建并启用RLS策略

```
-- 关键步骤1：创建一条策略，规定只有当salesperson列的值等于当前用户名时，该行才可见
CREATE POLICY sales_rls_policy ON sales FOR SELECT USING (salesperson = current_user);

-- 关键步骤2：在表上启用行级别安全
ALTER TABLE sales ENABLE ROW LEVEL SECURITY;
```

3）验证RLS效果

```
#切换到alice用户
test_db=# set role alice ;
test_db=> select * from sales;
 id | salesperson | amount 
----+-------------+--------
  1 | alice       |    100
  3 | alice       |    200
#切换到bob用户
test_db=> set role bob ;
test_db=> select * from sales;
 id | salesperson | amount 
----+-------------+--------
  2 | bob         |    150
```

* 以alice用户登录并查询sales表，应只能看到salesperson为'alice'的两行数据。

* 以bob用户登录并查询，应只能看到salesperson为'bob'的一行数据。

**实验证明**

RLS提供了一种强大而优雅的方式来实现数据行级别的精细化访问控制，是构建多租户应用和满足复杂合规需求的核心技术。

**05**

**权限的查看**

理解如何精确地查看和审计权限，是权限管理工作不可或缺的一环。

1

使用psql元命令快速查看

psql提供了一系列便捷的“反斜杠”命令来快速检查权限。

**【实验五：使用psql元命令】**

* **查看表的权限：\dp [table\_name] 或 \z [table\_name]**

```
test_db=# \dp public.t1 
                                 Access privileges
 Schema | Name | Type  |     Access privileges      | Column privileges | Policies 
--------+------+-------+----------------------------+-------------------+----------
 public | t1   | table | postgres=arwdDxtm/postgres+|                   | 
        |      |       | db_tester=r/postgres       |                   | 
(1 row)
```

* **查看模式的权限：\dn+**

```
test_db=# \dn+ public 
                                       List of schemas
  Name  |       Owner       |           Access privileges            |      Description       
--------+-------------------+----------------------------------------+------------------------
 public | pg_database_owner | pg_database_owner=UC/pg_database_owner+| standard public schema
        |                   | =U/pg_database_owner                   | 
(1 row)
```

* 查看角色信息：\du 或 \dg

```
test_db=# \du postgres 
                             List of roles
 Role name |                         Attributes                         
-----------+------------------------------------------------------------
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS
```

**06**

**结论**

PostgreSQL提供了一套层次分明、功能强大的对象权限管理体系。

其核心在于：

1. 统一的角色模型

   将用户和组统一为角色，通过成员关系实现权限继承，简化了管理。
2. 明确的授权机制

   使用GRANT和REVOKE及其丰富的参数（如WITH GRANT OPTION, CASCADE）对对象进行精确授权。
3. 自动化的默认权限

   通过ALTER DEFAULT PRIVILEGES解决新对象的权限授予问题，是构建健壮权限体系的关键。
4. 精细化的行级控制

   利用行级别安全（RLS）实现数据行级别的多租户隔离和访问控制。
5. 完善的审计工具

   结合psql元命令和系统视图，可以对权限进行全面、深入的检查与审计。

在实践中，应始终遵循最小权限原则，精心设计角色体系，并充分利用PostgreSQL提供的各项高级功能，才能构建一个既安全可靠，又易于维护的数据库环境。

**写在最后**

数据库权限管理从来不是“配置几条命令”那么简单，它更是一种体系化的安全思维。

通过合理设计角色体系、精准使用GRANT/REVOKE、利用默认权限实现自动化、结合RLS实现精细化隔离，DBA与架构师才能真正构建起一个既安全、合规，又高效易维护的PostgreSQL环境。

未来，数据安全与合规只会愈发重要，而权限管理正是其中的核心支点。**掌握今天的技巧，就是赢得明天的安全与信任。**

**作者介绍**

大家好，我是刘峰，安丫科技创始人 & 数据库技术高级讲师，专注于 PostgreSQL、国产数据库运维与迁移、数据库性能优化 等方向。

作为 PG中国分会官方授权讲师、PostgreSQL ACE 讲师认证专家，我长期活跃在一线项目实战中，拥有 10年以上大型数据库管理与优化经验，曾深度参与电信、金融、政务等多个行业的数据库性能调优与迁移项目。

欢迎关注我，一起深入探索数据库的无限可能，技术交流不设限！

📌 觉得有收获的话，记得点赞、收藏、转发支持一下哦，别忘了关注我获取更多数据库干货~

**安呀智数据坊｜我们能做什么**

无论你是业务系统的技术负责人，还是数据部门的第一响应人，我们都能为你提供可靠的支持：

* **数据库类型支持**

  Oracle / MySQL / PostgreSQL / PG / SQL Server 等主流数据库
* **核心服务内容**

  性能优化 / 故障处理 / 数据迁移 / 备份恢复 / 版本升级 / 补丁管理
* **系统性支持**

  深度巡检 / 高可用架构设计 / 应用层兼容评估 / 运维工具集成
* **专项能力补充**

  定制课程培训 / 甲方团队辅导 / 复杂问题协作排查 / 紧急救援支持

📮 如果你有一张删不掉的表、一个跑不动的查询，或者一场说不清的升级风险，欢迎来找我们聊聊。

**END**

**关键词回复(可见相应文章****):**

oracle、mysql、pg、postgresql、sql、性能优化、故障处理、数据迁移、备份恢复、版本升级、补丁管理、深度巡检、解决方案、架构设计......

**小助手**

有任何问题或疑问，欢迎加V进群探讨哦～

\ | /

★

动动你的手指

给**【安呀智数据坊】**加个星标吧~

这样你就不会丢下我啦~

**记得加星标呀！**