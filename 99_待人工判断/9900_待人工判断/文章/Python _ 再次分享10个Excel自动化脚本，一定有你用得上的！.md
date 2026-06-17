---
title: Python | 再次分享10个Excel自动化脚本，一定有你用得上的！
author: 印象Python
date: 
url: http://mp.weixin.qq.com/s?__biz=MzkzNDcxOTk0Ng==&mid=2247488258&idx=1&sn=930d8ee700cfad4e929cb59295053c8a&chksm=c382110d62574ad30a6b0a581d0caf0943ca31d8bff4e122973e15da1049d7eac02f2163ad0b&mpshare=1&scene=24&srcid=1213Uto94AMwTwHIyl9Hti6I&sharer_shareinfo=067acb46f8da273f83a177691b231fa1&sharer_shareinfo_first=067acb46f8da273f83a177691b231fa1#rd
---

# 在数据处理和分析的过程中，Excel文件是我们日常工作中常见的格式。通过Python，我们可以实现对Excel文件的各种自动化操作，提高工作效率。

# 本文将再次分享10个实用的Excel自动化脚本，以帮助新手小白更轻松地掌握这些技能。

## 

## 1. Excel单元格批量填充

```
import pandas as pd  
  
# 批量填充指定列的单元格  
def fill_column(file_path, column_name, value):  
    df = pd.read_excel(file_path)  
    df[column_name] = value  # 将指定列的所有单元格填充为value  
    df.to_excel(file_path, index=False)  
  
fill_column('example.xlsx', '备注', '已处理')  
print("备注列已成功填充！")
```

### 解释

此脚本将example.xlsx中的“备注”列全部填充为“已处理”。对于普通用户来说，处理大量数据时常需要对某一列进行统一标记，这个功能就显得尤为重要。

## 

## 2. 设置行高与列宽

```
from openpyxl import load_workbook  
  
# 设置Excel的行高与列宽  
def set_row_column_size(file_path):  
    wb = load_workbook(file_path)  
    ws = wb.active  
  
    # 设置第一行行高、第一列列宽  
    ws.row_dimensions[1].height = 30  # 设置行高  
    ws.column_dimensions['A'].width = 20  # 设置列宽  
  
    wb.save(file_path)  
  
set_row_column_size('example.xlsx')  
print("行高和列宽设置成功！")
```

### 解释

这个脚本为Excel文件设置了第一行的行高和第一列的列宽。适当调整行高和列宽可以提高表格的可读性，尤其是在内容较多或较复杂时，使用此功能可以使报告更加美观易读。

## 

## 3. 根据条件删除行

```
# 根据条件删除Excel中的行  
def delete_rows_based_on_condition(file_path, column_name, condition):  
    df = pd.read_excel(file_path)  
    df = df[df[column_name] != condition]  # 删除满足条件的行  
    df.to_excel(file_path, index=False)  
  
delete_rows_based_on_condition('example.xlsx', '状态', '无效')  
print("符合条件的行已删除！")
```

### 解释

该脚本从Excel中删除“状态”列中值为“无效”的行。这种操作在数据清理过程中非常常见，有助于减少数据集中的噪声，提高数据分析的准确性。

## 

## 4. 创建新的Excel工作表

```
# 在现有Excel文件中创建新的工作表  
def create_new_sheet(file_path, sheet_name):  
    wb = load_workbook(file_path)  
    wb.create_sheet(title=sheet_name)  # 创建新的工作表  
    wb.save(file_path)  
  
create_new_sheet('example.xlsx', '新工作表')  
print("新工作表创建成功！")
```

### 解释

该脚本在已有的Excel文件中创建一个新的工作表。这对于组织数据，分开不同任务或项目的数据非常有用，保持文件结构的清晰。

## 

## 5. 导入CSV文件到Excel

```
# 将CSV文件导入到Excel工作表  
def import_csv_to_excel(csv_file, excel_file):  
    df = pd.read_csv(csv_file)  
    df.to_excel(excel_file, index=False)  
  
import_csv_to_excel('data.csv', 'imported_data.xlsx')  
print("CSV文件成功导入到Excel！")
```

### 解释

这个脚本将CSV文件导入到Excel中。很多时候，数据是以CSV格式提供的，通过该脚本可以方便地将其转换为Excel格式，便于后续分析和处理。

## 

## 6. 数据透视表生成

```
# 生成数据透视表并保存到新的Excel文件  
def generate_pivot_table(file_path, index_column, values_column, output_file):  
    df = pd.read_excel(file_path)  
    pivot_table = df.pivot_table(index=index_column, values=values_column, aggfunc='sum')  # 汇总  
    pivot_table.to_excel(output_file)  
  
generate_pivot_table('sales_data.xlsx', '地区', '销售额', 'pivot_output.xlsx')  
print("透视表生成成功！")
```

### 解释

该脚本根据给定的“地区”和“销售额”列生成汇总透视表，并保存到新文件中。在进行业务分析时，透视表能快速展示不同维度下的数据总结。

## 

## 7. 格式化Excel

```
from openpyxl.styles import Font, Color  
  
# 设置Excel单元格字体样式  
def format_cells(file_path):  
    wb = load_workbook(file_path)  
    ws = wb.active  
  
    for cell in ws['A']:  # 遍历A列  
        cell.font = Font(bold=True, color="FF0000")  # 设置字体加粗和红色  
  
    wb.save(file_path)  
  
format_cells('example.xlsx')  
print("单元格格式化成功！")
```

### 解释

该脚本将example.xlsx中的A列字体设置为加粗和红色。这种格式化通常用于强调特定数据，使报告更具视觉吸引力。

## 

## 8. 分析并输出描述性统计

```
# 输出描述性统计到Excel  
def descriptive_statistics(file_path, output_file):  
    df = pd.read_excel(file_path)  
    stats = df.describe()  # 计算描述性统计  
    stats.to_excel(output_file)  
  
descriptive_statistics('example.xlsx', 'statistics_output.xlsx')  
print("描述性统计输出成功！")
```

### 解释

该脚本计算Excel文件的描述性统计信息（如均值、标准差等），并将结果保存到新的Excel文件中。这对于了解数据的基本特征非常重要，尤其在数据分析前期阶段。

## 9. 批量修改Excel文件名称

```
import os  
  
# 批量重命名指定目录下的Excel文件  
def rename_excel_files(directory, prefix):  
    for filename in os.listdir(directory):  
        if filename.endswith('.xlsx'):  
            new_name = f"{prefix}_{filename}"  
            os.rename(os.path.join(directory, filename), os.path.join(directory, new_name))  
            print(f"已将 {filename} 重命名为 {new_name}")  
  
rename_excel_files('/path/to/excel/files', '2024')
```

### 解释

该脚本批量重命名指定目录中的所有Excel文件，在每个文件名前面添加一个前缀。对于需要处理大量Excel文件的用户来说，这种批量操作非常便利，比如根据年份或项目为文件命名，以便于管理和归档。

## 10. 自动发送包含Excel数据的电子邮件

```
import smtplib  
from email.mime.multipart import MIMEMultipart  
from email.mime.application import MIMEApplication  
from email.mime.text import MIMEText  
  
# 自动发送带有Excel附件的电子邮件  
def send_email(to_address, subject, body, excel_file):  
    from_address = "your_email@example.com"  
    password = "your_password"  
  
    msg = MIMEMultipart()  
    msg['From'] = from_address  
    msg['To'] = to_address  
    msg['Subject'] = subject  
  
    # 添加正文  
    msg.attach(MIMEText(body, 'plain'))  
  
    # 添加Excel附件  
    with open(excel_file, "rb") as attachment:  
        part = MIMEApplication(attachment.read(), Name=os.path.basename(excel_file))  
        part['Content-Disposition'] = f'attachment; filename="{os.path.basename(excel_file)}"'  
        msg.attach(part)  
  
    # 发送邮件  
    with smtplib.SMTP('smtp.example.com', 587) as server:  
        server.starttls()  
        server.login(from_address, password)  
        server.send_message(msg)  
  
send_email('recipient@example.com', 'Monthly Report', 'Please find attached the monthly report.', 'report.xlsx')  
print("邮件发送成功！")
```

### 解释

此脚本使用SMTP协议自动发送一封电子邮件，其中附带了一个Excel文件。这个功能在工作中尤其有用，比如每月定期发送财务报表或业绩报告给相关人员。通过自动化邮件发送，可以节省时间并减少人为错误。

希望这些工具能为你的工作带来便利，并激发你进一步探索Python在数据处理方面的潜力！如果你还有其他问题或想法，欢迎随时交流！

**全套Python学习资料分享：**

一、Python所有方向的学习路线

Python所有方向路线就是把Python常用的技术点做整理，形成各个领域的知识点汇总，它的用处就在于，你可以按照上面的知识点去找对应的学习资源，保证自己学得较为全面。

二、全套PDF电子书

书籍的好处就在于权威和体系健全，刚开始学习的时候你可以只看视频或者听某个人讲课，但等你学完之后，你觉得你掌握了，这时候建议还是得去看一下书籍，看权威技术书籍也是每个程序员必经之路。

三、python入门资料大全

四、python进阶资料大全

五、python爬虫专栏

六、入门学习视频全套

我们在看视频学习的时候，不能光动眼动脑不动手，比较科学的学习方法是在理解之后运用它们，这时候练手项目就很适合了。

七、实战案例

光学理论是没用的，要学会跟着一起敲，要动手实操，才能将自己的所学运用到实际当中去，这时候可以搞点实战案例来学习。

八、python最新面试题

**获取资料：**

1、点赞+再看

2、关注公众号【印象Python】

3、公众号内回复【获取2024最新Python全套资料大全】即可获取