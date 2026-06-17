---
title: 从零到一开发 Text-to-SQL MCP 数据查询服务器
author: Agent笔记
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkyNDczNjEyOA==&mid=2247484575&idx=1&sn=ba2f9c3295b546a507a85c687edc33db&chksm=c0402531f3b19820847d1b59175ea3bd9cd10b0a786546627d89823b39ef161a5beb5075f63a&mpshare=1&scene=24&srcid=0910WFDTMVFLQMAy9Xlx5bCl&sharer_shareinfo=5bd5b3d398d63f9f4ace63b19d3fafb9&sharer_shareinfo_first=5bd5b3d398d63f9f4ace63b19d3fafb9#rd
---

# 引言

在数据驱动的时代，自然语言到 SQL 的转换（Text-to-SQL）技术正变得越来越重要。它允许非技术用户通过自然语言查询数据库，极大降低了数据分析的门槛。本文将详细介绍如何从零开始构建一个基于 FastMCP 框架的 Text-to-SQL 服务器，实现安全、可控的数据库查询服务。

## 项目概述

我们开发的 Text-to-SQL MCP 服务器是一个基于 Model Context Protocol (MCP) 的安全数据库查询服务。该服务器允许通过自然语言生成 SQL 查询，并在严格的权限控制下执行查询操作，确保数据安全的同时提供便捷的数据访问能力。

### 核心功能

* • **数据库连接管理**：安全的 MySQL 数据库连接和查询
* • **权限认证**：基于 RSA 密钥对的 Bearer Token 认证
* • **安全查询**：防止 SQL 注入和危险操作的安全检查
* • **表结构查询**：获取数据库表列表和表结构信息
* • **SQL 执行**：安全的 SQL 查询执行，支持结果限制
* • **健康检查**：服务状态监控

### 技术栈

* • **Python 3.10+**：主要编程语言
* • **FastMCP**：MCP 服务器框架
* • **MySQL**：数据库系统

## 开发环境搭建

### 1. 项目初始化

首先创建项目目录结构：

```
mkdir text-to-sql-mcp  
cd text-to-sql-mcp
```

### 2. 依赖安装

创建 `requirements.txt` 文件：

```
fastmcp==2.10.6  
python-dotenv==1.1.0  
mysql-connector-python==8.2.0  
uvicorn==0.24.0
```

安装依赖：

```
pip install -r requirements.txt
```

### 3. 环境配置

创建 `.env.example` 文件：

```
# 数据库配置  
DB_HOST=localhost  
DB_PORT=3306  
DB_USER=your_username  
DB_PASSWORD=your_password  
DB_NAME=your_database
```

复制并配置环境变量：

```
cp .env.example .env
```

编辑 `.env` 文件，填入实际的数据库连接信息。

### 4. 数据库初始化

创建 `dataset.sql` 文件，包含示例表结构和数据：

```
CREATE DATABASE IF NOT EXISTS your_database;  
USE your_database;  
  
CREATE TABLE `contracts` (  
  `id` int NOT NULL AUTO_INCREMENT,  
  `created_at` timestamp NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',  
  `updated_at` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',  
  `contract_name` varchar(255) NOT NULL COMMENT '合同名称',  
  `client_name` varchar(255) NOT NULL COMMENT '客户名称',  
  `signing_date` varchar(10) NOT NULL COMMENT '签订日期（格式2025-01-01）',  
  `contract_amount` decimal(12,2) NOT NULL COMMENT '签订金额',  
  `status` varchar(10) DEFAULT NULL COMMENT '合同状态 executing（履行中）、completed（已完成）',  
  PRIMARY KEY (`id`),  
  KEY `idx_signing_date` (`signing_date`)  
) ENGINE=InnoDB COMMENT='合同信息表';  
  
INSERT INTO dataset.contracts  
(id, created_at, updated_at, contract_name, client_name, signing_date, contract_amount, status)  
VALUES(1, '2025-07-21 21:15:25', '2025-07-21 21:20:40', '2023年度IT运维服务合同', '北京智云科技有限公司', '2023-01-10', 150000.00, 'executing');  
INSERT INTO dataset.contracts  
(id, created_at, updated_at, contract_name, client_name, signing_date, contract_amount, status)  
VALUES(2, '2025-07-21 21:15:25', '2025-07-21 21:21:09', '电商平台开发项目', '上海锐创网络有限公司', '2023-02-15', 320000.00, 'completed');  
INSERT INTO dataset.contracts  
(id, created_at, updated_at, contract_name, client_name, signing_date, contract_amount, status)  
VALUES(3, '2025-07-21 21:15:25', '2025-07-21 21:20:40', '办公楼装修工程合同', '广州建工集团有限公司', '2023-03-22', 1850000.50, 'executing');  
INSERT INTO dataset.contracts  
(id, created_at, updated_at, contract_name, client_name, signing_date, contract_amount, status)  
VALUES(4, '2025-07-21 21:15:25', '2025-07-21 21:20:40', '品牌全案营销服务协议', '深圳星耀传媒有限公司', '2023-04-05', 680000.00, 'executing');  
INSERT INTO dataset.contracts  
(id, created_at, updated_at, contract_name, client_name, signing_date, contract_amount, status)  
VALUES(5, '2025-07-21 21:15:25', '2025-07-21 21:20:40', '生产设备采购合同', '成都机械制造厂', '2023-05-18', 2450000.00, 'executing');
```

## 核心模块开发

### 1. 认证模块 (auth\_token.py)

认证是安全系统的核心，我们使用 RSA 密钥对实现 JWT Token 认证：

```
from fastmcp.server.auth import BearerAuthProvider  
from fastmcp.server.auth.providers.bearer import RSAKeyPair  
  
def create_auth_components():  
    # 生成RSA密钥对  
    key_pair = RSAKeyPair.generate()  
      
    # 创建访问令牌  
    access_token = key_pair.create_token(  
        subject="58bf32d9-ef25-484f-bb7d-bfc683e5b3eb",  
        issuer="https://fastmcp.example.com",  
        audience="data-analysis-mcp",  
        scopes=["data:read_tables", "data:read_table_data"]  
    )  
    # 模拟生成token  
    print(f'Authorization=Bearer {access_token}')  
      
    # 创建认证提供者  
    auth = BearerAuthProvider(  
        public_key=key_pair.public_key,  
        audience="data-analysis-mcp",  
    )  
      
    return auth
```

**关键点**：

* • 使用 RSA 非对称加密确保 Token 安全
* • 为 Token 设置特定的 audience 防止跨服务使用
* • 通过 scopes 实现细粒度权限控制

### 2. 数据库管理模块 (database.py)

数据库管理模块负责所有与数据库的交互：

```
import os  
import mysql.connector  
from typing import Optional, Dict, Any, List  
  
class DatabaseManager:  
    def __init__(self):  
        self.host = os.getenv('DB_HOST', 'localhost')  
        self.port = int(os.getenv('DB_PORT', 3306))  
        self.user = os.getenv('DB_USER')  
        self.password = os.getenv('DB_PASSWORD')  
        self.database = os.getenv('DB_NAME')  
        self.connection = None  
          
    def connect(self) -> bool:  
        try:  
            self.connection = mysql.connector.connect(  
                host=self.host,  
                port=self.port,  
                user=self.user,  
                password=self.password,  
                database=self.database,  
                charset='utf8mb4'  
            )  
            print("✅ 成功连接到数据库: {}".format(self.database))  
            return True  
        except Exception as e:  
            print("❌ 数据库连接失败: {}".format(str(e)))  
            return False  
      
    def execute_query(self, query: str) -> Optional[List[Dict[str, Any]]]:  
        try:  
            if not self.connection:  
                print("❌ 数据库未连接")  
                return None  
              
            cursor = self.connection.cursor(dictionary=True)  
            cursor.execute(query)  
            results = cursor.fetchall()  
            cursor.close()  
              
            if results:  
                converted_results = [self._convert_row_types(row) for row in results]  
                print("✅ 查询成功，返回 {} 行数据".format(len(converted_results)))  
                return converted_results  
            return []  
              
        except Exception as e:  
            print("❌ 查询执行失败: {}".format(str(e)))  
            return None  
      
    def _convert_row_types(self, row: Dict[str, Any]) -> Dict[str, Any]:  
        """转换数据类型为JSON可序列化的类型"""  
        converted = {}  
        for key, value in row.items():  
            if value is None:  
                converted[key] = None  
            elif isinstance(value, (int, float, str, bool)):  
                converted[key] = value  
            elif hasattr(value, 'isoformat'):  # datetime objects  
                converted[key] = value.isoformat()  
            elif isinstance(value, bytes):  
                converted[key] = value.decode('utf-8', errors='ignore')  
            else:  
                converted[key] = str(value)  
        return converted  
      
    def get_table_info(self, table_name: str) -> Dict[str, Any]:  
        # 获取表结构  
        structure_query = f"DESCRIBE {table_name}"  
        structure_data = self.execute_query(structure_query)  
          
        # 获取表数据样本  
        sample_query = f"SELECT * FROM {table_name} LIMIT 5"  
        sample_data = self.execute_query(sample_query)  
          
        # 获取表统计信息  
        count_query = f"SELECT COUNT(*) as total_rows FROM {table_name}"  
        count_data = self.execute_query(count_query)  
          
        total_rows = count_data[0]['total_rows'] if count_data and len(count_data) > 0 else 0  
          
        return {  
            'structure': structure_data,  
            'sample_data': sample_data,  
            'total_rows': total_rows  
        }
```

**关键点**：

* • 使用连接池管理数据库连接
* • 自动处理数据类型转换，确保 JSON 序列化
* • 提供表结构、样本数据和统计信息的统一接口

### 3. 主服务器模块 (mcp\_server.py)

主服务器模块整合所有功能，提供 MCP 接口：

```
from fastmcp import FastMCP, Context  
from fastmcp.exceptions import ToolError  
from fastmcp.server.dependencies import get_access_token, AccessToken  
from dotenv import load_dotenv  
  
from database import DatabaseManager  
from auth_token import create_auth_components  
  
load_dotenv()  
  
db_manager = None  
auth = create_auth_components()  
mcp = FastMCP(name="data-analysis-mcp", auth=auth)  
  
def initialize_services():  
    global db_manager  
    if db_manager is None:  
        db_manager = DatabaseManager()  
        if not db_manager.connect():  
            raise Exception("数据库连接失败")  
  
def get_validated_access_token() -> AccessToken:  
    try:  
        access_token = get_access_token()  
        if access_token is None:  
            raise ToolError("未提供访问令牌或令牌无效")  
        return access_token  
    except Exception as e:  
        raise ToolError(f"权限验证失败: {str(e)}")  
  
def check_permissions(access_token: AccessToken, required_scopes: list) -> None:  
    if not access_token.scopes:  
        raise ToolError("用户没有任何权限")  
    missing_scopes = [scope for scope in required_scopes if scope not in access_token.scopes]  
    if missing_scopes:  
        raise ToolError(f"权限不足：需要以下权限: {', '.join(missing_scopes)}")  
  
@mcp.tool  
async def get_database_tables(ctx: Context) -> Dict[str, Any]:  
    """获取数据库中所有表的列表"""  
    access_token = get_validated_access_token()  
    check_permissions(access_token, ["data:read_tables"])  
      
    try:  
        initialize_services()  
        tables = db_manager.get_all_tables()  
        return {  
            "user_id": access_token.client_id,  
            "tables": tables,  
            "total_tables": len(tables),  
            "message": f"成功获取 {len(tables)} 个表"  
        }  
    except Exception as e:  
        raise ToolError(f"获取表列表失败: {str(e)}")  
  
@mcp.tool  
async def execute_sql_query(ctx: Context, sql_query: str, limit: int = 100) -> Dict[str, Any]:  
    """执行SQL查询"""  
    access_token = get_validated_access_token()  
    check_permissions(access_token, ["data:read_table_data"])  
      
    # 安全检查：禁止危险操作  
    dangerous_keywords = ['drop', 'delete', 'update', 'insert', 'alter', 'create', 'truncate']  
    if any(keyword in sql_query.lower() for keyword in dangerous_keywords):  
        raise ToolError("安全限制：不允许执行修改数据的操作")  
      
    try:  
        initialize_services()  
        # 添加LIMIT限制  
        if 'limit' not in sql_query.lower():  
            sql_query = f"{sql_query.rstrip(';')} LIMIT {limit}"  
          
        result_data = db_manager.execute_query(sql_query)  
        if result_data is None:  
            raise ToolError("查询执行失败")  
          
        columns = list(result_data[0].keys()) if result_data else []  
        return {  
            "user_id": access_token.client_id,  
            "query": sql_query,  
            "row_count": len(result_data),  
            "columns": columns,  
            "data": result_data,  
            "message": f"查询成功，返回 {len(result_data)} 行数据"  
        }  
    except Exception as e:  
        raise ToolError(f"查询执行失败: {str(e)}")  
  
@mcp.tool  
async def health_check(ctx: Context) -> Dict[str, Any]:  
    """健康检查"""  
    try:  
        initialize_services()  
        return {  
            "status": "healthy",  
            "database_connected": db_manager is not None,  
            "message": "服务运行正常"  
        }  
    except Exception as e:  
        return {  
            "status": "unhealthy",  
            "database_connected": False,  
            "message": f"服务异常: {str(e)}"  
        }
```

**关键点**：

* • 使用 FastMCP 框架快速创建 MCP 服务器
* • 集中处理权限验证逻辑
* • 为每个工具实现详细的安全检查
* • 提供统一的错误处理机制

## 安全机制实现

### 1. 权限认证系统

我们实现了基于 RSA 密钥对的 JWT Token 认证系统：

* • **密钥生成**：使用 RSA 非对称加密生成密钥对
* • **Token 创建**：包含用户 ID、权限范围等信息
* • **权限验证**：每个工具执行前验证用户权限

权限级别：

* • `data:read_tables`：读取表结构权限
* • `data:read_table_data`：读取表数据权限

### 2. 查询安全检查

为防止 SQL 注入和危险操作，我们实现了多层安全检查：

```
# 禁止危险操作  
dangerous_keywords = ['drop', 'delete', 'update', 'insert', 'alter', 'create', 'truncate']  
if any(keyword in sql_query.lower() for keyword in dangerous_keywords):  
    raise ToolError("安全限制：不允许执行修改数据的操作")  
  
# 敏感数据检查  
sensitive_keywords = ['password', 'secret', 'token', 'private', 'confidential']  
is_sensitive = any(keyword in sql_query.lower() for keyword in sensitive_keywords)  
if is_sensitive:  
    check_permissions(access_token, ["data:read_table_data"])  
  
# 自动添加LIMIT限制  
if 'limit' not in sql_query.lower():  
    sql_query = f"{sql_query.rstrip(';')} LIMIT {limit}"
```

### 3. 数据库安全最佳实践

* • **最小权限原则**：数据库用户只授予必要的查询权限
* • **连接加密**：生产环境使用 SSL/TLS 加密数据库连接
* • **查询限制**：自动限制返回行数，防止大量数据泄露
* • **敏感数据保护**：对包含敏感关键词的查询进行额外权限检查

## 服务启动与测试

### 1. 启动服务

运行主服务器文件：

```
python mcp_server.py
```

服务启动后会显示：

```
Authorization=Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...  
🚀 启动MCP数据查询服务器...  
📍 地址: http://127.0.0.1:8000  
📋 可用工具:  
   - health_check: 健康检查  
   - get_user_permissions: 获取用户权限  
   - get_database_tables: 获取数据库表列表  
   - get_table_structure: 获取表结构  
   - execute_sql_query: 执行SQL查询  
   - generate_sql_from_question: 自然语言生成SQL  
   - analyze_query_result: 查询结果分析
```

### 2. 测试工具功能

使用 Cherry Studio 客户端测试各个工具

MCP 服务器端配置：

工具列表：

数据查询流程：

1. 1. 查询数据库有哪些表
2. 2. 根据表信息，判断需要从那个表中查询数据，然后调用工具获取表信息
3. 3. 根据表信息，拼装成SQL发送到服务器执行，得到数据结果。

## 总结

本文详细介绍了从零开始构建 Text-to-SQL MCP 服务器的完整过程，包括环境搭建、核心模块开发、安全机制实现和服务测试。通过 FastMCP 框架，我们快速构建了一个安全、可控的数据库查询服务，实现了自然语言到 SQL 的转换功能。

在实际开发过程中，我们重点关注了安全性和可控性，通过多层安全检查和细粒度权限控制，确保数据安全的同时提供便捷的数据访问能力。该服务器可以作为数据平台的基础组件，为各种应用提供安全的数据查询接口。

未来，我们可以进一步扩展该服务，支持更多数据库类型、增强自然语言理解能力、优化查询性能，并集成更多数据分析功能，打造更强大的数据服务平台。

---

项目源码：https://github.com/tmstack/text-to-sql-mcp-server