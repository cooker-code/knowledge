> 已吸收至：[[02_Agent与AI工程/0201_Agent框架/020117_DataAgent/020117_核心知识点/Jupyter数据分析Agent工作流边界|Jupyter数据分析Agent工作流边界]]
---
title: 所见即所得：基于Jupyter Lab的智能数据分析工作流
author: 数据打工人的自我修养
date:
url: https://mp.weixin.qq.com/s?__biz=MzE5MTU2NjM5NQ==&mid=2247484088&idx=1&sn=6c85cf28e35d8efc63c82556a1581616&chksm=974e32b910ccb48f8b886071b95511d053c316dc92f00f4dda47516004a17282e83383714975&mpshare=1&scene=24&srcid=09120tBVl6MtiSToVRdBun4z&sharer_shareinfo=39cfd7e69974c55d6a2b90f3fe9bdb51&sharer_shareinfo_first=39cfd7e69974c55d6a2b90f3fe9bdb51#rd
---

在AI数据分析的实际应用过程中，相比于直接将原始数据输入AI并依赖其自动输出各类报表和业务洞察，就当前与业务对接的需求而言，我更倾向于采用“AI辅助生成数据处理过程代码”的方式，而非依赖端到端的提示词自动生成结果。

这一倾向主要基于以下几点考量：

数据准确性与责任归属：业务决策强烈依赖数据的准确性，而直接使用AI全自动生成的数据结果存在一定风险。AI可能产生“幻觉”或输出不完整、不合理的内容，若将有潜在错误的数据交付业务方，可能引发误导。虽然人工校验是一种方式，但若AI过程不透明，校验成本极高，近乎需人工重做，效率低下。

过程可控与可解释性需求：AI直接生成的数据处理流程往往像一个“黑箱”，其内部逻辑、步骤和决策依据缺乏透明性。这种不可见性使得其可靠性存疑，也增加了评估结果可信度的难度。而当AI辅助生成代码时，开发者能够审查、调整和优化每一步数据处理逻辑，确保与业务理解一致，并且更容易符合审计和合规要求

错误追溯与系统性风险：如果AI在自动处理数据的某个环节出错，其错误很可能在后续环节中被放大或掩盖，导致最终输出的一系列分析结论均不可用。由于过程不透明，定位和修复这些错误的成本非常高，甚至可能需要推倒重来。而通过AI生成的、由人审核和执行的代码，一旦出现问题，可以更快速、更精准地定位和修复。

灵活调整与人的主导权：完全依赖提示词让AI输出结果，其调整过程往往比较僵化。可能需要反复修改提示词但仍难以获得满意结果，最终不得不回归手动处理，反而降低了效率。而“AI生成过程代码”的模式则将最终控制权交还给开发者。开发者可以理解、干预、修正和优化AI生成的代码，使其更贴合实际业务需求，实现人机协同的良性循环。

---

AI分析结果准确度三要素

* 需求描述的清晰度与准确性（提示词）

清晰、无歧义的提示词是获得高质量分析结果的前提。如果需求描述本身含糊不清、存在多种解释或关键信息缺失，即使最智能的AI系统也难以准确理解意图，其输出结果很可能包含大量推测性和不确定的内容，直接影响后续分析的可靠性。

* 模型能力与选择

选择合适的模型，对保障输出质量至关重要。在代码生成任务中，国内的DeepSeek和通义千问Code等系列模型目前获得了较好的社区评价。模型不同的设计、参数配置以及其训练数据的质量和范围，对于输出质量可能差异很大。

* 数据质量与治理水平

高质量、标准化、一致性好的数据能极大提升AI输出的可信度。数据是AI进行分析的基石。如果底层数据缺乏治理，例如数据采集（埋点）没有统一规范、上报流程混乱且数据格式不一，那么即使提示词再精确、模型再强大，AI（例如让其编写SQL查询）也难以从低质量的数据中提炼出准确有效的信息。

---

如果具备一定的代码能力（掌握SQL、Python pandas等数据处理库），推荐采用基于Jupyter Lab的环境进行数据分析，并辅以AI代码生成。推荐使用JupyterAI，相比借助其他AI客户端生成代码，JupyterAI可点击按钮直接将生成/修复的代码直接插入/覆盖到单元格，不需要来回辅助粘贴。（页面左侧有AI聊天对话框）

JupyterAI配置请参考：[一键Jupyter AI：聊天对话秒生成代码，开发效率飙升300%！](https://mp.weixin.qq.com/s?__biz=MzE5MTU2NjM5NQ==&mid=2247483951&idx=1&sn=0e86e01064386cd5e3509f4064a9d2ee&scene=21#wechat_redirect)

以下为个人Jupyter AI应用经验，谨与各位同行分享，欢迎大家提供改进意见~

整体应用：

1. 准备TEXT2SQL知识库，相关文章：[效率飙升！数分师的AI新玩具：打造SQL知识库，解锁智能下钻自由](https://mp.weixin.qq.com/s?__biz=MzE5MTU2NjM5NQ==&mid=2247483785&idx=1&sn=224276f5b32c123bfd6daf0624b3f2c7&scene=21#wechat_redirect)

2. TEXT2SQL：构造提示词，AI生成SQL

3. 审查/修改SQL，执行SQL返回Pandas DataFrame数据集df

4. 基于df构造python数据处理及可视化提示词，AI生成python分析代码

5. 审查python代码，执行python语句返回数据及数据可视化。

6. 业务洞察：构造提示词（包括数据明细），AI返回结论及数据洞察。

数据集如果在本地，可直接pandas读取文件，基于AI生成的Python代码进行分析处理

---

如下是相关代码块：

准备工作

数据模块导入：我倾向于一次性导入

```
import jsonimport osimport pandas as pdfrom odps import ODPS  # 数据库接口，按数据库编写import numpy as npfrom openai import OpenAI  # AI接口调用import reimport matplotlibimport matplotlib.pyplot as pltimport seaborn as snsimport plotly.express as pximport plotly.graph_objects as goimport ipywidgets as widgets   # 文本框控件from IPython.display import display, HTML
```

创建AI连接：

按需配置，如下展示了deepseek和百炼api调用示例，api\_key从环境变量中获取

```
# deepseek的连接示例client = OpenAI(    api_key=os.getenv("DEEPSEEK_API_KEY"),     base_url="https://api.deepseek.com")# 百炼，也都兼容OpenAI接口client = OpenAI(    # 若没有配置环境变量，请用百炼API Key将下行替换为：api_key="sk-xxx",    api_key=os.getenv("BAILIAN_API_KEY"), # 如何获取API Key：https://help.aliyun.com/zh/model-studio/developer-reference/get-api-key    base_url="https://dashscope.aliyuncs.com/compatible-mode/v1",)t
```

构造TEXT2SQL提示词：请先准备好本地表元数据及示例SQL脚本文件（推荐json格式）

```
def load_json(file_path):    """加载 JSON 文件"""    with open(file_path, 'r', encoding='utf-8') as f:        return str(json.load(f))
def build_system_prompt(    table_comments_path=r"tablemeta/table_info.json",    table_details_path=r"tablemeta/table_details.json",    limit_rows=100000):    """    构建用于 AI 模型生成 SQL 的完整提示词    参数:        table_comments_path: 表元数据json文件路径        table_details_path: 表结构详情json文件路径        limit_rows: 为orderby语句（如存在）后添加limit记录行数    返回:        str: 用于 AI 模型的完整提示词    """    table_comments = load_json(table_comments_path)    table_details = load_json(table_details_path)    prompt_parts = [        "你是一个数据库专家，擅长编写HIVE、SparkSQL、Trino等引擎查询SQL，编写时你会充分考虑生成SQL的准确性及执行性能。",        "你将基于如下元数据信息为用户编写SQL：\n",        "# 表注释信息：\n" + table_comments + '\n',        "# 表详情：\n" + table_details + '\n',        '# 期望：',        "1. 只需输出sql语句以及代码注释，不要输出其他内容"        "2. 字段名使用中文，中文字段名可以使用``包裹",        "3. 确保SQL正确且高效",        f"4. sql如果有orderby语句，请在order by后面带上limit {limit_rows}语句"    ]    return "\n".join(prompt_parts)
```

构造Python代码AI生成提示词：

```
# 提示词生成def generate_prompt_with_df(df, user_prompt, use_df_name='df', head_rows=2):    """    根据传入的 DataFrame 和用户提示词，生成整合数据结构信息的提示词。    参数:        df (pd.DataFrame): 输入的数据框。        user_prompt (str): 用户的原始提示词。    返回:        str: 整合后的提示词，包含字段信息和样例数据。    """    # 获取字段名和字段类型    schema_info = "\n".join([f"- {col} ({dtype})" for col, dtype in df.dtypes.items()])    # 获取前两行数据作为样例    sample_data = df.head(head_rows).to_string(index=False)    # 构建整合后的提示词    prompt_parts = [        "你是一个Python数据科学专家，擅长使用pandas和plotly/seaborn进行数据分析和可视化。请根据如下信息，生成相应的Python代码来完成用户的请求。",        "# 用户需求",        user_prompt + '\n',        f"你可以直接基于以下已存在的数据框（{use_df_name}:DataFrame）进行统计：\n",        "# 字段类型信息：",        schema_info + '\n',        f"# 数据样例（前{head_rows}行）：",        sample_data + '\n',        "# 注意事项：",        "1. 代码应包括必要的导入语句，并尽可能简洁高效。",        "2. 对于图表需考虑中文显示，图表要生成必要的数值标签，标签视情况加上百分比。",        f"3. 不要在原数据集({use_df_name})上做修改，比如新增列。如有必要，你可以copy一份数据"    ]    final_prompt = '\n'.join(prompt_parts)    print(final_prompt)    return final_prompt
```

编写AI生成SQL函数：输出使用了Textarea文本框，允许用户对生成的sql再编辑

```
def generate_sql(system_prompt, user_prompt,model='deepseek-chat'):    global sql_input    response = client.chat.completions.create(        model=model,        messages=[            {"role": "system", "content": system_prompt},            {"role": "user", "content": user_prompt},        ],        # stream=False    )    response_string = response.choices[0].message.content    sql = re.findall(        string=response_string,        pattern=r'```sql\s(.+)\s```',        flags=re.DOTALL    )    if sql:        # 创建一个 Textarea 控件        sql_input = widgets.Textarea(            value=sql[0],  # 默认值            placeholder='请输入 SQL 语句...',            description='SQL:',            rows=12,            layout=widgets.Layout(width='100%', height='200px')        )        # 显示控件        display(sql_input)    else:        print('未能生成标准sql：\n' + response_string)    return None  # 通过sql_input传递变量，空返回
```

编写SQL运行函数：返回DataFrame数据结构，按自己数据库接口编写函数

```
# 这里是odps的样例，仅供参考。实际应用根据自己数据接口编写。def execute_sql(sql_query, n_cores=4):    with o.execute_sql(sql_query).open_reader(tunnel=True) as reader:        df = reader.to_pandas(n_process=n_cores)    return df
```

---

AI代码生成应用

编写SQL生成提示词：

```
user_prompt = "请帮拉取24年1至28年8月，每月各个活跃结构下的活跃设备数，包括流失设备，活跃结构请打成列展示。"
```

AI根据用户提示词返回SQL：文本框可编辑修改SQL

```
system_prompt = build_system_prompt()  # 生成系统提示词sql = generate_sql(system_prompt,user_prompt)
```

执行SQL：审核/修改完SQL后，执行SQL返回DataFrame。个人测试基于核心场景构造的知识库，十次生成基本七八次不用改，其他稍调下就好。

```
df = execute_sql(sql_input.value) # 从控件获取sql
```

构造python分析提示词：

```
user_prompt = "请分析并可视化 2024 年 1 月至 2025 年 8 月期间的流失设备数变化趋势，绘制折线图，其中横轴为月份（1-12月），纵轴为流失设备数，分别用两条折线展示 2024年 和 2025年 的数据对比，并在折线上标注具体数值，图表需支持中文显示（如标题、坐标轴标签等），"df_prompt = generate_prompt_with_df(df, user_prompt)
```

```
你是一个Python数据科学专家，擅长使用pandas和plotly/seaborn进行数据分析和可视化。请根据如下信息，生成相应的Python代码来完成用户的请求。# 用户需求请分析并可视化 2024 年 1 月至 2025 年 8 月期间的流失设备数变化趋势，绘制折线图，其中横轴为月份（1-12月），纵轴为流失设备数，分别用两条折线展示 2024年 和 2025年 的数据对比，并在折线上标注具体数值，图表需支持中文显示（如标题、坐标轴标签等），你可以直接基于以下已存在的数据框（df:DataFrame）进行统计：# 字段类型信息：- 月份 (object)- 留存设备数 (int64)- 回流设备数 (int64)- 新增设备数 (int64)- 流失设备数 (int64)# 数据样例（前2行）：        月份   留存设备数  回流设备数  新增设备数  流失设备数2024-01-01 429   28  862  7902024-02-01 546  705  745  473# 注意事项：1. 代码应包括必要的导入语句，并尽可能简洁高效。2. 对于图表需考虑中文显示，图表要生成必要的数值标签，标签视情况加上百分比。3. 不要在原数据集(df)上做修改，如有必要，你可以copy一份数据，比如新增列
```

以上数值仅为修改后的样例，非真实数据，提示词仅供参考

JupyterAI根据以上提示词生成python代码：

复制提示词到聊天框，然后点击按钮将代码插入到单元格运行。当然也可以同以上SQL那样使用AI接口直接返回python代码。

截图-按钮：将代码插入到活动单元格下一个单元格

AI生成代码样例：这里只展示了处理部分，大家可干预审核修改

```
# 1. 复制数据以避免修改原始数据框df_copy = df.copy()
# 2. 将“月份”列转换为日期类型并提取年份和月份df_copy['日期'] = pd.to_datetime(df_copy['月份'])df_copy = df_copy[(df_copy['日期'].dt.year == 2024) | (df_copy['日期'].dt.year == 2025)]df_copy['年份'] = df_copy['日期'].dt.yeardf_copy['月份_num'] = df_copy['日期'].dt.month
# 3. 透视数据以便于绘图pivot_df = df_copy.pivot(index='月份_num', columns='年份', values='流失设备数').reset_index()pivot_df.columns.name = None  # 移除列名名称
```

绘图样例如下：

如需文件下载，执行数据导出代码即可：

```
pivot_df.to_excel('24-25年流失设备数对比.xlsx',index=False)
```

---

关于TEXT2SQL准确度补充：

* 写好提示词
* 选好模型：国内当前deepseek、qwen3-coder-plus基本够用了，我ds用v3版本，基本满足需求。（R1更好，但是慢）
* 确保知识库（json文件）数据质量，一堆残缺不规范的数据，论谁也难搞。
* 可以从核心场景开始构造，构造宽表（维度冗余、反范式），统计指标将多个维度整合到一个表里，这样能减少数据关联，AI基于单表输出更准确，日常分析维度拆解也将变得更容易。

数据安全：

AI应用中，数据安全一直很关键，尽管以上只涉及了元数据信息，倘若涉及到业务核心规则，也请审视，视情况使用。

相关文章推荐：

[AI浪潮下的数据分析师转型：从数据民工到决策架构师——在技术革命中重塑不可替代的价值](https://mp.weixin.qq.com/s?__biz=MzE5MTU2NjM5NQ==&mid=2247484018&idx=1&sn=4d74c8f804e8bc65161f549da8552b49&scene=21#wechat_redirect)

[一键Jupyter AI：聊天对话秒生成代码，开发效率飙升300%！](https://mp.weixin.qq.com/s?__biz=MzE5MTU2NjM5NQ==&mid=2247483951&idx=1&sn=0e86e01064386cd5e3509f4064a9d2ee&scene=21#wechat_redirect)

[效率飙升！数分师的AI新玩具：打造SQL知识库，解锁智能下钻自由](https://mp.weixin.qq.com/s?__biz=MzE5MTU2NjM5NQ==&mid=2247483785&idx=1&sn=224276f5b32c123bfd6daf0624b3f2c7&scene=21#wechat_redirect)

[一款可视化 + 智能分析的 AI 数据分析平台](https://mp.weixin.qq.com/s?__biz=MzE5MTU2NjM5NQ==&mid=2247484046&idx=1&sn=2c68d70c2840a75a33797ec1fc42dc65&scene=21#wechat_redirect)

[用户标签智能生成：AI+场景适配](https://mp.weixin.qq.com/s?__biz=MzE5MTU2NjM5NQ==&mid=2247483878&idx=1&sn=7989bca4a6623ab53b377568d09d467a&scene=21#wechat_redirect)

[数据分析核心技能：指标异常界定与根因定位的框架拆解](https://mp.weixin.qq.com/s?__biz=MzE5MTU2NjM5NQ==&mid=2247483957&idx=1&sn=d9048d71604c061f6cd804e5e84266b0&scene=21#wechat_redirect)

[Spark/Hive避坑指南：GROUPING SETS与COUNT(DISTINCT)的膨胀机制解析及优化](https://mp.weixin.qq.com/s?__biz=MzE5MTU2NjM5NQ==&mid=2247484010&idx=1&sn=297f3e573c1a50b7f44ca521f2632bb4&scene=21#wechat_redirect)

[SQL跑不动？数据倾斜惹的祸！全场景优化方案大揭秘](https://mp.weixin.qq.com/s?__biz=MzE5MTU2NjM5NQ==&mid=2247483826&idx=1&sn=de5d8134932605da5badc3e1fa8855c8&scene=21#wechat_redirect)

---

**>>>>>>>>>> ✧ END ✧ <<<<<<<<<<**

感谢您的阅读！

点击下方链接关注，持续为您呈现更多内容~