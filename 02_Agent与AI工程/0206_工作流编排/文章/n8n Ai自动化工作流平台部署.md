---
title: n8n Ai自动化工作流平台部署
author: 微茫不朽
date: 
url: https://mp.weixin.qq.com/s?__biz=MzU2NzY2Mzg1Nw==&mid=2247487737&idx=1&sn=24378602419df99c9974e5975257e327&chksm=fde8f7449aa13c68fa74fe70a2cce614ec552b0115adcb206f86e6211f99034c9bc86e4d5edd&mpshare=1&scene=24&srcid=0925L5mujX5FAEQwhPJMpRw1&sharer_shareinfo=ef19bf68a2fbd188b3375490042ab434&sharer_shareinfo_first=ef19bf68a2fbd188b3375490042ab434#rd
---

# 

# 上图该流程还在测试节点，下期详解

## unsetunset🐳 一、Docker 镜像加速配置（中国大陆优化）unsetunset

为提升 Docker 镜像拉取速度，推荐配置国内镜像源。以下为完整镜像源配置文件，涵盖主流高校、云厂商及社区镜像站。

### ✅ 1. 编辑 Docker Daemon 配置文件

```
sudo vi /etc/docker/daemon.json
```

> **操作提示**：
>
> * 按 `Insert` 键进入编辑模式
> * 粘贴下方 JSON 内容
> * 按 `ESC` → 输入 `:wq` 保存退出

### ✅ 2. 配置镜像源（推荐多源冗余）

```
{  
    "registry-mirrors": [  
        "https://2a6bf1988cb6428c877f723ec7530dbc.mirror.swr.myhuaweicloud.com",  
        "https://docker.m.daocloud.io",  
        "https://hub-mirror.c.163.com",  
        "https://mirror.baidubce.com",  
        "https://your_preferred_mirror",  
        "https://dockerhub.icu",  
        "https://docker.registry.cyou",  
        "https://docker-cf.registry.cyou",  
        "https://dockercf.jsdelivr.fyi",  
        "https://docker.jsdelivr.fyi",  
        "https://dockertest.jsdelivr.fyi",  
        "https://mirror.aliyuncs.com",  
        "https://dockerproxy.com",  
        "https://mirror.baidubce.com",  
        "https://docker.m.daocloud.io",  
        "https://docker.nju.edu.cn",  
        "https://docker.mirrors.sjtug.sjtu.edu.cn",  
        "https://docker.mirrors.ustc.edu.cn",  
        "https://mirror.iscas.ac.cn",  
        "https://docker.rainbond.cc",  
        "https://docker.m.daocloud.io",  
        "https://docker.1panel.live",  
        "https://hub.rat.dev"  
    ]  
}
```

> ⚠️ 注意：部分镜像可能偶尔不稳定，建议保留 3~5 个常用源即可，如：
>
> * 阿里云 `https://mirror.aliyuncs.com`
> * DaoCloud `https://docker.m.daocloud.io`
> * 网易 `https://hub-mirror.c.163.com`
> * 中科大 `https://docker.mirrors.ustc.edu.cn`

### ✅ 3. 重启 Docker 服务

```
sudo systemctl daemon-reload  
sudo systemctl restart docker  
# 或使用旧版命令  
sudo service docker restart
```

---

## unsetunset🚀 二、使用 Docker 部署 n8nunsetunset

n8n 是一个开源、可扩展的工作流自动化工具，支持数百个应用的集成。

### ✅ 1. 启动 n8n 容器（推荐持久化配置）

```
docker run -d \  
  --name n8n \  
  --restart unless-stopped \  
  -p 5678:5678 \  
  -v n8n_data:/home/node/.n8n \  
  -e N8N_SECURE_COOKIE=false \  
  -e N8N_HOST=your-domain.com \  
  -e N8N_PROTOCOL=https \  
  -e N8N_PORT=5678 \  
  -e WEBHOOK_URL=https://your-domain.com/ \  
  n8nio/n8n
```

#### 🔧 环境变量说明：

| 环境变量 | 说明 |
| --- | --- |
| `N8N_SECURE_COOKIE` | 关闭安全 Cookie（开发环境用） |
| `N8N_HOST` | 指定访问域名（用于 webhook 回调） |
| `N8N_PROTOCOL` | 协议：http / https |
| `N8N_PORT` | 服务端口（默认 5678） |
| `WEBHOOK_URL` | 外部可访问的 webhook 地址（必须配置 HTTPS） |

### ✅ 2. 管理容器

```
# 停止容器  
docker stop n8n  
  
# 启动容器  
docker start n8n  
  
# 查看日志  
docker logs -f n8n  
  
# 进入容器  
docker exec -it n8n /bin/bash
```

### ✅ 3. 访问验证

浏览器访问：`http://<服务器IP>:5678`

首次访问需注册管理员账号：

* Email
* First Name
* Password

成功后进入仪表盘：

---

## unsetunset🌐 三、汉化支持（中文界面）unsetunset

官方未提供中文界面，社区提供汉化包：

### ✅ 1. 下载汉化资源

GitHub 项目地址：

```
https://github.com/other-blowsnow/n8n-i18n-chinese
```

### ✅ 2. 手动替换语言文件（进阶用户）

1. 进入容器：

   ```
   docker exec -it n8n /bin/bash
   ```
2. 替换语言文件（路径示例）：

   ```
   cd /usr/local/lib/node_modules/n8n/packages/editor-ui/src/plugins/i18n/locales
   ```
3. 下载并替换 `zh-CN.json`（需自行构建或从社区项目获取）

> ⚠️ 注意：升级 n8n 版本后汉化可能失效，建议关注项目更新或使用反向代理注入翻译脚本。

---

## unsetunset🧩 四、n8n 核心节点详解（Node Usage）unsetunset

n8n 的工作流由“节点（Nodes）”组成，每个节点代表一个操作或服务。

---

### 📌 1. HTTP Request 节点

用于调用 REST API。

#### ⚙️ 配置参数：

* **Method**: GET / POST / PUT / DELETE
* **URL**: 接口地址
* **Authentication**: Basic / Bearer / OAuth2 / None
* **Headers**: 自定义请求头
* **Body**: JSON / Form / Binary
* **Options**: Timeout, Redirects, Proxy 等

#### 🧪 示例：获取天气数据

```
{  
  "url": "https://api.weatherapi.com/v1/current.json",  
  "method": "GET",  
  "qs": {  
    "key": "YOUR_API_KEY",  
    "q": "Beijing"  
  }  
}
```

---

### 📌 2. Webhook 节点

用于接收外部系统触发（如 GitHub、微信、钉钉回调）。

#### ⚙️ 配置：

* **Webhook URL**: 自动生成或自定义路径
* **HTTP Method**: GET / POST
* **Response Mode**:

+ `On Received`：立即响应
+ `Last Node`：工作流结束后响应

* **Response Data**: 返回 JSON / HTML / Text

> ⚠️ 生产环境必须配置 `WEBHOOK_URL` 环境变量，确保外部可访问。

---

### 📌 3. Function 节点（JavaScript）

自定义逻辑处理，支持完整 JS 语法 + n8n 内置 API。

#### 🧩 内置对象：

* `$input`：获取上游节点数据
* `$json`：当前 item 的 JSON 数据
* `$itemIndex`：当前 item 索引
* `$workflow`：工作流信息
* `$now`：当前时间戳

#### 🧪 示例：数据转换

```
return {  
  json: {  
    fullName: $input.item.json.firstName + " " + $input.item.json.lastName,  
    timestamp: $now,  
    processed: true  
  }  
}
```

---

### 📌 4. Schedule Trigger 节点

定时触发工作流（类似 Cron）。

#### ⚙️ 配置：

* **Cron Expression**: `0 9 * * 1`（每周一 9 点）
* **Timezone**: 设置时区（如 `Asia/Shanghai`）
* **Trigger Times**: 指定具体时间点（可多个）

> 支持可视化 Cron 编辑器，适合非技术人员。

---

### 📌 5. IF 节点（条件判断）

根据条件分流执行路径。

#### ⚙️ 配置：

* **Condition Type**: Basic / Expression
* **Basic Mode**:

+ Value 1 / Operator / Value 2
+ 如：`{{ $json.status }}``equals``"success"`

* **Expression Mode**:

+ 使用 JS 表达式：`$json.amount > 100 && $json.currency === 'CNY'`

---

### 📌 6. Spreadsheet File 节点

读写 Excel/CSV 文件（本地或 S3/Google Drive）。

#### ⚙️ 支持操作：

* 读取工作表 → 输出为 JSON 数组
* 写入数据 → 生成新文件
* 支持公式、样式（高级版）

#### 🧪 示例：读取 CSV

```
{  
  "file": "data.csv",  
  "options": {  
    "headerRow": true,  
    "sheetName": "Sheet1"  
  }  
}
```

---

### 📌 7. Email 节点（SMTP / IMAP）

发送或接收邮件。

#### ⚙️ 发送邮件配置：

* SMTP Host / Port
* Auth: User + Password / OAuth2
* From / To / Subject / Body
* 支持附件、HTML 模板

#### 🧪 示例模板：

```
<h3>订单通知</h3>  
<p>订单号：{{ $json.orderId }}</p>  
<p>金额：{{ $json.amount }} 元</p>
```

---

### 📌 8. Google Sheets / Airtable / Notion 节点

连接主流 SaaS 平台。

#### Google Sheets 示例操作：

* Append Row
* Update Row
* Get Rows (支持 filter / sort)
* Clear Range

> 需先配置 OAuth2 凭证（n8n 有向导）

---

### 📌 9. Code 节点（Python / Shell / Go）

企业版支持，可执行任意语言脚本。

#### Python 示例：

```
import json  
data = json.loads(input)  
data['processed'] = True  
print(json.dumps(data))
```

---

### 📌 10. Merge 节点 & Split In Batches

* **Merge**: 合并多个输入流
* **Split In Batches**: 将长列表分批处理（防 API 限流）

#### 🧪 Split 示例：

* Batch Size: 10
* 每批 10 条数据，依次触发下游

---

## unsetunset🛠 五、最佳实践与技巧unsetunset

### ✅ 1. 使用环境变量管理密钥

```
-e N8N_ENCRYPTION_KEY=your-32-char-key  
-e DB_TYPE=postgresdb  
-e DB_POSTGRESDB_DATABASE=n8n
```

### ✅ 2. 启用执行日志（调试用）

```
-e N8N_LOG_LEVEL=debug
```

### ✅ 3. 使用 PostgreSQL 替代 SQLite（生产推荐）

```
-e DB_TYPE=postgresdb  
-e DB_POSTGRESDB_HOST=your-pg-host  
-e DB_POSTGRESDB_PORT=5432  
-e DB_POSTGRESDB_DATABASE=n8n  
-e DB_POSTGRESDB_USER=user  
-e DB_POSTGRESDB_PASSWORD=pass
```

### ✅ 4. 配置反向代理（Nginx + HTTPS）

```
server {  
    listen 443 ssl;  
    server_name n8n.yourdomain.com;  
  
    location / {  
        proxy_pass http://localhost:5678;  
        proxy_set_header Host $host;  
        proxy_set_header X-Real-IP $remote_addr;  
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  
        proxy_set_header X-Forwarded-Proto $scheme;  
    }  
}
```

---

## unsetunset🧭 六、学习资源推荐unsetunset

* 官方文档：https://docs.n8n.io
* 节点库：https://n8n.io/integrations
* 社区模板：https://n8n.io/workflows
* 中文教程：https://www.bilibili.com/search?keyword=n8n
* GitHub 汉化：https://github.com/other-blowsnow/n8n-i18n-chinese

---

> 💡 提示：n8n 支持自定义节点开发，适合企业深度集成。可通过 `n8n-nodes-base` 模板快速开发。

---