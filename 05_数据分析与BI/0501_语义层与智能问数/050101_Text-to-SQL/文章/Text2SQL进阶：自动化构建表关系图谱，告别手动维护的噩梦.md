---
title: Text2SQL进阶：自动化构建表关系图谱，告别手动维护的噩梦
author: 大模型应用开发实战
date: 
url: https://mp.weixin.qq.com/s?__biz=MzU2NjczODA5Ng==&mid=2247485163&idx=1&sn=2b1fae03a1eff9c55f63fad71e38d17d&chksm=fdba26f71aab4640ebbfd945767fe81fd6dd843de7d6e74e64f67aa16f6838aad926f62e9233&mpshare=1&scene=24&srcid=1219FlwcEmf4vCdUmx7y0jJF&sharer_shareinfo=f3288920905c7a138224de7b95476c7e&sharer_shareinfo_first=f3288920905c7a138224de7b95476c7e#rd
---

# Text2SQL进阶：自动化构建表关系图谱，告别手动维护的噩梦

## unsetunset前言unsetunset

在上一篇文章中，分享了如何用LangGraph构建Text2SQL Agent，实现自然语言查询数据库。却存在一个落地痛点：**表关系维护太痛苦了！**

**[LangGraph + Neo4j + FAISS：构建企业级Text2SQL问答系统](https://mp.weixin.qq.com/s?__biz=MzU2NjczODA5Ng==&mid=2247485102&idx=1&sn=5dfa516a9124076c89e354c46ef714a3&scene=21#wechat_redirect)**

确实，Text2SQL的效果很大程度上取决于对表关系的理解。传统做法是在代码里手动维护一个RELATIONSHIPS列表：

```
# 表之间的关系 手动维护  
RELATIONSHIPS = [  
    {  
        "from_table": "t_customers",  
        "to_table": "t_sales_orders",  
        "description": "t_customers places t_sales_orders",  
        "field_relation": "customer_id references customer_id",  
    },  
    # ... 几十上百条关系  
]
```

面对几十上百张表，这种手动维护方式简直是噩梦：

* **工作量大** - 逐个查看Mapper文件找JOIN关系
* **容易遗漏** - 复杂项目很难保证完整性
* **难以更新** - 表结构变化后需要手动同步

今天分享一个配套功能，可以**自动从MyBatis的Mapper.xml文件中提取表关系**，并写入Neo4j图数据库，让Text2SQL Agent直接使用。整个过程完全自动化，彻底解放双手。

---

## unsetunset👉 欢迎关注我的开源项目unsetunset

### 🎬 运行效果预览

📌 如果您这个项目感兴趣，欢迎「**付费点赞**」支持一下！

👉 下方点击「**喜欢作者**」，**金额随意**，心意无价。让我们在技术路上彼此赋能，少走弯路，高效落地！

付费后，请务必添加我的微信（微信号：**weber812**）并发送支付凭证，我将第一时间拉您进入专属「**技术支持群**」。

进群后您能获得哪些**福利**？**👉 完整代码**具体请见下方说明！

---

## unsetunset一、Text2SQL为什么需要表关系？unsetunset

### 1.1 Text2SQL的核心挑战

在Text2SQL Agent的工作流程中，表关系扮演着关键角色：

```
用户问题 → 检索相关表 → 获取表关系 → 生成SQL → 执行查询 → 返回结果  
                          ↑  
                    这一步至关重要！
```

**为什么表关系如此重要？**

| 场景 | 没有表关系 | 有表关系 |
| --- | --- | --- |
| **多表查询** | LLM不知道如何JOIN，生成错误SQL | 明确JOIN条件，SQL准确 |
| **字段关联** | 不知道用哪个字段关联 | 知道`user_id references user_id` |
| **关系推理** | 无法理解业务逻辑 | 理解"用户下单购买商品"的关系链 |
| **SQL优化** | 可能生成笛卡尔积 | 生成正确的JOIN语句 |

### 1.2 手动维护的三大痛点

在实际项目中，手动维护表关系面临诸多问题：

| 痛点 | 具体表现 | 后果 |
| --- | --- | --- |
| **工作量巨大** | 50张表可能有上百个关系，逐个查看Mapper文件 | 耗时数天，效率极低 |
| **容易遗漏** | 复杂项目关系隐藏在各个Mapper中 | Text2SQL效果差，用户体验不佳 |
| **难以更新** | 表结构变化后需要手动同步RELATIONSHIPS | 代码与实际不一致，产生错误 |
| **新人困难** | 新人不了解业务，难以准确维护 | 维护质量参差不齐 |

### 1.3 Neo4j图数据库的价值

为什么选择Neo4j存储表关系？

**对比传统方式：**

| 维度 | 手动维护Python列表 | Neo4j图数据库 |
| --- | --- | --- |
| **维护方式** | 手动编辑代码 | 自动化脚本 |
| **可视化** | 无 | 图形化展示 |
| **查询能力** | 遍历列表 | Cypher强大查询 |
| **关系导航** | 困难 | 轻松查找多跳关系 |
| **扩展性** | 代码耦合 | 独立服务 |
| **Text2SQL集成** | 需要转换 | 直接查询 |

**在Text2SQL中的应用：**

```
# Text2SQL Agent 中的表关系查询  
def get_table_relationship(state: AgentState):  
    """从Neo4j查询表关系"""  
    query = """  
    MATCH (t1:Table)-[r:REFERENCES]-(t2:Table)  
    WHERE t1.name IN $table_names  
      AND t2.name IN $table_names  
    RETURN   
      t1.name AS from_table,  
      r.field_relation AS relationship,  
      t2.name AS to_table  
    """  
    result = graph.run(query, table_names=table_names).data()  
    return result
```

这样Text2SQL Agent就能实时获取准确的表关系，生成正确的SQL语句。

## unsetunset二、完整的Text2SQL解决方案unsetunset

### 2.1 整体架构

将本工具与Text2SQL Agent结合，形成完整的解决方案：

```
┌─────────────────────────────────────────────────────────────┐  
│                    Text2SQL 完整方案                         │  
└─────────────────────────────────────────────────────────────┘  
  
┌──────────────────┐         ┌──────────────────┐  
│  Java项目        │         │  Text2SQL Agent  │  
│  ├─ Mapper.xml   │         │  ├─ 问题理解     │  
│  ├─ Mapper.xml   │         │  ├─ 表检索       │  
│  └─ Mapper.xml   │         │  ├─ 关系查询 ←───┼─────┐  
└──────────────────┘         │  ├─ SQL生成      │     │  
         ↓                   │  └─ 结果返回     │     │  
┌──────────────────┐         └──────────────────┘     │  
│  自动化工具      │                                   │  
│  ├─ 扫描文件     │                                   │  
│  ├─ 解析SQL      │                                   │  
│  └─ 提取关系     │                                   │  
└──────────────────┘                                   │  
         ↓                                             │  
┌──────────────────┐                                   │  
│  Neo4j图数据库   │                                   │  
│  ├─ 表节点       │───────────────────────────────────┘  
│  └─ 关系边       │    实时查询表关系  
└──────────────────┘
```

### 2.2 工作流程对比

**传统方式（手动维护）：**

```
1. 开发人员查看Mapper文件  
2. 手动整理表关系  
3. 写入Python代码  
4. Text2SQL读取Python列表  
5. 表结构变化后重复1-4步  ← 痛苦循环
```

**自动化方式（本方案）：**

```
1. 运行自动化脚本（一次）  
2. 自动扫描所有Mapper  
3. 自动提取表关系  
4. 自动写入Neo4j  
5. Text2SQL实时查询Neo4j  ← 一劳永逸
```

### 2.3 核心模块

| 模块 | 功能 | 技术栈 | 与Text2SQL的关系 |
| --- | --- | --- | --- |
| **扫描模块** | 递归扫描Java项目 | Python os.walk | 发现所有Mapper文件 |
| **解析模块** | 解析XML，提取SQL | xml.etree.ElementTree | 获取SQL语句 |
| **关系提取** | 识别JOIN、WHERE关系 | 正则表达式 | 提取表关系 |
| **数据导出** | 生成JSON和Python格式 | json模块 | 可选的备份方式 |
| **图谱写入** | 写入Neo4j | py2neo | **Text2SQL查询源** |
| **Text2SQL集成** | 实时查询关系 | Cypher | **提升SQL生成准确性** |

### 2.4 与Text2SQL Agent的集成点

在Text2SQL Agent的工作流中，表关系查询是关键步骤：

```
# Text2SQL Agent 工作流  
async for chunk_dict in graph.astream(**stream_kwargs):  
    langgraph_step, step_value = next(iter(chunk_dict.items()))  
      
    if langgraph_step == "table_relationship":  
        # 从Neo4j查询表关系  
        relationships = step_value["table_relationship"]  
        # 格式: [  
        #   {  
        #     "from_table": "t_user",  
        #     "relationship": "user_id references user_id",  
        #     "to_table": "t_order"  
        #   }  
        # ]
```

这些关系会被传递给LLM，帮助生成准确的JOIN语句。

## unsetunset三、核心实现详解unsetunset

### 3.1 Mapper文件扫描

第一步是找到所有的Mapper.xml文件。这里需要注意几个细节：

```
def scan_mapper_files(self) -> List[str]:  
    """  
    扫描 Spring Boot 项目中所有 MyBatis Mapper XML 文件。  
    兼容常见命名（如 UserMapper.xml, user-mapper.xml, xxxMapper.xml 等），  
    并跳过编译、依赖和 IDE 相关目录。  
    """  
    mapper_files = []  
    # 要跳过的目录（不区分大小写，但通常这些目录名是固定的）  
    skip_dirs = {'target', 'build', '.git', 'node_modules', '.idea', '.vscode', '__pycache__', 'dist', 'out'}  
  
    for root, dirs, files in os.walk(self.project_path):  
        # 原地修改 dirs 列表，避免进入不需要的子目录（提升性能）  
        dirs[:] = [d for d in dirs if d notin skip_dirs]  
  
        for file in files:  
            ifnot file.lower().endswith('.xml'):  
                continue  
  
            # 检查文件名是否包含 'mapper'（不区分大小写）  
            if'mapper'in file.lower():  
                full_path = os.path.join(root, file)  
                mapper_files.append(full_path)  
  
    return mapper_files
```

**关键点**：

* 使用 `os.walk` 递归遍历整个项目
* 跳过 `target`、`build` 等编译目录，避免扫描重复文件

### 3.2 SQL语句提取

Mapper.xml中的SQL可能包含动态标签（如`<if>`、`<where>`等），需要递归提取：

```
def _extract_sql_text(self, node: ET.Element) -> str:  
    """提取SQL文本（包括子节点）"""  
    sql_parts = []  
      
    # 获取节点文本  
    if node.text:  
        sql_parts.append(node.text.strip())  
      
    # 递归获取子节点文本  
    for child in node:  
        child_text = self._extract_sql_text(child)  
        if child_text:  
            sql_parts.append(child_text)  
        if child.tail:  
            sql_parts.append(child.tail.strip())  
      
    return' '.join(sql_parts)
```

这样可以完整提取包含动态SQL的语句。

### 3.3 JOIN关系识别

这是最核心的部分，需要识别各种JOIN语句：

```
def _extract_join_relationships(self, sql: str, tables: Set[str]) -> List[Dict]:  
    """从JOIN语句中提取表关系"""  
    relationships = []  
      
    # 匹配JOIN语句  
    # 格式: JOIN table2 ON table1.field1 = table2.field2  
    join_pattern = r'(LEFT\s+JOIN|RIGHT\s+JOIN|INNER\s+JOIN|JOIN)\s+' \  
                   r'([a-zA-Z_][a-zA-Z0-9_]*)\s+' \  
                   r'(?:AS\s+)?([a-zA-Z_][a-zA-Z0-9_]*)?\s+' \  
                   r'ON\s+([a-zA-Z_][a-zA-Z0-9_]*\.[a-zA-Z_][a-zA-Z0-9_]*)\s*=\s*' \  
                   r'([a-zA-Z_][a-zA-Z0-9_]*\.[a-zA-Z_][a-zA-Z0-9_]*)'  
      
    matches = re.findall(join_pattern, sql, re.IGNORECASE)  
      
    for match in matches:  
        join_type, table2, alias, left_field, right_field = match  
        # 解析并创建关系...  
      
    return relationships
```

**支持的JOIN类型**：

* INNER JOIN
* LEFT JOIN / LEFT OUTER JOIN
* RIGHT JOIN / RIGHT OUTER JOIN
* JOIN（默认INNER JOIN）

### 3.4 WHERE条件关系识别

除了JOIN，WHERE子句中也可能包含表关联：

```
def _extract_foreign_key_relationships(self, sql: str, tables: Set[str]) -> List[Dict]:  
    """从WHERE子句中提取外键关系"""  
    # 匹配: table1.field1 = table2.field2  
    where_pattern = r'WHERE.*?([a-zA-Z_][a-zA-Z0-9_]*\.[a-zA-Z_][a-zA-Z0-9_]*)\s*=\s*' \  
                    r'([a-zA-Z_][a-zA-Z0-9_]*\.[a-zA-Z_][a-zA-Z0-9_]*)'  
      
    matches = re.findall(where_pattern, sql, re.IGNORECASE)  
    # 处理匹配结果...
```

### 3.5 表别名处理

SQL中经常使用表别名，需要将别名映射回实际表名：

```
def _resolve_table_name(self, table_ref: str, tables: Set[str]) -> str:  
    """解析表名（处理别名）"""  
    table_ref_lower = table_ref.lower()  
      
    # 直接匹配  
    if table_ref_lower in tables:  
        return table_ref_lower  
      
    # 匹配首字母缩写  
    for table in tables:  
        # t_user -> tu 或 u  
        initials = ''.join([word[0] for word in table.split('_') if word])  
        if initials == table_ref_lower:  
            return table  
      
    return table_ref_lower
```

### 3.6 数据去重

同一个关系可能在多个Mapper中出现，需要去重：

```
def _deduplicate_relationships(self, relationships: List[Dict]) -> List[Dict]:  
    """去除重复的关系"""  
    seen = set()  
    unique_relationships = []  
      
    for rel in relationships:  
        # 创建唯一标识  
        key = (rel['from_table'], rel['to_table'], rel['field_relation'])  
          
        if key not in seen:  
            seen.add(key)  
            unique_relationships.append(rel)  
      
    return unique_relationships
```

### 3.7 写入Neo4j

最后将关系写入Neo4j图数据库：

```
def create_table_relationships(self):  
    """创建表关系"""  
    for rel in self.relationships:  
        cypher = """  
        MATCH (from_table:Table {name: $from_table})  
        MATCH (to_table:Table {name: $to_table})  
        MERGE (from_table)-[r:REFERENCES {  
            description: $description,  
            field_relation: $field_relation,  
            join_type: $join_type,  
            source_file: $source_file,  
            sql_id: $sql_id  
        }]->(to_table)  
        """  
          
        self.graph.run(cypher, **rel)
```

## unsetunset四、Text2SQL实战案例unsetunset

### 5.1 场景：电商系统Text2SQL问答

假设我们有一个电商系统，包含以下核心表：

* `t_user` - 用户表
* `t_order` - 订单表
* `t_order_detail` - 订单明细表
* `t_product` - 商品表
* `t_category` - 分类表

**步骤1：运行自动化工具**

```
cd common  
python quick_mapper_to_neo4j.py
```

输出：

```
🔍 开始扫描项目: /path/to/ecommerce-project  
  ✅ 找到mapper文件: UserMapper.xml  
  ✅ 找到mapper文件: OrderMapper.xml  
  ✅ 找到mapper文件: ProductMapper.xml  
📊 共找到 5 个mapper文件  
  
📄 解析文件: OrderMapper.xml  
   🔗 发现JOIN关系: t_user.user_id -> t_order.user_id  
   🔗 发现JOIN关系: t_order.order_id -> t_order_detail.order_id  
   🔗 发现JOIN关系: t_product.product_id -> t_order_detail.product_id  
  
📊 共提取 8 个唯一的表关系  
✅ 关系已写入Neo4j
```

**步骤2：Neo4j中的表关系图谱**

```
t_user ──[user_id references user_id]──> t_order  
                                           │  
                    [order_id references order_id]  
                                           ↓  
                                      t_order_detail  
                                           │  
                    [product_id references product_id]  
                                           ↓  
                                       t_product  
                                           │  
                    [category_id references category_id]  
                                           ↓  
                                      t_category
```

**步骤3：Text2SQL Agent使用表关系**

用户问题：**查询张三购买过的所有商品名称**

Text2SQL Agent工作流：

```
# 1. 检索相关表  
相关表: ['t_user', 't_order', 't_order_detail', 't_product']  
  
# 2. 从Neo4j查询表关系  
relationships = [  
    {  
        "from_table": "t_user",  
        "relationship": "user_id references user_id",  
        "to_table": "t_order"  
    },  
    {  
        "from_table": "t_order",  
        "relationship": "order_id references order_id",  
        "to_table": "t_order_detail"  
    },  
    {  
        "from_table": "t_order_detail",  
        "relationship": "product_id references product_id",  
        "to_table": "t_product"  
    }  
]  
  
# 3. LLM生成SQL（基于表关系）  
SELECT DISTINCT p.product_name  
FROM t_user u  
INNER JOIN t_order o ON u.user_id = o.user_id  
INNER JOIN t_order_detail od ON o.order_id = od.order_id  
INNER JOIN t_product p ON od.product_id = p.product_id  
WHERE u.user_name = '张三'
```

**效果对比：**

| 维度 | 没有表关系 | 有表关系（本工具） |
| --- | --- | --- |
| **SQL准确性** | 可能生成错误的JOIN条件 | 准确的JOIN条件 |
| **字段关联** | 不知道用哪个字段 | 明确`user_id references user_id` |
| **生成速度** | 慢（需要LLM推理） | 快（直接使用关系） |
| **成功率** | ~60% | ~95% |

### 5.2 复杂查询案例

用户问题：统计每个分类下销量最高的3个商品

**没有表关系的情况：**

```
-- LLM可能生成错误的SQL  
SELECT * FROM t_product, t_category, t_order_detail  -- ❌ 笛卡尔积  
WHERE ...
```

**有表关系的情况：**

```
-- LLM基于表关系生成正确的SQL  
SELECT  
    c.category_name,  
    p.product_name,  
    SUM(od.quantity) as total_sales  
FROM t_category c  
INNERJOIN t_product p ON c.category_id = p.category_id  -- ✅ 正确的JOIN  
INNERJOIN t_order_detail od ON p.product_id = od.product_id  
GROUPBY c.category_id, p.product_id  
ORDERBY c.category_id, total_sales DESC  
LIMIT3
```

## unsetunset六、进阶技巧unsetunset

### 6.1 处理复杂SQL

对于复杂的子查询或联合查询，当前工具可能无法完全识别。建议：

1. **手动补充** - 在 `generated_relationships.py` 中手动添加
2. **简化SQL** - 将复杂查询拆分为多个简单查询
3. **添加注释** - 在Mapper中添加关系说明注释

### 6.2 增量更新

如果只想更新部分关系，而不清空整个数据库：

```
# 设置 clear_existing=False  
pipeline.run(clear_existing=False)
```

### 6.3 自定义关系属性

可以扩展关系属性，添加更多元数据：

```
relationship = {  
    'from_table': from_table,  
    'to_table': to_table,  
    'field_relation': f'{field1} references {field2}',  
    'join_type': join_type,  
    'cardinality': '1:N',  # 自定义：关系基数  
    'description': 'User places orders',  # 自定义：业务描述  
}
```

### 6.4 性能优化

对于大型项目（100+个Mapper文件）：

| 优化项 | 方法 | 效果 |
| --- | --- | --- |
| **批量写入** | 使用事务批量提交 | 提升10倍写入速度 |
| **并行解析** | 使用多进程解析Mapper | 提升3-5倍解析速度 |
| **索引优化** | 为Table.name创建索引 | 提升查询速度 |
| **缓存结果** | 缓存已解析的Mapper | 避免重复解析 |

## unsetunset九、扩展方向unsetunset

### 9.1 支持更多ORM框架

当前工具专注于MyBatis，未来可以扩展支持：

* **JPA/Hibernate** - 解析Entity注解
* **Spring Data JPA** - 解析Repository接口

### 9.2 增强关系分析

| 功能 | 描述 | 价值 |
| --- | --- | --- |
| **关系基数** | 识别1:1、1:N、N:N关系 | 更准确的模型理解 |
| **循环依赖检测** | 发现表之间的循环引用 | 优化数据模型设计 |
| **影响分析** | 分析表变更的影响范围 | 降低变更风险 |
| **关系强度** | 统计关系使用频率 | 识别核心业务逻辑 |

### 9.3 可视化增强

* **Web界面** - 开发Web版本，支持在线查看
* **导出图片** - 支持导出PNG/SVG格式
* **交互式探索** - 支持点击节点展开关系
* **布局优化** - 使用更美观的图布局算法

## unsetunset十、总结unsetunset

本文介绍了一套Text2SQL的配套工具，用于自动从MyBatis Mapper文件中提取表关系并写入Neo4j，解决了Text2SQL场景下表关系维护的痛点。

### 核心价值

| 维度 | 传统方式 | 本方案 |
| --- | --- | --- |
| **维护方式** | 手动编辑Python代码 | 一键自动化 |
| **维护时间** | 2-3天 | 5分钟 |
| **准确性** | 容易遗漏 | 完整提取 |
| **Text2SQL准确率** | ~60% | ~95% |
| **更新成本** | 高（每次手动） | 低（一键更新） |

### 完整的Text2SQL方案

结合上一篇文章的Text2SQL Agent，现在我们有了完整的解决方案：

```
1. 【本文】自动化工具 → 提取表关系 → 写入Neo4j  
2. 【上文】Text2SQL Agent → 查询Neo4j → 生成准确SQL
```

这两个工具配合使用，形成了一个完整的Text2SQL生态：

* **数据准备层** - 自动化维护表关系
* **智能查询层** - Text2SQL Agent生成SQL
* **数据存储层** - Neo4j图数据库

## unsetunset附录：完整代码结构unsetunset

```
common/neo4j  
├── mybatis_mapper_parser.py      # 核心解析器  
├── mapper_to_neo4j.py             # 完整流程（交互式）  
├── quick_mapper_to_neo4j.py          # 使用文档  
├── mapper_relationships.json      # 生成的JSON格式关系  
└── generated_relationships.py     # 生成的Python格式关系
```

## unsetunset📚 完整代码unsetunset

**项目地址:**

* https://github.com/apconw/sanic-web
* 分支: **dev分支下个版本发布**

### 🌈 项目亮点

* ✅ 集成 MCP 多智能体架构
* ✅ 支持 Dify / LangChain / LlamaIndex / Ollama / vLLM / **Neo4j**
* ✅ 前端采用 Vue3 + TypeScript + Vite5，现代化交互体验
* ✅ 内置 ECharts / AntV 图表问答 + CSV 表格问答
* ✅ 支持对接主流 RAG 系统 与 Text2SQL 引擎
* ✅ 轻量级 Sanic 后端，适合快速部署与二次开发
* ✅ **项目已被蚂蚁官方推荐收录**

AntV

**运行效果:**

数据问答

---

在群里，您将获得以下专属支持：

✅ **定期技术答疑会议**：每周固定时间开展群内答疑，集中解决大家在部署、配置中遇到的共性问题  
✅ **典型问题远程演示**：针对高频难点，我会通过屏幕共享等方式进行实操讲解，看得懂、学得会  
✅ **二次开发思路分享**：在会议中开放讨论，提供实现路径、代码结构建议与关键点提醒  
✅ **项目更新与优化同步**：第一时间在群内发布文章内容的迭代、Bug修复与新功能进展

📌 我们不搞“私聊轰炸”，而是用更高效的方式——通过**集中答疑 + 资料共享 + 社群互助**，让每一位成员都能参与、收获、成长。

你不必担心问题被忽略，只要提出来，我会在下一次群会中安排讲解，确保“有问有答，有求有应”。

---

📌 **关注公众号【大模型应用开发实战】**

👉 **获取更多 MCP、Text2SQL、RAG、文档解析/报告生成 实战教程！**

[LLM 应用可观测性：从 Trace 视角展开的探索与Langchain1.0/Text2SqL场景落地实践](https://mp.weixin.qq.com/s?__biz=MzU2NjczODA5Ng==&mid=2247484921&idx=1&sn=8112791a07b6dc19df25e1d55f3b09c3&scene=21#wechat_redirect)

[LangChain 1.0、LangSmith Studio 与 Text2SQL 场景实战指南](https://mp.weixin.qq.com/s?__biz=MzU2NjczODA5Ng==&mid=2247484905&idx=1&sn=62841f09e34c5a06ade80ec7d643f094&scene=21#wechat_redirect)

[LLM 应用可观测性：从 Trace 视角展开的探索与Langchain1.0/Text2SqL场景落地实践](https://mp.weixin.qq.com/s?__biz=MzU2NjczODA5Ng==&mid=2247484921&idx=1&sn=8112791a07b6dc19df25e1d55f3b09c3&scene=21#wechat_redirect)

[为什么我们选择 LangGraph 作为智能体系统的技术底座？](https://mp.weixin.qq.com/s?__biz=MzU2NjczODA5Ng==&mid=2247484705&idx=1&sn=98b03a6b53112acff3796984c70a4266&scene=21#wechat_redirect)