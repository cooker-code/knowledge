---
title: PostgreSQL对象DDL获取工具
author: developerhonor
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzNjYzODMzMQ==&mid=2247484356&idx=1&sn=d72bd14734d027ba885dd583cd81b043&chksm=c3e32ce48e3e26c5c7f02fb580366f8684fc8221f963d16dfa21527d75ddff37315c3bd1e7d3&mpshare=1&scene=24&srcid=1012wlXSQ1ImHZ0INNhrK4QX&sharer_shareinfo=6eea320910a17e961e28c2529dbfc49d&sharer_shareinfo_first=6eea320910a17e961e28c2529dbfc49d#rd
---

请点击微信公众号关注

---

## PostgreSQL对象DDL获取工具

> **关于我**  
>   
> PostgreSQL 分会培训委员会委员  
>   
> PostgreSQL ACE  
>   
> Oracle OCM  
>   
> MySQL OCP 等  
>   
> make progress every day

> **场景导读**  
>   
> 由于 pgAdmin 工具和其它工具比较重，并且查看对象结构DDL需要通过鼠标去点击获取，因此这种繁琐的流程让本人抓狂，所以本人基于 Python Flask 框架开发了 pgobjview。pgobjview 是一款轻量级 PostgreSQL 数据库对象 DDL 生成工具，大小不到 `1MB`，专为数据库管理员和开发人员设计，提供直观的界面来查看和管理 PostgreSQL 数据库中的模式、表、视图等对象。通过简单的浏览器访问方式，用户可以快速了解数据库结构，无需安装复杂的客户端软件。

## 1. 环境要求

* • **操作系统**：Windows 或 Linux 均可部署
* • **Python 版本**：3.6 及以上
* • **网络要求**：有网络环境下可自动安装依赖；无网络环境需提前下载 pip 依赖包
* • **依赖包**： 通过 requirements.txt 安装

## 2. 部署方式

### 2.1 从 git 获取代码

# Windows 环境

```
 PS C:\Users\sungs> cd G:  
PS G:\> cd .\demo\  
PS G:\demo> ls  
PS G:\demo>  git clone https://gitee.com/developerhonor/pgobjview.git  
Cloning into 'pgobjview'...  
remote: Enumerating objects: 33, done.  
remote: Counting objects: 100% (33/33), done.  
remote: Compressing objects: 100% (31/31), done.  
remote: Total 33 (delta 9), reused 0 (delta 0), pack-reused 0 (from 0)  
Receiving objects: 100% (33/33), 101.23 KiB | 1.78 MiB/s, done.  
Resolving deltas: 100% (9/9), done.  
PS G:\demo> cd .\pgobjview\  
PS G:\demo\pgobjview>
```

# Linux 环境

```
[root@server ~]# git clone https://gitee.com/developerhonor/pgobjview.git  
Cloning into 'pgobjview'...  
remote: Enumerating objects: 33, done.  
remote: Counting objects: 100% (33/33), done.  
remote: Compressing objects: 100% (31/31), done.  
remote: Total 33 (delta 9), reused 0 (delta 0), pack-reused 0 (from 0)  
Unpacking objects: 100% (33/33), done.  
[root@server ~]# cd pgobjview
```

### 2.2 创建虚拟环境（推荐）

# Windows 环境

```
 PS G:\demo\pgobjview> python -m venv .venv  
PS G:\demo\pgobjview> dir  
  
  
 目录: G:\demo\pgobjview  
  
  
Mode                 LastWriteTime         Length Name  
----                 -------------         ------ ----  
d-----         2025/9/12     14:16                .venv  
d-----         2025/9/12     14:15                certs  
d-----         2025/9/12     14:15                config  
d-----         2025/9/12     14:15                static  
d-----         2025/9/12     14:15                templates  
-a----         2025/9/12     14:15          72194 pgobjview.py  
-a----         2025/9/12     14:15           2047 README.md  
-a----         2025/9/12     14:15            128 requirements.txt
```

# Linux 环境

```
[root@server pgobjview]# python3 -m venv .venv  
[root@server pgobjview]# ls -a  
.  ..  certs  config  .git  pgobjview.py  README.md  requirements.txt  static  templates  .venv
```

### 2.3 安装依赖

# Window 环境

```
# 先启用虚拟环境  
#Windows 10以上的启用虚拟环境需要在Powershell中执行以下命令  
Start-Process powershell -Verb runAs  
set-executionpolicy remotesigned  回车输入 Y即可  
PS G:\demo\pgobjview> .\.venv\Scripts\activate  
(.venv) PS G:\demo\pgobjview>  
(.venv) PS G:\demo\pgobjview> pip install -r .\requirements.txt  
Looking in indexes: https://pypi.org/simple, https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple  
Collecting Flask>=2.0.3 (from -r .\requirements.txt (line 1))  
  Downloading https://mirrors.tuna.tsinghua.edu.cn/pypi/web/packages/ec/f9/7f9263c5695f4bd0023734af91bedb2ff8209e8de6ead162f35d8dc762fd/flask-3.1.2-py3-none-any.whl (103 kB)  
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 103.3/103.3 kB 1.2 MB/s eta 0:00:00  
Collecting psycopg2-binary>=2.9.3 (from -r .\requirements.txt (line 2))  
  Using cached https://mirrors.tuna.tsinghua.edu.cn/pypi/web/packages/61/69/3b3d7bd583c6d3cbe5100802efa5beacaacc86e37b653fc708bf3d6853b8/psycopg2_binary-2.9.10-cp311-cp311-win_amd64.whl (1.2 MB)  
    
省略部分 ......................................................................................................  
  
Collecting blinker>=1.9.0 (from Flask>=2.0.3->-r .\requirements.txt (line 1))  
  Using cached https://mirrors.tuna.tsinghua.edu.cn/pypi/web/packages/10/cb/f2ad4230dc2eb1a74edf38f1a38b9b52277f75bef262d8908e60d957e13c/blinker-1.9.0-py3-none-any.whl (8.5 kB)  
Collecting click>=8.1.3 (from Flask>=2.0.3->-r .\requirements.txt (line 1))  
  Downloading https://mirrors.tuna.tsinghua.edu.cn/pypi/web/packages/85/32/10bb5764d90a8eee674e9dc6f4db6a0ab47c8c4d0d83c27f7c39ac415a4d/click-8.2.1-py3-none-any.whl (102 kB)  
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 102.2/102.2 kB 5.7 MB/s eta 0:00:00  
Collecting colorama (from click>=8.1.3->Flask>=2.0.3->-r .\requirements.txt (line 1))  
  Using cached https://mirrors.tuna.tsinghua.edu.cn/pypi/web/packages/d1/d6/3965ed04c63042e047cb6a3e6ed1a63a35087b6a609aa3a15ed8ac56c221/colorama-0.4.6-py2.py3-none-any.whl (25 kB)  
Installing collected packages: configparse, psycopg2-binary, MarkupSafe, itsdangerous, colorama, blinker, Werkzeug, Jinja2, click, Flask  
Successfully installed Flask-3.1.2 Jinja2-3.1.6 MarkupSafe-3.0.2 Werkzeug-3.1.3 blinker-1.9.0 click-8.2.1 colorama-0.4.6 configparse-0.1.5 itsdangerous-2.2.0 psycopg2-binary-2.9.10  
  
[notice] A new release of pip is available: 24.0 -> 25.2  
[notice] To update, run: python.exe -m pip install --upgrade pip
```

# Linux 环境

```
[root@server pgobjview]# source .venv/bin/activate  
(.venv) [root@server pgobjview]#   
[root@server pgobjview]# source .venv/bin/activate  
(.venv) [root@server pgobjview]# pip3 install -r requirements.txt   
Collecting Flask>=2.0.3 (from -r requirements.txt (line 1))  
  Using cached https://files.pythonhosted.org/packages/cd/77/59df23681f4fd19b7cbbb5e92484d46ad587554f5d490f33ef907e456132/Flask-2.0.3-py3-none-any.whl  
Collecting psycopg2-binary>=2.9.3 (from -r requirements.txt (line 2))  
  Using cached https://files.pythonhosted.org/packages/9d/3d/5ddb908d2e5fdeb8678470d3f654e987356c9f981867313489b063fbe814/psycopg2-binary-2.9.8.tar.gz  
Collecting itsdangerous>=2.0.1 (from -r requirements.txt (line 3))  
  Using cached https://files.pythonhosted.org/packages/9c/96/26f935afba9cd6140216da5add223a0c465b99d0f112b68a4ca426441019/itsdangerous-2.0.1-py3-none-any.whl  
Collecting Jinja2>=3.0.3 (from -r requirements.txt (line 4))  
  Using cached https://files.pythonhosted.org/packages/20/9a/e5d9ec41927401e41aea8af6d16e78b5e612bca4699d417f646a9610a076/Jinja2-3.0.3-py3-none-any.whl  
Collecting Werkzeug>=2.0.3 (from -r requirements.txt (line 5))  
  Using cached https://files.pythonhosted.org/packages/f4/f3/22afbdb20cc4654b10c98043414a14057cd27fdba9d4ae61cea596000ba2/Werkzeug-2.0.3-py3-none-any.whl  
    
省略部分 ......................................................................................................  
  
  Using cached https://files.pythonhosted.org/packages/fe/ca/75fac5856ab5cfa51bbbcefa250182e50441074fdc3f803f6e76451fab43/dataclasses-0.8-py3-none-any.whl  
Collecting importlib-metadata; python_version < "3.8" (from click>=7.1.2->Flask>=2.0.3->-r requirements.txt (line 1))  
  Using cached https://files.pythonhosted.org/packages/a0/a1/b153a0a4caf7a7e3f15c2cd56c7702e2cf3d89b1b359d1f1c5e59d68f4ce/importlib_metadata-4.8.3-py3-none-any.whl  
Collecting zipp>=0.5 (from importlib-metadata; python_version < "3.8"->click>=7.1.2->Flask>=2.0.3->-r requirements.txt (line 1))  
  Using cached https://files.pythonhosted.org/packages/bd/df/d4a4974a3e3957fd1c1fa3082366d7fff6e428ddb55f074bf64876f8e8ad/zipp-3.6.0-py3-none-any.whl  
Collecting typing-extensions>=3.6.4; python_version < "3.8" (from importlib-metadata; python_version < "3.8"->click>=7.1.2->Flask>=2.0.3->-r requirements.txt (line 1))  
  Using cached https://files.pythonhosted.org/packages/45/6b/44f7f8f1e110027cf88956b59f2fad776cca7e1704396d043f89effd3a0e/typing_extensions-4.1.1-py3-none-any.whl  
Installing collected packages: MarkupSafe, Jinja2, zipp, typing-extensions, importlib-metadata, click, itsdangerous, dataclasses, Werkzeug, Flask, psycopg2-binary, configparse  
  Running setup.py install for psycopg2-binary ... done  
Successfully installed Flask-2.0.3 Jinja2-3.0.3 MarkupSafe-2.0.1 Werkzeug-2.0.3 click-8.0.4 configparse-0.1.5 dataclasses-0.8 importlib-metadata-4.8.3 itsdangerous-2.0.1 psycopg2-binary-2.9.8 typing-extensions-4.1.1 zipp-3.6.0  
You are using pip version 9.0.3, however version 25.2 is available.  
You should consider upgrading via the 'pip install --upgrade pip' command
```

### 2.4.编辑配置文件

当前配置文件仅包含了指定的端口和 ssl 认证两部分  
关于证书可以配置绝对路径，也可以使用相对路径，不做严格要求，本文以绝对路径配置。

# Windows 环境

配置文件编辑如下

```
(.venv) PS G:\demo\pgobjview> type .\config\config.ini  
[PORT]  
port = 7363  
[SSL]  
server_crt = G:/demo/pgobjview/certs/pgobjview.com.crt  
server_key = G:/demo/pgobjview/certs/pgobjview.com.key
```

# Linux 环境

配置文件编辑如下

```
(.venv) [root@server pgobjview]# cat config/config.ini   
[PORT]  
port = 7363  
[SSL]  
server_crt = /root/pgobjview/certs/pgobjview.com.crt  
server_key = /root/pgobjview/certs/pgobjview.com.key
```

# 关于如何生成证书

证书生成请访问我的另一个项目 `easyca`，地址为: easyca项目  
附下载方式

```
git clone https://gitee.com/developerhonor/easyca.git
```

## 3. 启用应用

# Windows 环境

```
(.venv) PS G:\demo\pgobjview> python .\pgobjview.py  
[2025-09-12 14:38:04,052] [INFO] PosgreSQL对象结构获取工具 starting in NORMAL 模式  
[2025-09-12 14:38:04,052] [INFO] 从 G:\demo\pgobjview\config\config.ini 文件中读取 SSL 配置  
[2025-09-12 14:38:04,052] [INFO] 规范化 cert 文件路径: G:\demo\pgobjview\certs\pgobjview.com.crt  
[2025-09-12 14:38:04,052] [INFO] 规范化 key  文件路径: G:\demo\pgobjview\certs\pgobjview.com.key  
[2025-09-12 14:38:04,052] [INFO] 检查服务器证书路径: G:\demo\pgobjview\certs\pgobjview.com.crt, 存在: True  
[2025-09-12 14:38:04,052] [INFO] 检查服务器私钥路径: G:\demo\pgobjview\certs\pgobjview.com.key,  存在: True  
[2025-09-12 14:38:04,059] [INFO] 创建可兼容 Windows 的 SSL 上下文成功  
[2025-09-12 14:38:04,059] [INFO] 服务器将以 HTTPS 模式启动  
[2025-09-12 14:38:04,059] [INFO] 从 G:\demo\pgobjview\config\config.ini 配置文件中读取端口 7363  
[2025-09-12 14:38:04,070] [INFO] 服务器主机名: Devhonor  
[2025-09-12 14:38:04,070] [INFO] 可用的 IP 地址: 10.10.20.107, 10.10.20.116, 10.10.20.2, 192.168.10.1, 192.168.47.1  
[2025-09-12 14:38:04,070] [INFO] 服务器监听所有网络地址 (0.0.0.0:7363)  
[2025-09-12 14:38:04,074] [INFO] 访问标识: HTTPS://10.10.20.107:7363 or HTTPS://10.10.20.116:7363 or HTTPS://10.10.20.2:7363 or HTTPS://192.168.10.1:7363 or HTTPS://192.168.47.1:7363  
 * Serving Flask app 'pgobjview'  
 * Debug mode: off  
[2025-09-12 14:38:04,509] [INFO] WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.  
 * Running on all addresses (0.0.0.0)  
 * Running on https://127.0.0.1:7363  
 * Running on https://10.10.20.107:7363  
[2025-09-12 14:38:04,511] [INFO] Press CTRL+C to quit
```

# Linux 环境

```
(.venv) [root@server pgobjview]# python3 pgobjview.py   
[2025-09-12 14:38:36,144] [INFO] PosgreSQL对象结构获取工具 starting in NORMAL 模式  
[2025-09-12 14:38:36,145] [INFO] 从 /root/pgobjview/config/config.ini 文件中读取 SSL 配置  
[2025-09-12 14:38:36,145] [INFO] 规范化 cert 文件路径: /root/pgobjview/certs/pgobjview.com.crt  
[2025-09-12 14:38:36,145] [INFO] 规范化 key  文件路径: /root/pgobjview/certs/pgobjview.com.key  
[2025-09-12 14:38:36,145] [INFO] 检查服务器证书路径: /root/pgobjview/certs/pgobjview.com.crt, 存在: True  
[2025-09-12 14:38:36,145] [INFO] 检查服务器私钥路径: /root/pgobjview/certs/pgobjview.com.key,  存在: True  
[2025-09-12 14:38:36,145] [INFO] 创建可兼容 Windows 的 SSL 上下文成功  
[2025-09-12 14:38:36,145] [INFO] 服务器将以 HTTPS 模式启动  
[2025-09-12 14:38:36,145] [INFO] 从 /root/pgobjview/config/config.ini 配置文件中读取端口 7363  
[2025-09-12 14:38:41,205] [INFO] 服务器主机名: server  
[2025-09-12 14:38:41,206] [INFO] 可用的 IP 地址: 10.10.20.31, 172.17.0.1, 172.18.0.1, 192.168.47.219, 192.168.47.220, 192.168.47.221  
[2025-09-12 14:38:41,206] [INFO] 服务器监听所有网络地址 (0.0.0.0:7363)  
[2025-09-12 14:38:41,206] [INFO] 访问标识: HTTPS://10.10.20.31:7363 or HTTPS://172.17.0.1:7363 or HTTPS://172.18.0.1:7363 or HTTPS://192.168.47.219:7363 or HTTPS://192.168.47.220:7363 or HTTPS://192.168.47.221:7363  
 * Serving Flask app 'pgobjview' (lazy loading)  
 * Environment: production  
   WARNING: This is a development server. Do not use it in a production deployment.  
   Use a production WSGI server instead.  
 * Debug mode: off  
[2025-09-12 14:38:51,300] [WARNING]  * Running on all addresses.  
   WARNING: This is a development server. Do not use it in a production deployment.  
[2025-09-12 14:38:51,300] [INFO]  * Running on https://192.168.47.219:7363/ (Press CTRL+C to quit)
```

## 4. 使用方式

### 4.1.浏览器输入地址

该应用有两种获取，第一种是选择性获取，主要针对对象数量少的情况，第二种为搜索获取，主要针对对象数量非常多的情况。

我这里使用的是部署在 `Linux` 服务器地址

`Windows` 和 `Linux`都通过应用主机所在的 `IP` 地址访问，由于内容一样，这里仅仅演示 `Linux` 下的访问。我这里的环境有 `PostgreSQL V16` 和 `PostgreSQL V17` 两个版本，这两个都验证没有问题，这里就以17版本为演示。

### 登录

### 填写数据库连接信息

点击连接

### 首页面

### 4.2. 使用选择式获取DDL

### 获取表结构DDL

### 获取表上的索引DDL

在上一步选择了表后，如果想要获取表上的索引 DDL ，点击获取索引DDL即可

### 获取带有外键的表的DDL

### 获取视图DDL

### 获取物化视图DDL

### 获取序列DDL

### 获取类型DDL

### 获取函数DDL

### 获取存储过程DDL

### 获取触发器DDL

### 4.3. 使用搜索式获取DDL

搜索式获取，主要是需要知道对象名称

### 搜索函数并生成DDL

这里有重载或者重写的函数，因此可以根据搜索出来的结果选择想要获取的函数的DDL  
  
以下为函数重载获取演示

#### 第一个函数

#### 第二个函数

### 搜索带有外键约束的表的DDL

---

#### 内容涉及推荐访问

* • easyca工具gitee地址

---