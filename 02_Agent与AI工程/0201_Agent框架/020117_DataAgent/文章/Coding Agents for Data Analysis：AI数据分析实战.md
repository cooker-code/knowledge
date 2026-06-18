---
title: Coding Agents for Data Analysis：AI数据分析实战
author: 人机共生笔记
date: 
url: https://mp.weixin.qq.com/s?__biz=MzY4MzE1OTY1NQ==&mid=2247483905&idx=1&sn=b91aedfa9d6a5995e00be833f94d616d&chksm=f2db952c4966cb3e1bc2acb2990f590d76636b6ed12fdba04f0617e989c9676e9502dc527fbb&mpshare=1&scene=24&srcid=0403Oida4l8IqGVk6T8M1lVR&sharer_shareinfo=eace63da5de5e55684da600fe1c77ecb&sharer_shareinfo_first=eace63da5de5e55684da600fe1c77ecb#rd
---

Simon Willison在NICAR 2026的3小时工作坊精华。

---

# 开篇

# 你想过用AI来帮你分析数据吗？

**什么是Coding Agent数据分析？**

Coding Agent数据分析是让AI代理帮你完成数据探索、清洗、可视化的过程，你只需要描述你想要什么。

Simon Willison在NICAR 2026（数据新闻年会）上开设了3小时工作坊，手把手教你用Claude Code和OpenAI Codex做数据分析。

---

# 第一章：工具准备

## 1.1 必备工具

```
# 数据分析工具栈tools:  Claude_Code:    purpose: "AI编程代理"    strength: "代码理解能力强"  
  OpenAI_Codex:    purpose: "AI编程代理"    strength: "代码生成快"  
  Python:    purpose: "数据分析语言"    strength: "生态丰富"  
  SQLite:    purpose: "轻量数据库"    strength: "简单易用"  
  Datasette:    purpose: "数据可视化"    strength: "一键发布数据"
```

## 1.2 环境配置

```
# 安装Python（推荐使用conda）conda create -n data-analysis python=3.11conda activate data-analysis  
# 安装数据分析库pip install pandas numpy matplotlib jupyter  
# 安装Datasettepip install datasette  
# 验证安装python --version  # 3.11.xpandas --version  # 2.x
```

---

# 第二章：向数据库提问

## 2.1 自然语言转SQL

最神奇的能力之一：用日常语言查询数据库。

```
# 示例：查询SQLite数据库import sqlite3  
# 连接数据库conn = sqlite3.connect("my_database.db")cursor = conn.cursor()  
# 传统方式：写SQLquery = """SELECT date, SUM(sales) FROM orders WHERE status = 'completed'GROUP BY dateORDER BY date"""  
# AI方式：描述你要什么ai_query = """找出所有已完成订单，按日期汇总销售额"""  
# Claude Code会自动转成SQL
```

## 2.2 实战示例

```
# 使用Claude Code的自然语言查询from claude_code import ClaudeCode  
agent = ClaudeCode()  
# 描述你的需求result = agent.run("""读取orders.db数据库，找出2026年销售额最高的前10个产品，按销售额降序排列，并计算占总销售额的百分比。""")  
# Claude Code会：# 1. 探索数据库结构# 2. 编写SQL查询# 3. 执行查询# 4. 分析结果# 5. 输出格式化的报告
```

---

# 第三章：数据探索

## 3.1 自动数据发现

**什么是数据探索？**

数据探索是了解数据的过程，包括数据结构、数据类型、数据质量等。

```
# Claude Code自动探索数据exploration_prompt = """探索orders.db数据库：1. 列出所有表2. 找出每个表的列名和数据类型3. 显示每个表的前5行样本4. 统计缺失值情况"""  
# Claude Code会输出："""表结构：- orders (id, customer_id, date, amount, status)- customers (id, name, email, created_at)  
样本数据：orders表前5行：| id | customer_id | date       | amount | status    ||----|-------------|------------|--------|-----------|| 1  | 101        | 2026-01-01| 299.00 | completed |...  
缺失值：- orders.amount: 0个缺失- orders.status: 2个缺失"""
```

## 3.2 发现数据规律

```
# 让AI帮你发现数据规律pattern_prompt = """分析customers表：1. 找出注册用户最多的月份2. 分析用户地域分布3. 找出活跃用户 vs 流失用户4. 给出用户增长建议"""
```

---

# 第四章：数据清洗

## 4.1 常见数据问题

```
# 数据质量问题quality_issues:  - "缺失值"  - "重复记录"  - "格式不一致"  - "异常值/离群点"  - "编码问题"  
# 示例问题数据messy_data = {    "name": ["John", "john ", "JANE", None],    "phone": ["123-456-7890", "1234567890", "(123) 456-7890"],    "email": ["john@example.com", "john@", None],    "amount": [100, -50, 999999, None]}
```

## 4.2 AI辅助清洗

```
# 使用Claude Code清洗数据cleaning_prompt = """清洗customers表中的数据：  
问题：1. name列有大小写不一致、前后空格2. phone列格式不统一3. email列有无效格式4. amount列有负数和极端异常值  
要求：1. 标准化name为Title Case2. 统一phone格式为 123-456-78903. 验证并标记无效email4. 将负数设为0，极端值标记为异常  
输出清洗后的数据到 cleaned_customers.csv"""  
# Claude Code会：# 1. 识别每个问题# 2. 编写清洗代码# 3. 执行并验证# 4. 导出清洗后的数据
```

## 4.3 实战代码

```
import pandas as pdimport re  
def clean_phone(phone):    """清洗电话号码"""    if pd.isna(phone):        return None  
    # 移除所有非数字    digits = re.sub(r'\D', '', str(phone))  
    # 格式化为 123-456-7890    if len(digits) == 10:        return f"{digits[:3]}-{digits[3:6]}-{digits[6:]}"    return phone  
def clean_name(name):    """清洗姓名"""    if pd.isna(name):        return None    return str(name).strip().title()  
# Claude Code会生成类似的清洗函数
```

---

# 第五章：数据可视化

## 5.1 创建图表

```
# 让Claude Code创建可视化viz_prompt = """基于orders.db数据创建可视化：  
1. 每月销售额趋势图（折线图）2. 产品类别销售占比（饼图）3. Top10客户排名（水平条形图）  
要求：- 使用matplotlib- 添加中文标签- 配色专业- 保存为PNG"""  
# Claude Code会：# 1. 编写可视化代码# 2. 生成图表# 3. 保存为文件
```

## 5.2 交互式可视化

```
# 使用Leaflet创建交互地图map_prompt = """基于trees.db（旧金山树木数据）创建热力图：  
1. 使用Leaflet.js2. 使用Leaflet.heat插件3. 显示树木密度4. 添加OpenStreetMap底图  
输出HTML文件，可用浏览器打开"""  
# Claude Code生成的代码示例：html_code = """<!DOCTYPE html><html><head>    <title>旧金山树木分布热力图</title>    <link rel="stylesheet" href="leaflet.css" />    <script src="leaflet.js"></script>    <script src="leaflet.heat.js"></script></head><body>    <div id="map" style="height: 500px;"></div>    <script>        var map = L.map('map').setView([37.77, -122.43], 13);        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);  
        // 热力图层        var heat = L.heatLayer([            [37.77, -122.41, 0.5],            [37.78, -122.42, 0.8],            // ... 更多点        ], {radius: 25}).addTo(map);    </script></body></html>"""
```

---

# 第六章：数据抓取

## 6.1 网页数据抓取

```
# 使用Claude Code抓取网页数据scraping_prompt = """抓取 https://example.com/books 的书籍信息：  
1. 书名2. 作者3. 价格4. 评分  
保存到books.csv"""  
# Claude Code会：# 1. 分析网页结构# 2. 编写抓取代码# 3. 处理反爬措施# 4. 保存数据
```

## 6.2 实战代码

```
import requestsfrom bs4 import BeautifulSoupimport csv  
def scrape_books(url):    """抓取书籍信息"""    response = requests.get(url)    soup = BeautifulSoup(response.text, 'html.parser')  
    books = []    for item in soup.select('.book-item'):        book = {            'title': item.select_one('.title').text,            'author': item.select_one('.author').text,            'price': item.select_one('.price').text,            'rating': item.select_one('.rating')['data-value']        }        books.append(book)  
    return books  
# Claude Code会生成类似的抓取代码
```

---

# 第七章：工作坊成本

## 7.1 实际花费

```
# NICAR 2026工作坊统计workshop_stats:  duration: "3小时"  participants: "50+人"  total_cost: "$1,150"  # 所有参与者的总费用  per_person: "$23"     # 人均费用  
# 工具使用tool_usage:  OpenAI_Codex: "$23 token费用"  GitHub_Codespaces: "免费提供"
```

## 7.2 效率提升

```
# 传统方式 vs AI辅助comparison:  传统方式:    时间: "3-5小时"    技能: "需要SQL + Python + 可视化"  
  AI辅助:    时间: "30分钟-1小时"    技能: "只需要描述需求"  
  效率提升: "3-5倍"
```

---

# 结尾

用Coding Agent做数据分析：

* 快：3小时工作坊，人均$23搞定
* 简单：不需要精通SQL/Python
* 强大：AI帮你写代码、出图表

如果你每天和数据打交道，试试用Coding Agent辅助。