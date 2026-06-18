> 已吸收至：[[07_工程与架构/0701_后端架构/070103_Python/070103_核心知识点/Python来源校准与降权准则|Python来源校准与降权准则]]、[[07_工程与架构/0701_后端架构/070103_Python/070103_知识地图|070103_Python知识地图]]

---
title: FastAPI 5分钟上手：写出高性能 Python 接口（第1篇）
author: D先生的自白
date:
url: https://mp.weixin.qq.com/s?__biz=MzYzNTE0NDIyMg==&mid=2247483673&idx=1&sn=ef78b4e8f0132cbd50bc991f61e1a3ea&chksm=f16365e53e21726e72c87e27328bf11e309070f7511c16b3762cfe34abe0b4768aab0811a19c&mpshare=1&scene=24&srcid=1115dnTXaoDstonmGXZVQwoI&sharer_shareinfo=1a75d414718206cf45977f7916e37c3f&sharer_shareinfo_first=1a75d414718206cf45977f7916e37c3f#rd
---

> 配环境太繁琐、样板代码太多、文档总不同步？试试 FastAPI——一款为高效、可维护的现代 API 而生的 Python 框架。

## 🎯 阅读收获

* 🚀 5分钟跑通你的第一个 FastAPI 应用
* ⚡ 了解 FastAPI 为什么快、为什么省心
* 🧪 掌握本地启动、测试与交互式文档的使用

## 🚀 为什么是 FastAPI？

* ⚡ 性能优秀：基于 `Starlette` + `Uvicorn`，异步原生支持
* ⏱️ 开发效率高：类型提示即校验，自动生成 Swagger / ReDoc 文档
* 🔧 维护更省心：强类型带来更好的 IDE 补全与重构体验
* 🌱 生态友好：与 Pydantic、SQLModel、Celery 等配合顺畅

## 🧰 准备环境

建议使用 Conda 管理 Python 环境：

```
miniconda3 25.7.0
python 3.12.0
FastAPI 0.116.1
uvicorn 0.35.0
```

创建并切换到独立环境：

```
conda create --name fastapi-env python=3.12.0
conda activate fastapi-env
```

安装 FastAPI 与 Uvicorn（ASGI 服务器）：

```
conda install -n fastapi-env fastapi=0.116.1 uvicorn=0.35.0
```

💡 提示：如果你已在 `fastapi-env` 环境中，`-n fastapi-env` 可省略；也可用 `pip install fastapi uvicorn` 达到同样效果。

## 🧪 第一个 FastAPI 应用（不到 20 行）

在项目根目录创建 `main.py`：

```
from fastapi import FastAPI

# 创建 FastAPI 实例
app = FastAPI()

# 根路径：返回问候消息
@app.get("/")
def read_root():
    return {"message": "Hello World from FastAPI"}

# 带参数的路径：演示路径参数与可选查询参数
@app.get("/items/{item_id}")
def read_item(item_id: int, q: str | None = None):
    return {"item_id": item_id, "q": q}
```

* `/`：返回简单 JSON 信息
* `/items/{item_id}`：接收路径参数 `item_id` 与可选查询参数 `q`

## ▶️ 运行与测试

### 🟢 启动服务器

```
uvicorn main:app --reload
```

* `main`：Python 文件名（`main.py`）
* `app`：FastAPI 实例名
* `--reload`：代码变更自动重载（开发模式）

默认访问地址：`http://127.0.0.1:8000`

### 🧭 三种测试方式

1. 🌐 浏览器直达：

* 访问 `http://127.0.0.1:8000`，看到 `{"message": "Hello World from FastAPI"}`
* 访问 `http://127.0.0.1:8000/items/5?q=test` 查看参数效果

2. 📘 交互式文档（强烈推荐）：

* 打开 `http://127.0.0.1:8000/docs`
* Swagger UI 自动生成，可在线调试所有接口

3. 🧩 命令行 `curl`：

```
curl -X GET "http://127.0.0.1:8000/"
curl -X GET "http://127.0.0.1:8000/items/5?q=test"
```

## 🆘 常见坑位速查

* 🔌 端口占用：修改启动命令为 `uvicorn main:app --reload --port 8001`
* 📦 导入报错：确认当前目录下存在 `main.py`，且以 `main:app` 启动
* 🧭 环境混乱：用 `conda info --envs` 或 `which python` 检查当前 Python 来源

## 🍱 练习加餐（动手更快熟）

* ➕ 添加新端点：`/users/{user_id}`，返回用户 ID 和欢迎消息
* 📝 扩展现有端点：为 `/items/{item_id}` 增加可选查询参数 `description`
* 🔁 多种方式测试：浏览器、Docs、curl 都试一遍

## ✅ 小结

你已完成 FastAPI 的初体验：

* 🧰 准备并隔离了开发环境
* 🚀 创建了第一个应用并成功运行
* 🧭 掌握了浏览器、交互式文档与命令行三种测试方式

📣 明天我们将学习 FastAPI 的请求与响应模型，敬请期待！