---
title: 【python】uv发布pypi实操
author: 梦无矶测开实录
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg2NDA0NTQyMw==&mid=2247489944&idx=1&sn=e49f2576dc204b4916f251cc9d89014c&chksm=cf8d6ca981636dd511483727e4f74904341100ad8097628d15da2b07ab99849b658149b815aa&mpshare=1&scene=24&srcid=01180czIuuw1Hh2CaEulLRnL&sharer_shareinfo=e7bcdd94403affd14b6d1fc5bb3ca352&sharer_shareinfo_first=e7bcdd94403affd14b6d1fc5bb3ca352#rd
---
> 已吸收至：[[09_电脑工具/0901_开发工具与CLI/090101_uv/090101_核心知识点/uv项目入口与发布边界|uv项目入口与发布边界]]


# uv发布pypi实操

项目链接：https://github.com/Lvan826199/mwj\_tools

初始化项目，直接使用uv，如果没有下载uv，可以直接使用pip install uv进行下载

```
# 1. 创建项目目录
mkdir mwj_tools
# 2. 创建符合发布标准的库项目结构（含pyproject.toml），普通项目用 uv init
uv init --lib

# 3. 创建虚拟环境，会在项目目录下创建 .venv 文件夹
uv venv

# 4. 激活虚拟环境
Linux/macOS: source .venv/bin/activate
Windows: .venv\Scripts\activate

#5. 编写代码
在 src/my_tools中开始写代码内容。
```

### 注意事项

1. **包名唯一性**：`mwj-tools` 必须在PyPI上唯一，如果已被占用需要换名
2. **版本管理**：遵循语义化版本规范（semver）
3. **网络问题**：dddd吧？上一篇文章教过大家怎么配置
4. **初次发布**：首次发布后需要等待几分钟才能在PyPI搜索到
5. **令牌安全**：不要将API令牌提交到代码仓库！没人会这么蠢吧？嗯？

初始化生成的pyproject.toml内容如下：

```
[project]
name = "mwj-tools"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
authors = [
    { name = "mwj", email = "88888888@qq.com" }
]
requires-python = ">=3.13"
dependencies = []

[build-system]
requires = ["uv_build>=0.9.18,<0.10.0"]
build-backend = "uv_build"
```

### 📁 项目结构

完整项目目录结构如下：

```
mwj_tools/
├── pyproject.toml          # 项目配置和依赖声明 (uv专用)
├── README.md               # 项目说明文档
├── LICENSE                 # MIT许可证
├── src/
│   └── mwj_tools/                # 主包目录
│       ├── __init__.py     # 包初始化文件
│       ├── datetime_utils.py   # 时间日期处理模块
│       └── table_utils.py      # 表格处理模块
├── tests/                  # 测试目录
│   ├── test_datetime_utils.py
│   └── test_table_utils.py
└── examples/               # 使用示例
    ├── datetime_example.py
    └── table_example.py
```

### 1. 核心配置文件 (`pyproject.toml`)

```
[project]
# 包名：在PyPI上必须是唯一的
name = "mwj-tools"
version = "0.1.0"
description = "梦无矶的python工具库,不定时更新..."
readme = "README.md"
authors = [
    {name = "Your Name", email = "your.email@example.com"}
]
license = {text = "MIT"}
requires-python = ">=3.13"

# 项目依赖 - uv publish会正确处理这些
dependencies = [
    "openpyxl>=3.1.5",
    "pandas>=2.3.3",
    "python-dateutil>=2.9.0.post0",
]

# 可选依赖组
[dependency-groups]
dev = [
    "pytest>=9.0.2",
]

# 包发现配置 - 确保能找到src/mwj_tools
[tool.setuptools.packages.find]
where = ["src"]
include = ["mwj_tools*"]

[tool.setuptools.package-dir]
"" = "src"

[tool.uv.index]
name = "tuna"
url = "https://pypi.tuna.tsinghua.edu.cn/simple"
explicit = true

# 构建系统 - uv publish依赖于此
[build-system]
requires = ["hatchling"]  # uv publish推荐使用hatchling
build-backend = "hatchling.core"
# requires = ["uv_build>=0.9.18,<0.10.0"]
# build-backend = "uv_build"

# 项目URLs (可选但推荐)
[project.urls]
Homepage = "https://github.com/yourusername/mwj-tools"
Repository = "https://github.com/yourusername/mwj-tools.git"
"Issue Tracker" = "https://github.com/yourusername/mwj-tools/issues"

# 项目分类 (帮助PyPI分类)
[project.classifiers]
"Programming Language :: Python :: 3" = "详细说明见README.md"
"License :: OSI Approved :: MIT License" = ""
"Operating System :: OS Independent" = ""
```

使用`uv sync`进行依赖包下载，下载完成后开始代码编写。

如果没有写在.toml文件里，需要自己一个个下载依赖包的话，那使用`uv add`，这两个命令的区别参考上一篇文章：

```
uv add python-dateutil
uv add pandas
uv add openpyxl
```

### 2. 时间日期处理模块代码demo

`src/mwj_tools/datetime_utils.py`

```
"""
@Time : 2025/12/25 16:34
@Email : Lvan826199@163.com
@公众号 : 梦无矶测开实录
@File : table_utils.py
"""
__author__ = "梦无矶小仔"
"""
日期时间通用处理工具模块
提供时间计算、转换、格式化等常用功能
"""
from datetime import datetime, timedelta, date
from dateutil.relativedelta import relativedelta
import time
from typing import Union, Tuple, Optional

class DateTimeUtils:
"""日期时间处理工具类"""

    @staticmethod
    def now(fmt: str = None) -> Union[datetime, str]:
    """
        获取当前时间

        Args:
            fmt: 格式化字符串，如'%Y-%m-%d %H:%M:%S'

        Returns:
            格式化后的时间字符串或datetime对象
        """
        current = datetime.now()
        return current.strftime(fmt) if fmt else current

    @staticmethod
    def add_time(
        base_time: Union[datetime, str] = None,
        days: int = 0,
        hours: int = 0,
        minutes: int = 0,
        months: int = 0
    ) -> datetime:
    """
        时间加减计算

        Args:
            base_time: 基准时间，默认为当前时间
            days: 加减天数
            hours: 加减小时数
            minutes: 加减分钟数
            months: 加减月数

        Returns:
            计算后的datetime对象
        """
        if base_time isNone:
            base_time = datetime.now()
        elif isinstance(base_time, str):
            base_time = datetime.fromisoformat(base_time)

        if months:
        # 使用relativedelta处理月份加减（考虑不同月份天数）
            result = base_time + relativedelta(months=months)
            result += timedelta(days=days, hours=hours, minutes=minutes)
        else:
            result = base_time + timedelta(days=days, hours=hours, minutes=minutes)

        return result

    @staticmethod
    def to_timestamp(dt: Union[datetime, str] = None) -> float:
    """
        将datetime转换为时间戳

        Args:
            dt: datetime对象或时间字符串

        Returns:
            时间戳（浮点数，单位秒）
        """
        if dt is None:
            dt = datetime.now()
        elif isinstance(dt, str):
            dt = datetime.fromisoformat(dt)

        return dt.timestamp()

    # 后面部分参考源码
```

### 3. 表格处理模块代码demo

```
src/mwj_tools/table_utils.py
# -*- coding: utf-8 -*-
"""
@Time : 2025/12/25 16:34
@Email : Lvan826199@163.com
@公众号 : 梦无矶测开实录
@File : table_utils.py
"""
__author__ = "梦无矶小仔"
"""
表格数据处理工具模块
提供数据读取、清洗、转换、分析等常用功能
"""
import pandas as pd
import numpy as np
from typing import Union, List, Dict, Any, Optional
import json
import csv
from pathlib import Path


class TableUtils:
    """表格数据处理工具类"""

    @staticmethod
    def read_table(
            filepath: str,
            file_type: str = None
    ) -> pd.DataFrame:
        """
        读取表格文件

        Args:
            filepath: 文件路径
            file_type: 文件类型，可选：'csv', 'excel', 'json'
                        如为None则根据扩展名自动判断

        Returns:
            pandas DataFrame对象
        """
        if file_type is None:
            # 根据文件扩展名判断类型
            ext = Path(filepath).suffix.lower()
            if ext in ['.csv', '.tsv']:
                file_type = 'csv'
            elif ext in ['.xlsx', '.xls']:
                file_type = 'excel'
            elif ext == '.json':
                file_type = 'json'
            else:
                raise ValueError(f"不支持的文件格式: {ext}")

        readers = {
            'csv': pd.read_csv,
            'excel': pd.read_excel,
            'json': pd.read_json
        }

        if file_type not in readers:
            raise ValueError(f"不支持的文件类型: {file_type}")

        return readers[file_type](filepath)

    @staticmethod
    def save_table(
            df: pd.DataFrame,
            filepath: str,
            file_type: str = None
    ) -> None:
        """
        保存表格到文件

        Args:
            df: pandas DataFrame
            filepath: 保存路径
            file_type: 文件类型，自动判断或指定
        """
        if file_type is None:
            ext = Path(filepath).suffix.lower()
            if ext in ['.csv', '.tsv']:
                file_type = 'csv'
            elif ext in ['.xlsx', '.xls']:
                file_type = 'excel'
            elif ext == '.json':
                file_type = 'json'
            else:
                file_type = 'csv'  # 默认保存为CSV

        savers = {
            'csv': lambda: df.to_csv(filepath, index=False),
            'excel': lambda: df.to_excel(filepath, index=False),
            'json': lambda: df.to_json(filepath, orient='records', indent=2)
        }

        if file_type not in savers:
            raise ValueError(f"不支持的文件类型: {file_type}")

        savers[file_type]()

    #后面部分参考源码
```

### 4. 包初始化文件

`src/mwj_tools/__init__.py`

```
"""
MWJ Tools - 实用的Python工具库
"""

from .datetime_utils import DateTimeUtils
from .table_utils import TableUtils

__version__ = "0.1.0"
__author__ = "梦无矶"
__email__ = "your.email@163.com"

__all__ = ['DateTimeUtils', 'TableUtils']
```

### 5. 时间日期示例代码

`examples/datetime_example.py`

```
#!/usr/bin/env python3
"""
日期时间工具使用示例
"""
from mwj_tools import DateTimeUtils

def datetime_examples():
"""日期时间功能演示"""

    # 1. 获取当前时间
    print("当前时间:", DateTimeUtils.now())
    print("格式化当前时间:", DateTimeUtils.now("%Y-%m-%d %H:%M:%S"))

    # 2. 时间加减
    print("\n--- 时间加减 ---")
    now = DateTimeUtils.now()
    print("当前时间:", now)
    print("一天后:", DateTimeUtils.add_time(now, days=1))
    print("一个月后:", DateTimeUtils.add_time(now, months=1))
    print("3天5小时后:", DateTimeUtils.add_time(now, days=3, hours=5))

    # 3. 时间戳转换
    print("\n--- 时间戳 ---")
    timestamp = DateTimeUtils.to_timestamp()
    print("当前时间戳:", timestamp)
    print("时间戳转时间:", DateTimeUtils.from_timestamp(timestamp))

    # 4. 时间差计算
    print("\n--- 时间差 ---")
    time1 = "2024-01-01 10:00:00"
    time2 = "2024-01-02 14:30:00"
    diff_hours = DateTimeUtils.time_difference(time1, time2, 'hours')
    print(f"时间差: {diff_hours:.2f} 小时")

    # 5. 未来日期计算
    print("\n--- 未来日期 ---")
    print("100天后是:", DateTimeUtils.future_date(100))
    print("从2024-06-01起180天后是:", 
          DateTimeUtils.future_date(180, "2024-06-01"))

    # 6. 周判断
    print("\n--- 周判断 ---")
    print("今天是周末吗?", DateTimeUtils.is_weekend())
    week_start, week_end = DateTimeUtils.get_week_range()
    print(f"本周范围: {week_start} 到 {week_end}")

if __name__ == "__main__":
    datetime_examples()
```

示列结果展示

```
当前时间: 2025-12-25 16:54:50.371661
格式化当前时间: 2025-12-25 16:54:50

--- 时间加减 ---
当前时间: 2025-12-25 16:54:50.371708
一天后: 2025-12-26 16:54:50.371708
一个月后: 2026-01-25 16:54:50.371708
3天5小时后: 2025-12-28 21:54:50.371708

--- 时间戳 ---
当前时间戳: 1766652890.371786
时间戳转时间: 2025-12-25 16:54:50.371786

--- 时间差 ---
时间差: 28.50 小时

--- 未来日期 ---
100天后是: 2026-04-04
从2024-06-01起180天后是: 2024-11-28

--- 周判断 ---
今天是周末吗? False
本周范围: 2025-12-22 到 2025-12-28

Process finished with exit code 0
```

### 6. 表格处理示例代码

`examples/table_example.py`

```
#!/usr/bin/env python3
"""
表格工具使用示例
"""
import pandas as pd
from mwj_tools import TableUtils

def table_examples():
"""表格处理功能演示"""

    # 创建示例数据
    data = {
    '姓名': ['张三', '李四', '王五', '赵六', '钱七'],
    '年龄': [25, 30, 35, 28, 32],
    '部门': ['技术部', '市场部', '技术部', '人事部', '市场部'],
    '工资': [8000, 7000, 9000, 6000, 7500],
    '入职日期': ['2022-01-15', '2021-03-20', '2020-08-10', '2023-02-28', '2021-11-05']
    }

    df = pd.DataFrame(data)
    print("原始数据:")
    print(df)

    # 1. 数据筛选
    print("\n--- 数据筛选 ---")
    # 筛选技术部员工
    tech_dept = TableUtils.filter_data(df, {'部门': '技术部'})
    print("技术部员工:")
    print(tech_dept)

    # 筛选年龄大于30的员工
    age_gt_30 = TableUtils.filter_data(df, {'年龄': ('>', 30)})
    print("\n年龄大于30的员工:")
    print(age_gt_30)

    # 2. 数据聚合
    print("\n--- 数据聚合 ---")
    # 按部门统计平均工资和人数
    dept_stats = TableUtils.aggregate_data(
        df,
        group_by='部门',
        aggregations={'工资': 'mean', '姓名': 'count'}
    )
    print("部门统计:")
    print(dept_stats)

    # 3. 数据清洗
    print("\n--- 数据清洗 ---")
    # 假设数据中有缺失值
    df_with_na = df.copy()
    df_with_na.loc[2, '工资'] = None
    print("包含缺失值的数据:")
    print(df_with_na)

    df_cleaned = TableUtils.clean_data(df_with_na, strategy='fill', fill_value=0)
    print("\n清洗后的数据:")
    print(df_cleaned)

    # 4. 数据描述
    print("\n--- 数据描述 ---")
    stats = TableUtils.describe_data(df)
    print(f"数据形状: {stats['shape']}")
    print(f"缺失值统计: {stats['missing_values']}")

    # 5. 数据透视表
    print("\n--- 数据透视表 ---")
    # 按部门和年龄分组查看工资
    pivot_df = TableUtils.pivot_table(
        df,
        index='部门',
        columns='年龄',
        values='工资',
        aggfunc='mean'
    )
    print("工资透视表:")
    print(pivot_df)

    # 6. 文件操作示例
    print("\n--- 文件操作 ---")
    # 保存为CSV
    TableUtils.save_table(df, 'example_data.csv')
    print("数据已保存到 example_data.csv")

    # 读取CSV文件
    loaded_df = TableUtils.read_table('example_data.csv')
    print("\n从文件读取的数据:")
    print(loaded_df)

if __name__ == "__main__":
    table_examples()
```

示列结果展示

```
原始数据:
   姓名  年龄   部门    工资        入职日期
0  张三  25  技术部  8000  2022-01-15
1  李四  30  市场部  7000  2021-03-20
2  王五  35  技术部  9000  2020-08-10
3  赵六  28  人事部  6000  2023-02-28
4  钱七  32  市场部  7500  2021-11-05

--- 数据筛选 ---
技术部员工:
   姓名  年龄   部门    工资        入职日期
0  张三  25  技术部  8000  2022-01-15
1  王五  35  技术部  9000  2020-08-10

年龄大于30的员工:
   姓名  年龄   部门    工资        入职日期
0  王五  35  技术部  9000  2020-08-10
1  钱七  32  市场部  7500  2021-11-05

--- 数据聚合 ---
部门统计:
    部门      工资  姓名
0  人事部  6000.0   1
1  市场部  7250.0   2
2  技术部  8500.0   2

--- 数据清洗 ---
包含缺失值的数据:
   姓名  年龄   部门      工资        入职日期
0  张三  25  技术部  8000.0  2022-01-15
1  李四  30  市场部  7000.0  2021-03-20
2  王五  35  技术部     NaN  2020-08-10
3  赵六  28  人事部  6000.0  2023-02-28
4  钱七  32  市场部  7500.0  2021-11-05

清洗后的数据:
   姓名  年龄   部门      工资        入职日期
0  张三  25  技术部  8000.0  2022-01-15
1  李四  30  市场部  7000.0  2021-03-20
2  王五  35  技术部     0.0  2020-08-10
3  赵六  28  人事部  6000.0  2023-02-28
4  钱七  32  市场部  7500.0  2021-11-05

--- 数据描述 ---
数据形状: (5, 5)
缺失值统计: {'姓名': 0, '年龄': 0, '部门': 0, '工资': 0, '入职日期': 0}

--- 数据透视表 ---
工资透视表:
年龄       25      28      30      32      35
部门                                         
人事部     NaN  6000.0     NaN     NaN     NaN
市场部     NaN     NaN  7000.0  7500.0     NaN
技术部  8000.0     NaN     NaN     NaN  9000.0

--- 文件操作 ---
数据已保存到 example_data.csv

从文件读取的数据:
   姓名  年龄   部门    工资        入职日期
0  张三  25  技术部  8000  2022-01-15
1  李四  30  市场部  7000  2021-03-20
2  王五  35  技术部  9000  2020-08-10
3  赵六  28  人事部  6000  2023-02-28
4  钱七  32  市场部  7500  2021-11-05

Process finished with exit code 0
```

### 7. 项目说明文档 (`README.md`)

```
# mwj_tools 工具库

目前包含一个功能强大的Python日期时间和表格处理工具库。

## 功能特性

### 📅 日期时间处理 (DateTimeUtils)
- 获取和格式化当前时间
- 时间加减（支持天、小时、分钟、月份）
- 时间戳与datetime互转
- 计算时间差（支持秒、分、时、天）
- 计算未来日期
- 判断周末和工作日
- 获取周范围

### 📊 表格数据处理 (TableUtils)
- 多种格式文件读取（CSV、Excel、JSON）
- 智能数据筛选和过滤
- 数据分组聚合统计
- 多表合并连接
- 缺失值智能清洗
- 数据描述性统计
- 数据透视表生成

## 安装方法

```bash
# 从PyPI安装（发布后）
pip install mwj_tools
# 或
uv add mwj_tools
```

### 8、使用 `uv` 发布到 PyPI

| 步骤 | 命令 | 说明 |
| --- | --- | --- |
| **1. 安装uv** | `pip install uv` | 如果尚未安装 |
| **2. 项目准备** | 确保 `pyproject.toml` 正确 | 检查name、version等 |
| **3. 构建测试** | `uv build` | 生成dist/目录，验证配置 |
| **4. 设置PyPI令牌** | 创建 `~/.pypirc` 或环境变量 | 获取PyPI API令牌 |
| **5. 发布到PyPI** | `uv publish` | 一键构建并上传 |
| **6. 验证安装** | `pip install mwj-tools` | 从PyPI安装测试 |

详细步骤如下：

#### 步骤1-安装uv

```
# 1. 确保已安装uv，跟着文章步骤来的，这一步可以忽略
pip install uv
```

#### 步骤2-项目准备

```
# 2. 进入项目目录
cd mwj-tools
```

#### 步骤3-构建测试

```
# 3. 测试构建 
uv build
```

构建完成后，会在项目根目录下生成一个 dist/ 文件夹，内含 .whl 和 .tar.gz 文件。



#### 步骤4：配置PyPI认证

tips：pypi的双重认证令牌，可以使用腾讯身份验证器进行扫码验证。

`uv publish` 支持多种认证方式，推荐使用 **PyPI API令牌**：

1. 访问 pypi.org，登录后进入 Account Settings → API tokens
2. 创建新令牌（scope选整个账户）
3. 配置到\*\*.pypirc\*\*文件中

```
[pypi]
repository = https://upload.pypi.org/legacy/
username = __token__ # 保持这个不需要修改
password = pypi-<your-api-token-here>
```

注意，`password` 字段的值为 `pypi-` 前缀加上你的完整 API 令牌

#### 步骤5：一键发布

UV 会自动读取 `.pypirc` 中的配置，将 `dist/` 目录下的包上传到 PyPI。

```
# 发布到正式PyPI (默认)
uv publish

# 如果要发布到测试PyTI
uv publish --publish-url https://test.pypi.org/legacy/

# uv publish 实际上执行了以下操作：
# 1. uv build (构建包)
# 2. 上传到PyPI
# 3. 清理临时文件
```

**如果没有使用.pypirc**文件，可以使用`uv publish --token pypi-<API_TOKEN>`进行发布。

使用uv publish的时候会让你输入用户名密码，但现在不支持，用的是token，所以我们需要这样写

```
Enter username ('__token__' if using a token): __token__    ← 输入这个
Enter password: pypi-你的完整API令牌                      ← 输入这个
```



#### 步骤6：验证发布成功

等待几分钟让PyPI同步，然后测试安装，这里只是验证，所以使用的pip下载，如果使用uv可能会比较慢一些，因为仓库需要同步，或者直接去pypi上刷新看一眼



网络运行的情况下，可以直接用官方源下载，之后同步成功，可以用`uv add mwj-tools`进行下载

```
pip install mwj-tools
```

验证代码功能：参考示列代码

发布流程概览

* **构建:**`uv build` 生成分发文件。
* **配置:** 在 `.pypirc` 中设置 PyPI 凭证。
* **发布:**`uv publish` 一键上传至 PyPI。

### 9. 版本更新与后续发布

更新版本时，只需修改 `pyproject.toml` 中的 `version` 字段，然后再次运行 `uv publish`：

```
# pyproject.toml
[project]
name = "mwj-tools"
version = "0.1.1"  # ← 更新版本号
# ... 其他配置
# 发布新版本
uv publish
```

## 快速开始

```
from mwj_tools import DateTimeUtils, TableUtils

# 日期时间示例
now = DateTimeUtils.now()
tomorrow = DateTimeUtils.add_time(now, days=1)
timestamp = DateTimeUtils.to_timestamp(now)

# 表格处理示例
df = TableUtils.read_table('data.csv')
filtered = TableUtils.filter_data(df, {'age': ('>', 30)})
stats = TableUtils.describe_data(df)
```

## 详细示例

查看 `examples/` 目录中的示例文件：

* `datetime_example.py` - 日期时间功能演示
* `table_example.py` - 表格处理功能演示

uv系列更新结束！
