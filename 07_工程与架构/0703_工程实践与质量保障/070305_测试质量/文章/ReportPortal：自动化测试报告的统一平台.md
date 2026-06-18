---
title: ReportPortal：自动化测试报告的统一平台
author: 东汉末年出bug
date: 
url: http://mp.weixin.qq.com/s?__biz=MzkwNDc0MTI4NA==&mid=2247484880&idx=1&sn=a441de10090316d609fd8f345957ba46&chksm=c1b24e85b41bbed0463b56b7cc9f6867aa40ea9bf8063eea49dd1645745caeaf65e2ad0d9721&mpshare=1&scene=24&srcid=1224TDISf5RUKNzB0xj4WSN4&sharer_shareinfo=bd40ef29bfd2c36c7415a84c22391e53&sharer_shareinfo_first=bd40ef29bfd2c36c7415a84c22391e53#rd
---

ReportPortal，作为一个开源的、面向服务的、基于Web的平台，它集成了多种测试框架，提供了增强的机器学习分析、高级报告和实时数据聚合功能，帮助团队提高测试质量和效率。

#### 一、ReportPortal简介

ReportPortal（项目地址：https://gitcode.com/gh\_mirrors/re/reportportal 和 https://github.com/reportportal/reportportal）是一个集成了多种测试框架的统一报告、分析和可视化工具。它支持Cypress、Playwright、TestNG、Selenium等多种测试框架，能够实时展示测试结果，保留历史测试信息，并与Jira等bug跟踪系统集成。ReportPortal已被全球超过1600家公司使用，其中包括40多家财富500强企业。

#### 二、ReportPortal的搭建

在搭建ReportPortal之前，请确保您的系统已安装Docker和Docker Compose。以下是ReportPortal的快速启动步骤：

1. 1. **克隆项目仓库**：

   ```
   git clone https://github.com/reportportal/reportportal.git  
   cd reportportal
   ```
2. 2. **启动ReportPortal**：

   ```
   docker-compose -p reportportal up -d --force-recreate
   ```

   这将启动ReportPortal及其所有依赖服务。
3. 3. **访问ReportPortal**： 打开浏览器并访问 http://localhost:8080，您将看到ReportPortal的登录页面。默认的登录凭证为：为了安全起见，请更改管理员密码。

* • 用户名：superadmin
* • 密码：erebus

#### 三、ReportPortal的使用

ReportPortal提供了丰富的功能，以下是一些关键功能的介绍：

1. 1. **集成测试框架**： ReportPortal支持与多种测试框架集成，如Cypress、Playwright、TestNG、Selenium等。通过集成，您可以实现测试结果的实时报告和分析。
2. 2. **使用质量门**： 在项目交付周期中设置质量门，以确保软件质量达到预定标准。质量门可以帮助团队及时发现并修复潜在问题，提高软件质量。
3. 3. **利用AI功能**： ReportPortal的AI功能可以进行故障原因检测和缺陷分类，提高问题解决效率。通过机器学习算法，ReportPortal能够自动识别并分类测试中的故障，帮助团队快速定位问题。
4. 4. **生态项目**： ReportPortal拥有多个生态项目，如TDSpora、Drill4J和Healenium等。这些项目与ReportPortal集成，提供了更强大的功能：

* • TDSpora：一个测试数据管理工具，用于数据迁移和子集设置。
* • Drill4J：一个测试影响分析工具，用于最小化回归测试的范围。
* • Healenium：一个Selenium扩展，提供自愈功能，能够自动修复测试中的元素定位问题。

#### 四、ReportPortal与Python的集成

ReportPortal可以通过Python客户端将测试结果推送到平台。以下是详细的集成步骤：

1. 1. **安装Docker和docker-compose**： 确保您的系统已安装Docker和docker-compose。
2. 2. **下载docker-compose文件**： 将docker-compose文件下载到您想要安装的文件夹：

   ```
   curl https://raw.githubusercontent.com/reportportal/reportportal/master/docker-compose.yml -o docker-compose.yml
   ```
3. 3. **启动ReportPortal**： 在ReportPortal的文件夹执行docker-compose命令：

   ```
   docker-compose -p reportportal up -d
   ```
4. 4. **访问ReportPortal**： 在浏览器中打开 http://localhost:8080，使用用户名和密码进行访问。
5. 5. **安装reportportal-client-Python库**： 使用pip安装reportportal-client库：

   ```
   pip install reportportal-client
   ```

   注意：最新版本不支持低于5.0.0的ReportPortal版本。如果要安装3.0版本，请执行：

   ```
   pip install reportportal-client~=3.0
   ```
6. 6. **推送测试数据**： 使用reportportal-client库将测试数据推送到ReportPortal。以下是一个示例代码：

   ```
   import os  
   import subprocess  
   import traceback  
   from mimetypes import guess_type  
   from time import time  
     
   # Report Portal versions below 5.0.0:  
   # from reportportal_client import ReportPortalServiceAsync  
     
   # Report Portal versions >= 5.0.0:  
   from reportportal_client importReportPortalService  
     
   def timestamp():  
       return str(int(time()*1000))  
     
   endpoint = "http://localhost:8080"  
   project = "default"# You can get UUID from user profile page in the Report Portal.  
   token = "your_token_here"# Replace with your actual token.  
   launch_name = "Test launch"  
   launch_doc = "Testing logging with attachment."  
     
   def my_error_handler(exc_info):  
   """ This callback function will be called by async service client when error occurs.  
           Return True if error is not critical and you want to continue work.  
           :param exc_info: result of sys.exc_info() -> (type, value, traceback)  
           :return:   
       """  
       print("Error occurred: {}".format(exc_info[1]))  
       traceback.print_exception(*exc_info)  
     
   # Report Portal versions below 5.0.0:  
   # service = ReportPortalServiceAsync(endpoint=endpoint, project=project,  
   #                                    token=token, error_handler=my_error_handler)  
     
   # Report Portal versions >= 5.0.0:  
   service =ReportPortalService(endpoint=endpoint, project=project, token=token)  
     
   # Start launch  
   launch = service.start_launch(name=launch_name, description=launch_doc)  
     
   # Add test item and log results (this is just an example, you need to adapt it to your test framework)  
   test_item = service.start_test_item(name="Test Case 1", launch_id=launch.id,type="SUITE")  
   service.log(item_id=test_item.id, log_time=timestamp(), level="INFO", message="Test started")  
   # Simulate test steps and results  
   for i in range(5):  
       step_name =f"Step {i+1}"  
       service.log(item_id=test_item.id, log_time=timestamp(), level="INFO", message=step_name)  
       if i %2==0:  
           service.log(item_id=test_item.id, log_time=timestamp(), level="PASS", message=f"{step_name} passed")  
       else:  
           service.log(item_id=test_item.id, log_time=timestamp(), level="FAIL", message=f"{step_name} failed")  
     
   # Finish test item and launch  
   service.finish_test_item(item_id=test_item.id, status="PASSED")  
   service.finish_launch(id=launch.id, status="FINISHED")
   ```

   请注意，上述代码仅作为示例，您需要根据自己的测试框架和需求进行适当修改。

#### 五、ReportPortal与Robot Framework的集成

Robot Framework是一个流行的自动化测试框架，ReportPortal也支持与其集成。以下是集成步骤：

1. 1. **安装Robot Framework和ReportPortal相关库**：

   ```
   pip install robotframework  
   pip install robotframework-reportportal  
   pip install reportportal-client
   ```
2. 2. **配置Robot Framework**： 在Robot Framework的run界面中，添加ReportPortal的listener。例如：

   ```
   pybot --listener reportportal_listener.ReportPortalListener:endpoint=http://localhost:8080,project=default,token=your_token_here your_test_suite.robot
   ```

   注意：`reportportal_listener.ReportPortalListener`是ReportPortal提供的Python监听器，用于将测试结果推送到ReportPortal。
3. 3. **运行测试**： 使用Robot Framework运行测试，测试结果将自动推送到ReportPortal。