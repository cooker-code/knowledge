> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020203_Skill/020203_核心知识点/Skill能力封装与治理边界|Skill能力封装与治理边界]]
---
title: SQL 写到崩溃？30 分钟开发 PostgreSQL Skill，让 AI 接盘
author: 术哥无界
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg4MzcyOTQ2NQ==&mid=2247488934&idx=1&sn=1ed63fcadc51f310dd339080cdf3f374&chksm=ce6a792689f027163869bf9bf598d601dbc657bfbf96342442b344bb28e163a9a52e6feba569&mpshare=1&scene=24&srcid=0312ttoQk2iYrXPJJ1n0E6qU&sharer_shareinfo=ebf7b5228a859bfb65640c342b8bfc6b&sharer_shareinfo_first=ebf7b5228a859bfb65640c342b8bfc6b#rd
---

> 🚩 2026 年「术哥无界」系列实战文档 X 篇原创计划 第 *23* 篇，Skills 最佳实战「2026」系列第 *10* 篇
>
> 大家好，欢迎来到 **术哥无界 | ShugeX ｜ 运维有术**。
>
> 我是**术哥**，一名专注于 AI 编程、AI 智能体、Agent Skills、MCP、云原生、Milvus 向量数据库的**技术实践者与开源布道者**！
>
> **Talk is cheap, let's explore。无界探索，有术而行。**

上周五下午三点，产品经理突然在群里@我：**能不能让AI直接查数据库？现在的流程太慢了，先写SQL，再执行，再复制结果，半天过去了。**

说实话，我当时心里一惊。**让AI直接操作数据库？**这不就是传说中的SQL注入重灾区吗？但转念一想，如果我们能做好安全防护，这个问题确实值得解决。

这就是本篇要讲的故事：**如何从零开发一个PostgreSQL Skill，让AI安全地查询数据库。**

*图1：Skills开发实战系列封面*

## 1. 真实需求场景

### 业务痛点

在我们的日常工作中，经常需要从数据库查询数据。比如：

* 产品经理想看上周新注册的用户数据
* 运营同学需要查询某个活动的参与人数
* 市场部想了解不同地区的用户分布

传统流程是这样的：

1. 打开数据库客户端（比如DBeaver、pgAdmin）
2. 写好SQL查询语句
3. 执行查询
4. 复制结果到Excel或Google Sheets
5. 发送给需求方

一趟下来，少说也得5-10分钟。如果SQL写得有问题，还得重来。

### 期望效果

理想情况应该这样：

```
用户：查一下上周新注册的用户数
AI：正在查询... 找到158个新注册用户，其中北京地区最多（42人）
```

一条消息搞定，3秒钟出结果。但这里有个巨大的安全隐患：**如果用户说"删除所有用户"，AI真的执行了怎么办？**

这就是为什么我们必须谨慎开发这个Skill。

### 需求边界

经过和团队讨论，我们明确了边界：

**可以做的**：

* 只读查询（SELECT）
* 使用参数化查询（防止SQL注入）
* 限制查询数量（防止返回百万行数据）
* 只开放特定表的查询权限

**绝对不能做的**：

* 任何写操作（INSERT/UPDATE/DELETE）
* 禁止表名动态拼接（防止表注入）
* 禁止DROP/CREATE等DDL操作

有了这个边界，心里踏实多了。

## 2. 技术方案设计

### 技术选型：Psycopg2 vs Asyncpg

调研了PostgreSQL的Python连接库后，我们有两个选择：

| 方案 | 优点 | 缺点 | 选择 |
| --- | --- | --- | --- |
| Psycopg2 | 成熟稳定、生态完善、同步简单 | 性能相对较低 | ✅ **选这个** |
| Asyncpg | 性能极高（5x快）、纯异步 | 异步复杂度高、学习曲线陡 | ❌ 不选 |

说实话，我也想用Asyncpg（谁不喜欢快呢？）。但考虑到：

* 这个Skill主要是同步查询场景
* 团队成员对异步编程不熟悉
* Psycopg2文档更丰富，踩坑有解决方案

所以最终选了Psycopg2。如果你和我不一样是异步高手，倒是可以试试Asyncpg。

### 架构设计

```
┌─────────────┐
│   用户输入   │
│ "查询用户数" │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  AI理解意图  │
│ 生成SQL语句 │
└──────┬──────┘
       │
       ▼
┌──────────────────┐
│ PostgreSQL Skill │
│  参数化查询       │
│  JSON格式化       │
└──────┬───────────┘
       │
       ▼
┌──────────────┐
│  PostgreSQL  │
│  数据库      │
└──────────────┘
```

*图2：PostgreSQL Skill架构示意图*

核心设计原则：

1. **只读查询**：禁止任何写操作
2. **参数化查询**：杜绝SQL注入
3. **统一JSON输出**：方便AI解析
4. **完善的错误处理**：异常不能崩溃

### 数据安全设计

这是最重要的部分，我把安全设计单独拿出来讲：

**权限控制**：

```
# 创建只读用户
CREATE ROLE readonly_user WITH PASSWORD 'secure_password';
GRANT CONNECT ON DATABASE mydb TO readonly_user;
GRANT USAGE ON SCHEMA public TO readonly_user;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly_user;
```

**SQL注入防护**：

```
# ❌ 危险做法（绝对禁止）
query = f"SELECT * FROM users WHERE name = '{user_input}'"

# ✅ 安全做法
query = "SELECT * FROM users WHERE name = %s"
cursor.execute(query, (user_input,))
```

**表名防护**：

```
# 动态表名必须用sql.SQL()
from psycopg2 import sql
stmt = sql.SQL("SELECT * FROM {}").format(sql.Identifier(table_name))
```

**查询限制**：

```
# 限制返回行数
query = "SELECT * FROM users LIMIT 1000"
```

*图3：PostgreSQL Skill架构设计*

## 3. 代码实现

### 项目结构

```
postgresql-skill/
├── SKILL.md              # 技能描述文件
├── postgresql_skill.py    # 主实现文件
├── test_skill.py          # 测试文件
├── requirements.txt      # 依赖文件
└── README.md             # 使用文档
```

### 核心代码实现

先上完整代码，再逐段讲解：

```
"""
PostgreSQL Query Skill
安全的PostgreSQL查询技能，支持参数化查询和JSON输出
"""

import json
import psycopg2
from psycopg2 import Error, sql
from psycopg2.extras import RealDictCursor
from datetime import datetime, date
from decimal import Decimal
from typing import Optional, Dict, Any


class PostgreSQLSkill:
    """PostgreSQL查询技能封装"""

    def __init__(self, connection_params: Dict[str, Any]):
        """
        初始化数据库连接参数

        Args:
            connection_params: 数据库连接参数字典
                - user: 用户名
                - password: 密码
                - host: 主机地址
                - port: 端口（可选，默认5432）
                - database: 数据库名
        """
        self.connection_params = connection_params
        self.connection = None

    def connect(self) -> bool:
        """建立数据库连接"""
        try:
            self.connection = psycopg2.connect(**self.connection_params)
            returnTrue
        except Error as e:
            print(f"Connection error: {e}")
            returnFalse

    def disconnect(self):
        """断开数据库连接"""
        if self.connection:
            self.connection.close()

    def query_safe(
        self,
        query: str,
        params: Optional[tuple] = None
    ) -> str:
        """
        安全查询（参数化查询）

        Args:
            query: SQL查询语句
            params: 查询参数元组

        Returns:
            JSON格式字符串
        """
        ifnot self.connection:
            return json.dumps({
                'success': False,
                'error': 'Not connected to database',
                'data': []
            })

        try:
            with self.connection.cursor() as cursor:
                cursor.execute(query, params or ())

                columns = [desc[0] for desc in cursor.description]
                results = []

                for row in cursor.fetchall():
                    processed_row = {}
                    for col, val in zip(columns, row):
                        processed_row[col] = self._serialize_value(val)
                    results.append(processed_row)

                return json.dumps({
                    'success': True,
                    'data': results,
                    'count': len(results)
                }, indent=2, default=str)

        except Error as e:
            return json.dumps({
                'success': False,
                'error': f"Database error: {str(e)}",
                'data': []
            })

    def query_dict(
        self,
        query: str,
        params: Optional[tuple] = None
    ) -> str:
        """
        使用字典游标查询（更方便）

        Args:
            query: SQL查询语句
            params: 查询参数元组

        Returns:
            JSON格式字符串
        """
        ifnot self.connection:
            return json.dumps({
                'success': False,
                'error': 'Not connected to database',
                'data': []
            })

        try:
            with self.connection.cursor(
                cursor_factory=RealDictCursor
            ) as cursor:
                cursor.execute(query, params or ())

                results = []
                for row in cursor.fetchall():
                    processed_row = {}
                    for key, val in row.items():
                        processed_row[key] = self._serialize_value(val)
                    results.append(processed_row)

                return json.dumps({
                    'success': True,
                    'data': results,
                    'count': len(results)
                }, indent=2, default=str)

        except Error as e:
            return json.dumps({
                'success': False,
                'error': f"Database error: {str(e)}",
                'data': []
            })

    def query_dynamic(
        self,
        table_name: str,
        where_clause: str,
        params: Optional[tuple] = None
    ) -> str:
        """
        动态查询（安全地处理表名）

        Args:
            table_name: 表名
            where_clause: WHERE子句
            params: 查询参数

        Returns:
            JSON格式字符串
        """
        ifnot self.connection:
            return json.dumps({
                'success': False,
                'error': 'Not connected to database',
                'data': []
            })

        try:
            # 安全地构建SQL（处理表名）
            stmt = sql.SQL("""
                SELECT * FROM {table_name}
                WHERE {where_clause}
            """).format(
                table_name=sql.Identifier(table_name),
                where_clause=sql.SQL(where_clause)
            )

            with self.connection.cursor() as cursor:
                cursor.execute(stmt, params or ())

                columns = [desc[0] for desc in cursor.description]
                results = []

                for row in cursor.fetchall():
                    processed_row = {}
                    for col, val in zip(columns, row):
                        processed_row[col] = self._serialize_value(val)
                    results.append(processed_row)

                return json.dumps({
                    'success': True,
                    'data': results,
                    'count': len(results)
                }, indent=2, default=str)

        except Error as e:
            return json.dumps({
                'success': False,
                'error': f"Database error: {str(e)}",
                'data': []
            })

    @staticmethod
    def _serialize_value(value: Any) -> Any:
        """
        序列化值为JSON友好的格式

        Args:
            value: 要序列化的值

        Returns:
            序列化后的值
        """
        if value isNone:
            returnNone
        elif isinstance(value, (datetime, date)):
            return value.isoformat()
        elif isinstance(value, Decimal):
            return float(value)
        else:
            return value

    def __enter__(self):
        """上下文管理器支持"""
        self.connect()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        """上下文管理器支持"""
        self.disconnect()


# 使用示例
if __name__ == "__main__":
    # 配置数据库连接
    db_config = {
        'user': 'readonly_user',
        'password': 'your_password',
        'host': '127.0.0.1',
        'port': 5432,
        'database': 'mydb'
    }

    # 使用上下文管理器
    with PostgreSQLSkill(db_config) as pg:
        # 示例1：安全查询
        result = pg.query_safe(
            "SELECT * FROM users WHERE id = %s",
            (1,)
        )
        print("Query result:")
        print(result)

        # 示例2：字典游标查询
        result = pg.query_dict(
            "SELECT * FROM users WHERE name = %s",
            ('Alice',)
        )
        print("\nDict query result:")
        print(result)

        # 示例3：动态查询
        result = pg.query_dynamic(
            'users',
            'email = %s AND status = %s',
            ('alice@example.com', 'active')
        )
        print("\nDynamic query result:")
        print(result)
```

### 核心功能讲解

**1. 连接管理**

```
def connect(self) -> bool:
    """建立数据库连接"""
    try:
        self.connection = psycopg2.connect(**self.connection_params)
        return True
    except Error as e:
        print(f"Connection error: {e}")
        return False
```

这个方法很简单，就是建立数据库连接。但如果连接失败会返回False，调用方需要检查返回值。

我踩过的一个坑：忘记关闭连接。后来用了上下文管理器（`__enter__`和`__exit__`）来自动处理连接的生命周期。

**2. 参数化查询**

```
def query_safe(self, query: str, params: Optional[tuple] = None) -> str:
    """安全查询（参数化查询）"""
    with self.connection.cursor() as cursor:
        cursor.execute(query, params or ())
        # ...
```

这是最关键的方法。注意`params`是元组，如果只有一个参数也要写成`(value,)`。

比如：

```
# ✅ 正确
cursor.execute("SELECT * FROM users WHERE id = %s", (1,))

# ❌ 错误（这会报错）
cursor.execute("SELECT * FROM users WHERE id = %s", (1))
```

我第一次写的时候就犯了这个错，报错说"index out of range"，折腾了半小时才发现是元组的问题。

**3. JSON格式化输出**

```
def _serialize_value(value: Any) -> Any:
    """序列化值为JSON友好的格式"""
    if value is None:
        return None
    elif isinstance(value, (datetime, date)):
        return value.isoformat()
    elif isinstance(value, Decimal):
        return float(value)
    else:
        return value
```

PostgreSQL返回的有些类型JSON不能直接序列化：

* `datetime`和`date` → 转为ISO格式字符串
* `Decimal` → 转为float
* `None` → 保持None

如果你不处理这些，`json.dumps()`会抛`TypeError`。

**4. 动态表名查询**

```
from psycopg2 import sql

stmt = sql.SQL("""
    SELECT * FROM {table_name}
    WHERE {where_clause}
""").format(
    table_name=sql.Identifier(table_name),
    where_clause=sql.SQL(where_clause)
)
```

这招很关键。如果你直接用字符串拼接表名：

```
# ❌ 危险
query = f"SELECT * FROM {table_name}"
```

那就会被SQL注入攻击。正确的做法是用`sql.Identifier()`来处理表名。

*图4：查询执行完整流程*

## 4. 测试验证

### 单元测试

先写一个测试文件`test_skill.py`：

```
import unittest
from unittest.mock import Mock, patch
import json
from datetime import datetime
from postgresql_skill import PostgreSQLSkill


class TestPostgreSQLSkill(unittest.TestCase):
    def setUp(self):
        """设置测试环境"""
        self.mock_connection = Mock()
        self.mock_cursor = Mock()
        self.mock_connection.cursor.return_value.__enter__.return_value = self.mock_cursor
        self.mock_cursor.description = [('id',), ('name',), ('created_at',)]

    @patch('psycopg2.connect')
    def test_query_success(self, mock_connect):
        """测试成功查询"""
        # 设置模拟返回值
        mock_connect.return_value = self.mock_connection
        self.mock_cursor.fetchall.return_value = [
            (1, 'Alice', datetime(2026, 1, 1)),
            (2, 'Bob', datetime(2026, 1, 2)),
        ]

        # 执行查询
        result = PostgreSQLSkill({'user': 'test'}).query_safe(
            "SELECT * FROM users",
        )
        result_dict = json.loads(result)

        # 验证
        self.assertTrue(result_dict['success'])
        self.assertEqual(result_dict['count'], 2)
        self.assertEqual(len(result_dict['data']), 2)

    @patch('psycopg2.connect')
    def test_query_empty_result(self, mock_connect):
        """测试空结果集"""
        mock_connect.return_value = self.mock_connection
        self.mock_cursor.fetchall.return_value = []

        result = PostgreSQLSkill({'user': 'test'}).query_safe(
            "SELECT * FROM users",
        )
        result_dict = json.loads(result)

        self.assertTrue(result_dict['success'])
        self.assertEqual(result_dict['count'], 0)
        self.assertEqual(result_dict['data'], [])

    @patch('psycopg2.connect')
    def test_query_database_error(self, mock_connect):
        """测试数据库错误"""
        from psycopg2 import Error

        # 模拟数据库错误
        mock_connect.side_effect = Error("Connection failed")

        result = PostgreSQLSkill({'user': 'test'}).query_safe(
            "SELECT * FROM users",
        )
        result_dict = json.loads(result)

        self.assertFalse(result_dict['success'])
        self.assertIn('Connection failed', result_dict['error'])


if __name__ == '__main__':
    unittest.main()
```

运行测试：

```
python test_skill.py
```

恭喜你！如果看到`OK`说明测试通过了。

### 集成测试

单元测试只能验证逻辑，集成测试需要真实的数据库连接。

```
import unittest
import psycopg2
import os


class TestPostgreSQLIntegration(unittest.TestCase):
    @classmethod
    def setUpClass(cls):
        """设置测试数据库连接"""
        cls.connection = psycopg2.connect(
            user=os.getenv('TEST_DB_USER', 'postgres'),
            password=os.getenv('TEST_DB_PASSWORD', 'password'),
            host=os.getenv('TEST_DB_HOST', 'localhost'),
            database=os.getenv('TEST_DB_NAME', 'test_db')
        )
        cls.cursor = cls.connection.cursor()

        # 创建测试表
        cls.cursor.execute("""
            CREATE TABLE IF NOT EXISTS test_users (
                id SERIAL PRIMARY KEY,
                name VARCHAR(100) NOT NULL,
                email VARCHAR(100),
                created_at TIMESTAMP DEFAULT NOW()
            )
        """)
        cls.connection.commit()

    @classmethod
    def tearDownClass(cls):
        """清理测试数据"""
        cls.cursor.execute("DROP TABLE IF EXISTS test_users")
        cls.connection.commit()
        cls.cursor.close()
        cls.connection.close()

    def test_insert_and_select(self):
        """测试插入和查询"""
        # 插入测试数据
        self.cursor.execute(
            "INSERT INTO test_users (name, email) VALUES (%s, %s)",
            ('Test User', 'test@example.com')
        )
        self.connection.commit()

        # 查询数据
        self.cursor.execute(
            "SELECT * FROM test_users WHERE name = %s",
            ('Test User',)
        )
        result = self.cursor.fetchone()

        self.assertIsNotNone(result)
        self.assertEqual(result[1], 'Test User')
        self.assertEqual(result[2], 'test@example.com')


if __name__ == '__main__':
    unittest.main()
```

运行集成测试：

```
export TEST_DB_USER=postgres
export TEST_DB_PASSWORD=password
export TEST_DB_HOST=localhost
export TEST_DB_NAME=test_db
python test_skill.py
```

### Mock测试技巧

说实话，Mock测试一开始我很头疼。官方文档写得跟天书一样，我来翻译一下关键点：

**patch在哪里？**

```
# ✅ 正确：patch你import的地方
@patch('psycopg2.connect')
def test_something(self, mock_connect):
    # ...
```

如果你的代码是：

```
import psycopg2
# ...
conn = psycopg2.connect(...)
```

那就patch `'psycopg2.connect'`。

如果你的代码是：

```
from psycopg2 import connect
# ...
conn = connect(...)
```

那就patch `'your_module_name.connect'`。

**设置返回值：**

```
mock_cursor.fetchall.return_value = [(1, 'Alice'), (2, 'Bob')]
```

**模拟异常：**

```
mock_connect.side_effect = Error("Connection failed")
```

## 5. 遇到的问题与解决方案

### 问题1：SQL注入的恐惧

刚开始写的时候，我总是担心SQL注入。查了半天资料，最后找到了3个关键点：

1. **永远使用参数化查询**：

```
# ❌ 危险
query = f"SELECT * FROM users WHERE name = '{user_input}'"

# ✅ 安全
query = "SELECT * FROM users WHERE name = %s"
cursor.execute(query, (user_input,))
```

2. \*\*动态表名用sql.SQL()\*\*：

```
from psycopg2 import sql
stmt = sql.SQL("SELECT * FROM {}").format(sql.Identifier(table_name))
```

3. **限制只读权限**：数据库层面也要防护。

### 问题2：JSON序列化TypeError

报错信息：

```
TypeError: Object of type datetime is not JSON serializable
```

原因：PostgreSQL返回的`datetime`和`Decimal`不能直接转JSON。

解决方案：

```
def _serialize_value(value):
    if isinstance(value, (datetime, date)):
        return value.isoformat()
    elif isinstance(value, Decimal):
        return float(value)
    return value
```

然后在`json.dumps()`时使用：

```
json.dumps(data, default=self._serialize_value)
```

### 问题3：元组的陷阱

第一次写参数化查询时，我写了：

```
cursor.execute("SELECT * FROM users WHERE id = %s", (1))
```

报错：

```
IndexError: tuple index out of range
```

原因：`(1)`不是元组，是括号里的整数1。元组应该是`(1,)`。

正确写法：

```
cursor.execute("SELECT * FROM users WHERE id = %s", (1,))
```

别问我怎么知道的，说多了都是泪。

### 问题4：连接泄漏

一开始忘了关闭连接，后来发现连接数越来越多。

解决方案：使用上下文管理器自动管理连接生命周期。

```
with PostgreSQLSkill(db_config) as pg:
    result = pg.query_safe("SELECT * FROM users")
# 连接自动关闭
```

### 问题5：表名动态拼接的安全性

```
# ❌ 危险
query = f"SELECT * FROM {table_name}"
```

攻击者可以传入`"users; DROP TABLE users; --"`来删除表。

正确做法：

```
from psycopg2 import sql
stmt = sql.SQL("SELECT * FROM {}").format(sql.Identifier(table_name))
```

*图5：错误做法与正确做法对比*

## 6. 总结与方法论

这次开发PostgreSQL Skill，让我对Skills开发有了更深的理解。总结几个核心方法论：

### 1. 安全第一

开发数据库类Skill，**安全永远是第一优先级**。

* 只读权限
* 参数化查询
* SQL注入防护
* 查询数量限制

任何一点疏忽都可能酿成大祸。

### 2. 测试分层

测试不能只依赖一种方式，要分层：

1. **单元测试**：用Mock快速验证逻辑
2. **集成测试**：用真实数据库验证功能
3. **端到端测试**：完整流程验证

每层都有其价值，缺一不可。

### 3. 错误处理

异常处理要细致：

```
try:
    # 数据库操作
except Error as e:
    # 数据库特定错误
    return {'success': False, 'error': str(e)}
except Exception as e:
    # 其他未知错误
    return {'success': False, 'error': f"Unexpected error: {str(e)}"}
```

别让程序崩溃，要让错误可追踪。

### 4. 统一输出格式

Skill的输出要统一：

```
{
    'success': True/False,
    'data': [...],
    'count': 10,
    'error': '...'  # 失败时才有
}
```

这样AI才能稳定解析结果。

### 5. 文档先行

SKILL.md要写清楚：

* 功能特性
* 使用方法
* 安全性说明
* 依赖版本

用户看不懂文档，Skill就没人用。

## 下一步

PostgreSQL Skill只是开始。下一步可以扩展：

* 支持Asyncpg异步查询
* 增加连接池管理
* 添加查询缓存
* 支持复杂查询（JOIN、子查询）

**好啦，谢谢你观看我的文章，如果喜欢可以点赞转发给需要的朋友，我们下一期再见！敬请期待！**

**扫码关注，获取更多 AI 工具的最佳实战经验。不错过每一篇干货！**

联系方式

**加入飞书群，共同交流，共同进步。**