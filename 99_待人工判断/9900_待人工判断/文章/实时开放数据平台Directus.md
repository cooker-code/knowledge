---
title: 实时开放数据平台Directus
author: 各种折腾
date: 
url: http://mp.weixin.qq.com/s?__biz=MzI1NTQxMzc3MA==&mid=2247499840&idx=1&sn=3293f8c4c440878892ab68139bd03639&chksm=ea34db25dd435233eaebdbf2bbbf411a22c6b2150943492cb473416d8dbb3795512c11115a84&mpshare=1&scene=24&srcid=1214NjCRzhBQp9unWLt8ytO3&sharer_shareinfo=0eaf95e00891c339987f07462237f423&sharer_shareinfo_first=0eaf95e00891c339987f07462237f423#rd
---

**什么是 Directus ？**

> `Directus` 是一个实时 `API`和应用程序仪表板，用于管理 `SQL` 数据库内容。该平台为您团队中的每个人，无论其技术技能如何，为任何数据模型或项目提供平等的数据访问和数字文件资产管理。首先，将 `Directus` 链接到所需的 `SQL` 数据库和文件存储适配器。之后，`Directus` 使您能够执行 `CRUD` 操作、创建用户、分配具有完全可配置权限的角色、构建复杂而精细的查询、配置事件驱动的 `Webhook` 和任务自动化……

# 安装

在群晖上以 Docker 方式安装。

在注册表中搜索 `directus` ，选择第一个 `directus/directus`，版本选择 `latest`。

> 本文写作时， `latest` 版本对应为  `10.6.3`；

## 卷

在 `docker` 文件夹中，创建一个新文件夹 `directus`，并在其中建两个子文件夹 `database`  和 `uploads`

记得要赋予 `database` 目录 `Everyone`读写权限

| 文件夹 | 装载路径 | 说明 |
| --- | --- | --- |
| `docker/directus/database` | `/directus/database` | 存放数据库 |
| `docker/directus/uploads` | `/directus/uploads` | 存放上传文件 |

## 端口

本地端口不冲突就行，不确定的话可以用命令查一下

```
# 查看端口占用  
netstat -tunlp | grep 端口号
```

| 本地端口 | 容器端口 |
| --- | --- |
| `8055` | `8055` |

## 环境

| 可变 | 值 |
| --- | --- |
| `KEY` | 可以用 `openssl rand -base64 32` 生成 |
| `SECRET` | 可以用 `openssl rand -base64 32` 生成 |
| `ADMIN_EMAIL` | 管理员账号，要用邮件 |
| `ADMIN_PASSWORD` | 管理员密码 |
| `DB_CLIENT` | 数据库类型，用了 `SQLite` |
| `DB_FILENAME` | 指定 `SQLite` 文件名 |
| `WEBSOCKETS_ENABLED` | 启用 `Websocket` |

# 命令行安装

如果你熟悉命令行，可能用 `docker cli` 更快捷

```
# 新建文件夹 directus 和 子目录  
mkdir -p /volume1/docker/directus/{database,uploads}  
  
# 进入 directus 目录  
cd /volume1/docker/directus  
  
# 修改目录权限   
chmod 777 database  
  
# 运行容器  
docker run -d \  
   --restart unless-stopped \  
   --name directus \  
   -p 8055:8055 \  
   -v $(pwd)/database:/directus/database \  
   -v $(pwd)/uploads:/directus/uploads \  
   -e KEY="bJdYXhXB/dkKts6TEbPlWF+HKEuf3/lbFCdJhM6mMgg=" \  
   -e SECRET="k3Do33NXRyhudvQrRJfaj3eHYJFiGVoRIFJRvDSlZug=" \  
   -e ADMIN_EMAIL="wbsu2003@hotmail.com" \  
   -e ADMIN_PASSWORD="9RFhyk96" \  
   -e DB_CLIENT="sqlite3" \  
   -e DB_FILENAME="/directus/database/data.db" \  
   -e WEBSOCKETS_ENABLED="true" \  
   directus/directus:latest
```

也可以用 `docker-compose` 安装，将下面的内容保存为 `docker-compose.yml` 文件

```
version: '3'  
  
services:  
  directus:  
    image: directus/directus:latest  
    container_name: directus  
    ports:  
      - 8055:8055  
    volumes:  
      - ./database:/directus/database  
      - ./uploads:/directus/uploads  
    environment:  
      KEY: 'bJdYXhXB/dkKts6TEbPlWF+HKEuf3/lbFCdJhM6mMgg='  
      SECRET: 'k3Do33NXRyhudvQrRJfaj3eHYJFiGVoRIFJRvDSlZug='  
      ADMIN_EMAIL: 'wbsu2003@hotmail.com'  
      ADMIN_PASSWORD: '9RFhyk96'  
      DB_CLIENT: 'sqlite3'  
      DB_FILENAME: '/directus/database/data.db'  
      WEBSOCKETS_ENABLED: 'true'
```

然后执行下面的命令

```
# 新建文件夹 directus 和 子目录  
mkdir -p /volume1/docker/directus/{database,uploads}  
  
# 进入 directus 目录  
cd /volume1/docker/directus  
  
# 修改目录权限   
chmod 777 database  
  
# 将 docker-compose.yml 放入当前目录  
  
# 一键启动  
docker-compose up -d
```

# 运行

在浏览器中输入 `http://群晖IP:8055` 就能看到登录界面

登录成功后

## 设置中文

`Settings` --> `Project Settings` --> `Default Language` 下来找到 `Chinese（Simplified）`

设置完成后，点右上角的 `√`，生效后就是中文了

可能你也注意到了，还有一些依然是英文，刷新下就好了

## 数据模型

`数据模型` --> `+`

新建数据模型，例如 `test`

## 创建字段

再建一个字段

例如 `text`

现在就有两个字段了

## 创建条目

进入 `内容`，目前还没有数据

点 `创建条目` 开始添加数据

再来一条

现在我们有 `2` 条数据了

## 设置角色和权限

`Directus` 带有两个内置角色：`Public` 和 `Admin`

* `Public` 关闭了所有权限，并且可以通过完全精细的控制重新配置，以准确地公开您希望未经身份验证的用户看到的内容。
* `Admin` 角色具有完全权限，并且无法更改。
* 除了这些内置角色之外，还可以创建任意数量的新角色，所有角色都具有完全自定义的细粒度权限。

点 `公开`，默认权限都是禁止的

老苏都改成了允许

## 访问 API

打开`http://群晖IP:8055/items/test`，你会看到我们录入的两条数据

当然你也可以用 `API` 工具，例如客户端工具 `Apifox`

或者网页工具 `Yaade`

当然 `Directus` 的功能远不止这些，详细使用可以看官方的参考指南：https://docs.directus.io/getting-started/quickstart.html

或者你觉得英文看着费劲，也可以去看看中文的手册：https://ezdoc.cn/docs/directus

# 参考文档

> directus/directus: The Modern Data Stack 🐰 — Directus is an instant REST+GraphQL API and intuitive no-code data collaboration app for any SQL database.  
> 地址：https://github.com/directus/directus
>
> The Backend to Build Anything or Everything | Directus  
> 地址：https://directus.io/
>
> Directus Docs | Directus Docs  
> 地址：https://docs.directus.io/
>
> Self-Hosting Quickstart | Directus Docs  
> 地址：https://docs.directus.io/self-hosted/quickstart.html
>
> 首页 - Directus v10.6.3 中文开发文档手册 - 无头CMS - 中文手册  
> 地址：https://ezdoc.cn/docs/directus/

@所有人：写文不易，如果你都看到了这里，请点个`赞`和`在看`，分享给更多的朋友；为确保你能收到每一篇文章，请主页右上角设置星标。