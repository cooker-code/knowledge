---
title: 20个Python脚本，来自动化完成日常任务，提升工作效率
author: Python运维实践
date: 
url: https://mp.weixin.qq.com/s?__biz=MzUyMTYwMDYxMA==&mid=2247502867&idx=1&sn=2aa36c5823fed7f4569d2730f806d851&chksm=f803a371031264f6189c23ced5987f3c0a49fecefecdd4c71bde33b3f7266b2f37349669a323&mpshare=1&scene=24&srcid=0624JTMyyCy5Ek52e1RXgMO2&sharer_shareinfo=e3fb89f469bbe9a1eb66d3d8b17b706d&sharer_shareinfo_first=e3fb89f469bbe9a1eb66d3d8b17b706d#rd
---

Python凭借其简洁的语法和强大的库，堪称创建自动化脚本的最佳编程语言之一。无论你是程序员还是想简化日常工作的普通用户，Python都能提供得力的工具。

本文将分享用于自动化各类任务的20个Python脚本，这些脚本非常适合希望优化工作流程、提升效率的实践者。

#### **1.批量重命名文件**

逐个重命名文件耗时费力，而使用Python的os模块可以轻松实现自动化。

以下脚本可根据指定模式批量修改文件夹中的文件名：

```
import os  
  
def bulk_rename(folder_path, old_name_part, new_name_part):  
    for filename in os.listdir(folder_path):  
        if old_name_part in filename:  
            new_filename = filename.replace(old_name_part, new_name_part)  
            os.rename(os.path.join(folder_path, filename), os.path.join(folder_path, new_filename))  
            print(f"Renamed {filename} to {new_filename}")  
  
folder = '/path/to/your/folder' bulk_rename(folder, 'old_part', 'new_part')
```

此脚本搜索名称中包含的文件`old_part`，并将其替换为`new_part`。

#### **2.自动备份文件**

我们都知道定期备份文件有多重要，而利用Python的shutil模块可以轻松实现自动化备份。

以下脚本会将指定目录的所有文件复制到备份目录：

```
import shutil  
import os  
  
def backup_files(src_dir, dest_dir):  
    if not os.path.exists(dest_dir):  
        os.makedirs(dest_dir)  
    for file in os.listdir(src_dir):  
        full_file_name = os.path.join(src_dir, file)  
        if os.path.isfile(full_file_name):  
            shutil.copy(full_file_name, dest_dir)  
            print(f"Backed up {file} to {dest_dir}")  
  
source = '/path/to/source/directory' destination = '/path/to/destination/directory' backup_files(source, destination)
```

你可以通过cron（Linux系统）或任务计划程序（Windows系统）等工具，将该脚本设置为每日自动运行。

#### **3.从 Internet 下载文件**

如果经常需要从网络下载文件，使用 **aiohttp** 库就能轻松实现自动化下载。

以下是一个简单的脚本示例，可从指定 URL 下载文件：

```
import aiohttp  
import asyncio  
import aiofiles  
  
async def download_file(url, filename):  
    async with aiohttp.ClientSession() as session:  
        async with session.get(url) as response:  
            async with aiofiles.open(filename, 'wb') as file:  
                await file.write(await response.read())  
            print(f"Downloaded {filename}")  
  
urls = {  
    'https://example.com/file1.zip': 'file1.zip',  
    'https://example.com/file2.zip': 'file2.zip'  
}  
  
async def download_all():  
    tasks = [download_file(url, filename) for url, filename in urls.items()]  
    await asyncio.gather(*tasks)  
  
asyncio.run(download_all())
```

该脚本会从给定的 URL 下载文件，并自动保存到你指定的文件夹中。

#### **4.自动化电子邮件报告**

如果需要定期发送邮件报告，可以使用 **smtplib** 库实现自动化。这个库能轻松通过 Gmail 账户发送邮件：

```
import smtplib  
from email.mime.text import MIMEText  
from email.mime.multipart import MIMEMultipart  
  
def send_email(subject, body, to_email):  
    sender_email = 'youremail@gmail.com'  
    sender_password = 'yourpassword'  
    receiver_email = to_email  
  
    msg = MIMEMultipart()  
    msg['From'] = sender_email  
    msg['To'] = receiver_email  
    msg['Subject'] = subject  
    msg.attach(MIMEText(body, 'plain'))  
  
    try:  
        server = smtplib.SMTP('smtp.gmail.com', 587)  
        server.starttls()  
        server.login(sender_email, sender_password)  
        server.sendmail(sender_email, receiver_email, msg.as_string())  
        server.quit()  
        print("Email sent successfully!")  
    except Exception as e:  
        print(f"Failed to send email: {e}")  
  
subject = 'Monthly Report'  
body = 'Here is the monthly report.'  
send_email(subject, body, 'receiver@example.com')
```

该脚本会向指定收件人发送带主题和正文的简易邮件。注意：使用此方法前，请确保已在 Gmail 中启用『低安全性应用』权限。

#### **5.任务调度程序（任务自动化）**

定时任务自动化可以轻松通过 **schedule** 库实现，它能在指定时间自动执行发送邮件、运行备份脚本等操作：

```
import schedule  
import time  
  
def job():  
    print("Running scheduled task!")  
  
# Schedule the task to run every day at 10:00 AM  
schedule.every().day.at("10:00").do(job)  
  
while True:  
    schedule.run_pending()  
    time.sleep(1)
```

该脚本将持续运行，并在每天上午**10:00**准时触发预设任务。

#### **6.用于数据收集的 Web Scraping**

相比同步的 **requests** 库，使用 **aiohttp** 进行异步 HTTP 请求能让网页抓取效率大幅提升。

这个示例演示了如何并行抓取多个页面：

```
import aiohttp  
import asyncio  
from bs4 import BeautifulSoup  
  
async def fetch(session, url):  
    async with session.get(url) as response:  
        return await response.text()  
  
async def scrape(urls):  
    async with aiohttp.ClientSession() as session:  
        tasks = [fetch(session, url) for url in urls]  
        html_pages = await asyncio.gather(*tasks)  
        for html in html_pages:  
            soup = BeautifulSoup(html, 'html.parser')  
            print(soup.title.string)  
  
urls = ['https://example.com/page1', 'https://example.com/page2'] asyncio.run(scrape(urls))
```

#### **7.自动生成发票**

定期生成发票？用 **Fpdf** 这类库就能轻松实现自动化开票：

```
from fpdf import FPDF  
  
def create_invoice(client_name, amount):  
    pdf = FPDF()  
    pdf.add_page()  
    pdf.set_font("Arial", size=12)  
    pdf.cell(200, 10, txt="Invoice", ln=True, align='C')  
    pdf.cell(200, 10, txt=f"Client: {client_name}", ln=True, align='L')  
    pdf.cell(200, 10, txt=f"Amount: ${amount}", ln=True, align='L')  
    pdf.output(f"{client_name}_invoice.pdf")  
    print(f"Invoice for {client_name} created successfully!")  
  
create_invoice('John Doe', 500)
```

该脚本可快速生成标准发票并自动保存为PDF格式。

#### **8.监控网站正常运行时间**

利用 **requests** 库，我们可以轻松实现网站运行状态的自动化监控，定时检测目标网站是否在线：

```
import requests  
import time  
  
def check_website(url):  
    try:  
        response = requests.get(url)  
        if response.status_code == 200:  
            print(f"Website {url} is up!")  
        else:  
            print(f"Website {url} returned a status code {response.status_code}")  
    except requests.exceptions.RequestException as e:  
        print(f"Error checking website {url}: {e}")  
  
url = 'https://example.com' while True: check_website(url) time.sleep(3600) # Check every hour
```

该脚本会自动检测网站可用性，并返回当前状态码。

#### **9.自动回复电子邮件**

如果经常需要处理大量邮件，可以通过 **imaplib** 和 **smtplib** 库实现自动回复功能：

```
import imaplib  
import smtplib  
from email.mime.text import MIMEText  
  
def auto_reply():  
    # Connect to email server  
    mail = imaplib.IMAP4_SSL("imap.gmail.com")  
    mail.login('youremail@gmail.com', 'yourpassword')  
    mail.select('inbox')  
  
    # Search for unread emails  
    status, emails = mail.search(None, 'UNSEEN')  
  
    if status == "OK":  
        for email_id in emails[0].split():  
            status, email_data = mail.fetch(email_id, '(RFC822)')  
            email_msg = email_data[0][1].decode('utf-8')  
  
            # Send auto-reply  
            send_email("Auto-reply", "Thank you for your email. I'll get back to you soon.", 'sender@example.com')  
  
def send_email(subject, body, to_email):  
    sender_email = 'youremail@gmail.com'  
    sender_password = 'yourpassword'  
    receiver_email = to_email  
  
    msg = MIMEText(body)  
    msg['From'] = sender_email  
    msg['To'] = receiver_email  
    msg['Subject'] = subject  
  
    with smtplib.SMTP_SSL('smtp.gmail.com', 465) as server:  
        server.login(sender_email, sender_password)  
        server.sendmail(sender_email, receiver_email, msg.as_string())  
  
auto_reply()
```

该脚本会自动识别未读邮件，并使用预设内容进行回复。

#### **10.文件清理**

Python 能高效实现文件自动清理，特别适合定期删除或归档旧文件，保持目录整洁有序。

以下是一个基于 **os** 和 **time** 模块的脚本示例，可自动删除超过指定天数的文件：

```
import aiofiles  
import os  
import asyncio  
import time  
  
async def clean_up(folder_path, days_old):  
    now = time.time()  
    cutoff_time = now - (days_old * 86400)  
    for filename in os.listdir(folder_path):  
        file_path = os.path.join(folder_path, filename)  
        if os.path.getmtime(file_path) < cutoff_time:  
            await aiofiles.os.remove(file_path)  
            print(f"Deleted {filename}")  
  
folder = '/path/to/your/folder'  
asyncio.run(clean_up(folder, 30))
```

#### **11.自动生成密码**

生成高强度、唯一性密码是安全防护的关键，而借助 Python 的 **random** 模块，我们可以轻松实现自动化生成。

以下脚本可创建指定长度的随机密码，通过组合字母、数字和特殊字符来提升安全性：

```
import random  
import asyncio  
import string  
  
async def generate_password(length=12):  
    characters = string.ascii_letters + string.digits + string.punctuation  
    password = ''.join(random.choice(characters) for _ in range(length))  
    return password  
  
async def generate_multiple_passwords(n, length=12):  
    tasks = [generate_password(length) for _ in range(n)]  
    passwords = await asyncio.gather(*tasks)  
    print(passwords)  
  
asyncio.run(generate_multiple_passwords(5))
```

#### **12.任务追踪/提醒系统**

利用 Python 的 **datetime** 和 **asyncio** 模块，可以轻松构建任务追踪与定时提醒功能。

```
import asyncio  
from datetime import datetime  
  
async def task_reminder(task_name, interval):  
    while True:  
        print(f"Reminder: {task_name} - {datetime.now()}")  
        await asyncio.sleep(interval)  
  
async def main():  
    await asyncio.gather(  
        task_reminder("Drink Water", 7200),  # Remind every 2 hours  
        task_reminder("Take a Break", 3600)  # Remind every 1 hour  
    )  
  
asyncio.run(main())
```

该脚本会在预定时间自动发送任务提醒。

#### **13.自动生成每日报告**

使用 Python 自动采集数据并生成格式化报告：

```
import datetime  
import aiofiles  
import asyncio  
  
async def generate_report(data):  
    today = datetime.date.today()  
    filename = f"daily_report_{today}.txt"  
    async with aiofiles.open(filename, 'w') as file:  
        await file.write(f"Report for {today}\n")  
        await file.write("\n".join(data))  
    print(f"Report generated: {filename}")  
  
data = ["Task 1: Completed", "Task 2: Pending", "Task 3: Completed"]  
asyncio.run(generate_report(data))
```

#### **14.监控系统资源**

如果是系统管理员，则可以在`psutil`库的帮助下使用Python来监控系统的资源，例如CPU和内存使用情况。

```
import psutil  
  
def monitor_resources():  
    cpu_usage = psutil.cpu_percent(interval=1)  
    memory_usage = psutil.virtual_memory().percent  
    print(f"CPU Usage: {cpu_usage}%")  
    print(f"Memory Usage: {memory_usage}%")  
  
monitor_resources()
```

#### **15.批量调整图像大小**

如果需要批量调整图像大小，可以使用Python的`Pillow`库轻松实现。

```
from PIL import Image  
import os  
import asyncio  
from concurrent.futures import ProcessPoolExecutor  
  
def resize_image(filename, width, height):  
    img = Image.open(filename)  
    img = img.resize((width, height))  
    img.save(f"resized_{filename}")  
    return f"Resized {filename}"  
  
async def resize_images(folder_path, width, height):  
    with ProcessPoolExecutor() as executor:  
        loop = asyncio.get_event_loop()  
        tasks = []  
        for filename in os.listdir(folder_path):  
            if filename.endswith('.jpg'):  
                tasks.append(loop.run_in_executor(  
                    executor, resize_image, os.path.join(folder_path, filename), width, height))  
        results = await asyncio.gather(*tasks)  
        print(results)  
  
folder = '/path/to/your/images'  
asyncio.run(resize_images(folder, 800, 600))
```

此脚本将文件夹中的所有`.jpg`图像大小调整为指定的尺寸。

#### **16.自动将数据备份到云**

通过`pydrive` 等 Python 库，可以实现自动备份文件至 Google Drive 等云服务的功能。

```
from pydrive.auth import GoogleAuth  
from pydrive.drive import GoogleDrive  
  
def backup_to_google_drive(file_path):  
    gauth = GoogleAuth()  
    gauth.LocalWebserverAuth()  
    drive = GoogleDrive(gauth)  
    file = drive.CreateFile({'title': 'backup_file.txt'})  
    file.Upload()  
    print("Backup uploaded successfully!")  
  
file = '/path/to/your/file.txt' backup_to_google_drive(file)
```

#### **17.创建每日提醒**

使用 **time** 模块可以轻松设置定时提醒，比如每隔2小时提醒喝水：

```
import time  
  
def water_reminder():  
    while True:  
        print("Time to drink water!")  
        time.sleep(7200)  # Remind every 2 hours  
  
water_reminder()
```

#### **18.自动将数据输入到 Excel**

如果经常需要往Excel录入数据，使用`openpyxl` 库就能轻松实现自动化操作：

```
from openpyxl import Workbook  
  
def create_excel(data):  
    wb = Workbook()  
    ws = wb.active  
    for row in data:  
        ws.append(row)  
    wb.save('data.xlsx')  
    print("Excel file created successfully!")  
  
data = [  
    ["Name", "Age", "City"],  
    ["John", 30, "New York"],  
    ["Anna", 25, "London"],  
]  
create_excel(data)
```

#### **19.自动数据清理**

面对大型数据集时，利用Python可以轻松实现数据清洗自动化，例如自动删除`CSV`文件中的空行：

```
import csv  
  
def clean_csv(file_path):  
    with open(file_path, 'r') as infile:  
        reader = csv.reader(infile)  
        rows = [row for row in reader if any(row)]  
      
    with open(file_path, 'w', newline='') as outfile:  
        writer = csv.writer(outfile)  
        writer.writerows(rows)  
      
    print("Empty rows removed from CSV")  
  
file = '/path/to/your/data.csv' clean_csv(file)
```

#### **20.从图像中提取文本**

借助`pytesseract`库，Python 能轻松实现图像文字识别，特别适用于印刷内容数字化或扫描文档文本提取等场景。

```
from PIL import Image  
import pytesseract  
  
def extract_text_from_image(image_path):  
    # Open the image file  
    img = Image.open(image_path)  
      
    # Use pytesseract to extract text  
    text = pytesseract.image_to_string(img)  
      
    return text  
  
image_path = 'path_to_your_image.jpg'  
extracted_text = extract_text_from_image(image_path)  
print("Extracted Text:\n", extracted_text)
```