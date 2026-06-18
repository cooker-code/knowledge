> 已吸收至：[[07_工程与架构/0701_后端架构/070103_Python/070103_核心知识点/FastAPI生产边界与降权准则|FastAPI生产边界与降权准则]]、[[07_工程与架构/0701_后端架构/070103_Python/070103_核心知识点/Python架构实现路线|Python架构实现路线]]
---
title: 通用的 FastAPI 基础镜像到底该怎么构建并共享
author: AI程序员张总
date:
url: https://mp.weixin.qq.com/s?__biz=MzA5NjQ1ODYwNg==&mid=2659091995&idx=1&sn=19dbc25f1aafa05602c80e030a6b059e&chksm=8a3a6762c430be7474082e33c9c866b86a9d982777ffc5ebc130305f336f02cc7496ce08d333&mpshare=1&scene=24&srcid=0129SXu2S37sGzuweLiNhQZr&sharer_shareinfo=2a227f3bbed5e37592682ac7b418224d&sharer_shareinfo_first=2a227f3bbed5e37592682ac7b418224d#rd
---

在做 FastAPI 项目 Docker 化时，最容易被低估的一件事是：**每个项目都在重复搭同一套 Python + FastAPI 运行环境。**

Python 版本不统一、pip 行为不一致、国内环境下依赖下载慢、 Dockerfile 到处复制粘贴—— 单个项目问题不明显，但只要项目数量一多、开始跑 CI，这些问题就会集中爆发。

工程上更稳妥的做法是：**先构建一个通用的 FastAPI 基础镜像，把所有项目都会用到、但又不属于业务的内容提前固化。**

这篇文章只解决一个问题：**如何构建一个可长期复用的 FastAPI 基础镜像，并把它推送到远程仓库，供团队统一使用。**

## 一、什么样的 FastAPI 镜像才配叫“基础镜像”

基础镜像不是“能跑 FastAPI”就够了，它必须有清晰的工程边界。

一个合格的 FastAPI 基础镜像，应当具备这些特征：

* Python 版本明确、固定
* 已包含 FastAPI 运行所需的基础依赖
* pip 源已为国内环境做过优化
* 不包含任何业务代码
* 不定义具体应用启动命令
* 更新频率低，可被多个项目长期复用

它**不应该**包含：

* 具体 FastAPI 项目的源码
* requirements.txt / poetry.lock
* 数据库配置
* 应用级 CMD 或 ENTRYPOINT

一句话总结：

**基础镜像解决的是“怎么跑 FastAPI”，而不是“跑哪个 FastAPI”。**

## 二、准备基础镜像的构建目录

基础镜像必须和业务项目彻底解耦，目录越简单越好。

```
fastapi_base_image/
├── Dockerfile
```

这里只需要一个 Dockerfile，不需要任何 Python 源码。

## 三、选择合适的 Python 基础镜像

在工程实践中，FastAPI 基础镜像通常基于官方 Python 镜像构建。

推荐原则：

* 使用明确版本号
* 优先使用 `slim` 变体
* 避免 `latest`

示例选择：

```
python:3.11.9-slim
```

这个版本在体积、兼容性、生态成熟度之间比较平衡，非常适合作为长期基础环境。

## 四、编写通用 FastAPI 基础镜像 Dockerfile

下面是一份**工程可直接长期使用**的 Dockerfile，所有关键步骤都有明确注释。

```
# 使用明确版本的 Python 官方镜像
FROM python:3.11.9
#FROM python:3.11.9-slim

# 设置容器内统一工作目录
WORKDIR /app

# 配置 pip 国内镜像源
# 提升国内环境下依赖安装的稳定性
# 升级 pip，避免旧版本带来的兼容问题
# 安装 FastAPI 运行所需的基础依赖，这里只放“所有项目都会用到的内容”
# 验证 Python 与关键依赖版本
RUN pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple && \
    pip install --upgrade pip && \
    pip install fastapi==0.110.0 && \
    pip install uvicorn==0.29.0 && \
    python --version && pip --version

# 基础镜像不定义 CMD
# 具体项目自行决定如何启动应用
```

这份 Dockerfile 的工程特征非常明确：

* Python 版本固定
* FastAPI 与 Uvicorn 作为通用运行依赖
* 国内环境友好
* 不掺杂任何业务逻辑

## 五、本地构建 FastAPI 基础镜像

在 `fastapi` 目录中执行构建。

以下命令在 **PowerShell 和 Bash 中完全一致**：

```
docker build -t fastapi:python3.11.9 .
docker build -t fastapi:python3.11.9-slim .
```

构建完成后，验证镜像是否存在：

```
docker images
```

此时，这个镜像已经可以作为所有 FastAPI 项目的统一基础环境。

## 六、为基础镜像打可共享标签

要推送到远程仓库，镜像名必须包含用户名。

假设你的 Docker Hub 用户名是 `zhangdapeng`：

```
docker tag fastapi:python3.11.9 zhangdapeng520/fastapi:python3.11.9
docker tag fastapi:python3.11.9-slim zhangdapeng520/fastapi:python3.11.9-slim
```

命名建议遵循几个原则：

* 不使用 `latest`
* 标签中体现 Python 版本
* 镜像名明确表达“基础镜像”

## 七、推送基础镜像到远程仓库

确保已经完成登录：

```
docker login
```

然后执行推送：

```
docker push zhangdapeng520/fastapi:python3.11.9
docker push zhangdapeng520/fastapi:python3.11.9-slim
```

第一次推送速度较慢属于正常情况。

## 八、从使用者视角验证基础镜像

基础镜像推送完成后，一定要从“使用者”的角度验证。

```
docker pull zhangdapeng520/fastapi:python3.11.9
```

只要能够成功拉取，就说明这个基础镜像已经具备共享价值。

## 九、综合案例：基于通用基础镜像构建一个 FastAPI 项目

下面通过一个完整案例，验证这个基础镜像在真实项目中的使用方式。

### 项目目录结构

```
fastapi_project_demo/
├── Dockerfile
├── main.py
```

### 示例 FastAPI 应用代码

`main.py` 内容如下：

```
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "fastapi_base_image_running"}
```

### 项目 Dockerfile 示例

```
# 基于通用 FastAPI 基础镜像
FROM zhangdapeng520/fastapi:python3.11.9

WORKDIR /app

# 复制项目源码
COPY main.py .

# 暴露应用端口
EXPOSE 8000

# 定义应用启动命令
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

这个 Dockerfile 非常清晰：

* Python 与 FastAPI 环境由基础镜像提供
* 项目只关心自己的依赖与启动方式
* Dockerfile 可维护性显著提升

### 构建镜像并测试

构建镜像：

```
docker build -t fastapi_demo .
```

启动容器：

```
docker run -d --name fastapi_demo -p 8000:8000 fastapi_demo
```

浏览器访问：http://localhost:8000/

### 清理测试数据

```
docker rm -f fastapi_demo
docker rmi fastapi_demo
```

## 总结

通用 FastAPI 基础镜像不是“优化技巧”，而是**工程规模化之后的必然选择**。

一个合格的基础镜像，必须做到：

* Python 版本明确、固定
* FastAPI 运行环境统一
* 国内环境稳定
* 不包含任何业务代码

当你的团队开始：

* 先维护基础镜像
* 再在其之上构建项目镜像

说明你们已经从“每个 FastAPI 项目各自折腾 Docker”， 升级到了**可复用、可演进的后端容器化工程体系**。