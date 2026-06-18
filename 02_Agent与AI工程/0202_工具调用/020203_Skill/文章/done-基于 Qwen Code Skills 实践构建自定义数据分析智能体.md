> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020203_Skill/020203_核心知识点/Skill能力封装与治理边界|Skill能力封装与治理边界]]
---
title: 基于 Qwen Code Skills 实践构建自定义数据分析智能体
author: 狂热JAVA小毕超
date:
url: https://mp.weixin.qq.com/s?__biz=MzUxODg5Mjg3Ng==&mid=2247491788&idx=1&sn=98232e24df5d5691cb9b8138fdd0b7f8&chksm=f86885325db306ec0082fa343cce636b10cc96e2fd36887d3b8352169345655a9e4b58ebf524&mpshare=1&scene=24&srcid=0117owVz6OXthnFr9gZV0JQM&sharer_shareinfo=017cfcd4173b14239dfdbcf22455460a&sharer_shareinfo_first=017cfcd4173b14239dfdbcf22455460a#rd
---

## 一、Qwen Code Skills

`Qwen Code` 是一个面向开发者的智能编码助手，类似于 `Claude Code`，随着  `Claude Code` 推出 `Skills` 功能，  `Qwen Code` 也迅速集成了相关功能，通过 `Skills` 可以将专业知识打包成可发现的功能。每个技能由一个 `SKILL.md` 文件组成，其中包含模型在相关时可以加载的指令，以及可选的支持文件如脚本和模板。

本文将基于 `Qwen Code Skills` 功能构建一个数据库查询`Skill`，并基于该 `Skill` 实现数据方面的问答、分析、以及能够根据数据自动生成分析页面。实验效果如下所示：

其中实验使用数据会在下方实验时说明

技能调用问答：

技能调用、分析生成报表：

生成效果：

在开始实验前，先简单介绍下 `Qwen Code Skills` 的使用过程。

### 1.1 Qwen Code Skills 使用介绍

官方文档地址：

> https://qwenlm.github.io/qwen-code-docs/zh/users/features/skills/

`Skills` 功能目前是是实验阶段，需要更新至最新版本的 `Qwen Code` ，并且启动时使用 `--experimental-skills` 开启 `Skills` 功能，例如：

```
qwen --experimental-skills
```

`Skills` 技能可以放在个人名下或项目名下，个人技能在所有的项目中都可以使用用，技能信息存储在 `~/.qwen/skills/` 中，例如：

```
mkdir -p ~/.qwen/skills/my-skill-name
```

项目技能可以与团队共享，技能信息存储在项目内的 `.qwen/skills/` 中，例如：

```
mkdir -p .qwen/skills/my-skill-name
```

在技能目录下必须要有 `SKILL.md` 描述技能，示例模版如下：

```
---
name: your-skill-name
description: 简要描述此技能的作用以及何时使用它
---
 
# 你的技能名称
 
## 指令
为 Qwen Code 提供清晰、逐步的指导。
```

其中 `name` 和  `description` 不可为空！`Qwen Code` 会验证其中的内容。

额外文件支持，可以根据具体需要添加：

```
my-skill/
├── SKILL.md (必需)
├── reference.md (可选文档)
├── examples.md (可选示例)
├── scripts/
│   └── helper.py (可选工具)
└── templates/
    └── template.txt (可选模板)
```

## 二、构建自定义 Skill 数据分析智能体

### 2.1 使用数据说明

其中分析数据采用数据采用 `COVID-19` 测试案例，包括：美国 `2021-01-28` 号，各个县`county`的新冠疫情累计案例信息，包括确诊病例和死亡病例，数据格式如下所示：

```
date（日期）,county（县）,state（州）,fips（县编码code）,cases（累计确诊病例）,deaths（累计死亡病例）
2021-01-28,Pike,Alabama,01109,2704,35
2021-01-28,Randolph,Alabama,01111,1505,37
2021-01-28,Russell,Alabama,01113,3675,16
2021-01-28,Shelby,Alabama,01117,19878,141
2021-01-28,St. Clair,Alabama,01115,8047,147
2021-01-28,Sumter,Alabama,01119,925,28
2021-01-28,Talladega,Alabama,01121,6711,114
2021-01-28,Tallapoosa,Alabama,01123,3258,112
2021-01-28,Tuscaloosa,Alabama,01125,22083,283
2021-01-28,Walker,Alabama,01127,6105,185
2021-01-28,Washington,Alabama,01129,1454,27
```

下载地址如下：

> https://github.com/BIXUECHAO/covid-19-tes-data/blob/main/us-covid19-counties.csv

创建数据表，并导入上述数据：

```
CREATE TABLE `us_covid19_counties` (
  `date` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '日期',
  `county` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '县',
  `state` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '州',
  `fips` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '县编码code',
  `cases` int DEFAULT NULL COMMENT '累计确诊病例',
  `deaths` int DEFAULT NULL COMMENT '累计死亡病例'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='COVID-19 案例';
```

### 2.2 构建 Skill

在项目的目录下创建 `.qwen/skills/db` 技能包，整体结构如下所示：

```
.qwen/skills/db/
├── SKILL.md
├── scripts/
│   └── run_sql.py
```

其中 `scripts/run_sql.py` 用于动态查询数据库中的数据，接收一个`SQL`参数，通过 `bash` 指令运行。

#### 2.2.1 编写 SKILL.md

主要描述出技能的能力，这里数据查询的场景，可以放入表结构，如果表特别多的情况下，也可以写一个 `scripts` 脚本动态获取。另外为了更好的让大模型处理，最好提供调用脚本的指令示例。

内容示例如下所示，注意 `Python` 路径修改为你电脑的实际路径，如果配置了环境变量可直接改为 `python -u scripts/run_sql.py --sql "{sql}"`：

```
---
name: db-skill
description: COVID-19数据查询, 包括：美国 2021-01-28 号，各个县county的新冠疫情累计案例信息，包括确诊病例和死亡病例
---

# 数据查询

## 数据表结构
``sql
CREATE TABLE `us_covid19_counties` (
  `date` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '日期',
  `county` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '县',
  `state` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '州',
  `fips` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '县编码code',
  `cases` int DEFAULT NULL COMMENT '累计确诊病例',
  `deaths` int DEFAULT NULL COMMENT '累计死亡病例'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='COVID-19 案例';
``

## 数据样例
``data
date（日期）,county（县）,state（州）,fips（县编码code）,cases（累计确诊病例）,deaths（累计死亡病例）
2021-01-28,Pike,Alabama,01109,2704,35
2021-01-28,Randolph,Alabama,01111,1505,37
2021-01-28,Russell,Alabama,01113,3675,16
2021-01-28,Shelby,Alabama,01117,19878,141
2021-01-28,St. Clair,Alabama,01115,8047,147
2021-01-28,Sumter,Alabama,01119,925,28
2021-01-28,Talladega,Alabama,01121,6711,114
2021-01-28,Tallapoosa,Alabama,01123,3258,112
2021-01-28,Tuscaloosa,Alabama,01125,22083,283
2021-01-28,Walker,Alabama,01127,6105,185
2021-01-28,Washington,Alabama,01129,1454,27
``

## 指令
运行辅助脚本：
``bash
# 查询天气
E:\anaconda\conda\envs\openai\python.exe -u scripts/run_sql.py --sql "{sql}"
``

## 示例
- 统计每个州的累计确诊病例
``bash
E:\anaconda\conda\envs\openai\python.exe -u scripts/run_sql.py --sql "SELECT state, COUNT(*) AS total_cases FROM us_covid19_counties GROUP BY state ORDER BY total_cases DESC"
``
```

#### 2.2.2 编写 scripts 脚本

添加一个 `run_sql.py` 脚本，执行用户数据的查询动作，放在  `.qwen/skills/db/scripts` 下，内容如下所示：

```
import json
import argparse
import pymysql


def get_conn():
    return pymysql.connect(
        host="127.0.0.1",
        port=3306,
        database="test3",
        user="root",
        password="root",
        autocommit=True
    )


def query(sql):
    conn = get_conn()
    cursor = conn.cursor()
    cursor.execute(sql)
    columns = [column[0] for column in cursor.description]
    res = list()
    for row in cursor.fetchall():
        res.append(dict(zip(columns, row)))
    cursor.close()
    conn.close()
    return res


def run_sql(sql: str):
    """执行MySQL SQL语句查询数据，一次仅能执行一句SQL！"""
    try:
        if not sql:
            print("请传递需要查询的SQL！")
            return
        fetch = query(sql)
        print(f"数据库查询结果: \n{fetch}")
    except Exception as e:
        print(f"执行SQL错误：{str(e)} ,请修正后重新发起。")

if __name__ == '__main__':
    parser = argparse.ArgumentParser(description="Run a SQL query.")
    parser.add_argument('--sql', type=str, required=True, help='SQL query to execute')
    args = parser.parse_args()
    sql = args.sql
    run_sql(sql)
```

到这 `Skill` 就已经添加完毕了。

### 2.3 使用 Skill

在当前项目目录下唤醒 `Qwen Code`，注意需要添加 `--experimental-skills`：

```
qwen --experimental-skills
```

> 输入：**你当前有哪些Skills**

> 输入：**确诊人数Top10的县是哪几个？**

> 输入：**针对确诊人数前5名的州的信息, 深度分析其各个维度，为我生成一个炫酷的报表页面**

执行过程：

最终生成页面如下：