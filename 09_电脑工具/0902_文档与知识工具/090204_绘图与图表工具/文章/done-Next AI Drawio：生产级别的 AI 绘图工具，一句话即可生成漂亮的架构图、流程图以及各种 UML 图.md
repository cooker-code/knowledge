---
title: Next AI Drawio：生产级别的 AI 绘图工具，一句话即可生成漂亮的架构图、流程图以及各种 UML 图
author: 计算机低手
date:
url: https://mp.weixin.qq.com/s?__biz=MzU1NzA1MTMwMg==&mid=2247485042&idx=1&sn=3a1d70bdf4946ddcbabbbc32eb4167b7&chksm=fdc0327a3b7f7d65640f2a76e926d264c7439ea8fa3885ccc7ca58dc2fe71b5f6e99b0d59ad2&mpshare=1&scene=24&srcid=1208gJNLjYsCzBJ6AeX3MGXb&sharer_shareinfo=810303151fa380353c0960dd9a4ebab7&sharer_shareinfo_first=810303151fa380353c0960dd9a4ebab7#rd
---
> 已吸收至：[[09_电脑工具/0902_文档与知识工具/090204_绘图与图表工具/090204_核心知识点/AI绘图与图表工具验收边界|AI 绘图与图表工具验收边界]]


## 一、简介

* • Next AI Drawio 是一个基于 Next.js 的 Web 应用程序，它将 AI 功能与 draw.io 图表集成在一起，
* • 该工具支持通过自然语言命令和 AI 辅助可视化地来创建、修改和增强图表
* • 支持市面上大多数模型供应商接入，可自定义使用各种模型
* • 支持使用 Docker 部署，简单上手，一键启动
* • 该工具的开源地址参考：https://github.com/DayuanJiang/next-ai-draw-io
* • 该工具的工作原理和架构图参考如下：

## 二、安装

* • 提前安装好Docker、docker-compose软件环境
* • 新建 docker-compose.yml 配置文件，配置内容如下：

```
version: '3.8'
services:
  next-ai-draw-io:
    image: ghcr.io/dayuanjiang/next-ai-draw-io:latest
    container_name: next-ai-draw-io
    ports:
      - "3000:3000"
    environment:
      - AI_PROVIDER=openai  #openai兼容的模型
      - AI_MODEL=claude-sonnet-4-5-20250929 #自定义模型
      - OPENAI_API_KEY=sk-xxx  #api key
      - OPENAI_BASE_URL=https://api.openai.com/v1  #自定义基础地址
      #- ACCESS_CODE_LIST=admin123 #页面需要授权访问时设置，多个用逗号隔开 
    restart: unless-stopped
```

* • 配置 docker-compose.yml 所在目录下，执行命令启动，如下

```
docker-compose up -d
```

## 三、使用

### 1. 安装完成后，访问：http://server\_ip:3000，进入绘图页面

### 2. AI 对话绘图示例

#### 2.1 示例一

* • 对话框输入：

```
描绘出tcp的三次握手和四次挥手
```

* • 绘制图表如下：

#### 2.2 示例二

* • 对话框输入：

```
生成一个包含**AWS 图标**的基于 PHP SWOOLE 的微服务架构图。在此图中，需要展示负载均衡、数据库主从、多实例Redis、Rabbit MQ消息队列
```

* • 绘制图表如下：

### 2.3 示例三

* • 对话框输入：

```
怎么利用第一性原理实现赚第一桶金
```

* • 绘制图表如下：

### 2.4 示例四

* • 对话框输入：

```
请为我画一只可爱的猫
```

* • 绘制图表如下：

## 四、总结

* • Drawio 是一个很强大的绘图软件，可以绘制各种类型的图，该工具实现了 AI 赋能，让 Drawio 绘图更加高效、智能、简单轻松
* • 可以使用 Docker 快速部署启动，配置简单，自定义模型，使用方便
* • 可以作为绘图生产力工具，在绘图前期可以给出大概的想法和需求，让AI大致帮你绘出雏形，后面再手动编辑优化，就能得到比较不错的出品

 

> 部署了一个临时试用地址，可以参考使用：https://cas.luler.top/?search=693294ff23f74

 

---

推荐一下个人网站（本文内容如有错漏只会在个人博客更新）：

* 博客：https://blog.luler.top/d/95
* 应用：https://cas.luler.top/
* 导航站：https://nav.luler.top/
* 开源推荐：https://gitshare.luler.top/
