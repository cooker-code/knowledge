---
title: IT-Tools：高效程序员的开源工具合辑
author: 爱踢人生sre
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI0MDg2NTc2Ng==&mid=2247499936&idx=1&sn=47962af137e20f8ecf75d4ba976835f5&chksm=e83d73365e72b4a543b3220568e95a06ac2b248f88877876a9389ecfbcab0fdacb18e8abe871&mpshare=1&scene=24&srcid=1201QjYOhLJxG2qSMfjvi1Pc&sharer_shareinfo=852f75adf1e6d3a601a7363ac578a2e5&sharer_shareinfo_first=852f75adf1e6d3a601a7363ac578a2e5#rd
---

1、IT-Tools简介

IT-Tools 是一款为开发人员打造的便捷在线工具。它功能全面，可以让开发者用更高效方式完成任务。本文会详细介绍如何用 Docker 的方式在本地部署 it-tools 到个人服务器，让用户享受到快捷的访问与使用体验。

2、核心功能

加密工具：Token生成、Hash文本、UUID生成、RSA密钥对等。

转换工具：Base64编码、YAML转XML、JSON美化、时间戳转换等。

开发辅助：Docker CLI转Compose、定时任务生成、Emoji编码生成等。

其他工具：HTTP状态码查询、二维码生成、IP地址解析等。 ‌

# 3、安装docker

1、安装依赖包

yum install -y yum-utils device-mapper-persistent-data lvm2

2、配置docker yum源

yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo 

3、安装docker

yum install -y docker-ce

4、修改docker配置文件

mkdir /etc/docker -p

sudo tee /etc/docker/daemon.json <<-'EOF'

{

    "registry-mirrors": [

      "https://docker.credclouds.com",

      "https://k8s.credclouds.com",

      "https://quay.credclouds.com",

      "https://gcr.credclouds.com",

      "https://k8s-gcr.credclouds.com",

      "https://ghcr.credclouds.com",

      "https://do.nark.eu.org",

      "https://docker.m.daocloud.io",

      "https://docker.nju.edu.cn",

      "https://docker.mirrors.sjtug.sjtu.edu.cn",

      "https://docker.1panel.live",

      "https://docker.rainbond.cc"

    ],

   "data-root": "/etc/docker"

}

EOF

5、启动docker

systemctl daemon-reload

systemctl enable docker --now

systemctl restart docker

6、安装docker-compose

#下载docker-compose文件

curl -L "https://github.com/docker/compose/releases/download/v2.29.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# 给他一个执行权限

chmod +x /usr/local/bin/docker-compose

ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose 

# 查看是否安装成功

docker-compose --version

# 4、拉取it-tools镜像

docker pull corentinth/it-tools:latest

# 5、创建数据目录

mkdir -p /data/it-tools

chmod 777 /data/it-tools

# 6、docker安装it-tools

docker run -d \

  --name it-tools \

  --restart unless-stopped \

  -p 8000:80 \

  corentinth/it-tools:latest

# 7、编辑docker-compose.yaml文件

vi /data/it-tools/docker-compose.yaml

services:

  it-tools:

    image: corentinth/it-tools:latest

    container\_name: it-tools

    restart: unless-stopped

    ports:

      - "8000:80"

# 8、启动it-tools容器

cd /data/it-tools/

docker-compose up -d

docker-compose ps

# 9、查看容器日志

docker logs -f it-tools

# 10、访问it-tools服务

浏览器访问: http://192.168.52.15:8000

# 10、it-tools介绍

设置中文界面:

Token生成器:

Hash文本:

 

加密:

UUIDs生成器:

ULID生成器:

加密/解密文本:

 

BIP39密码生成器:

Hmac生成器:

RSA密钥对生成器:

密码强度分析仪:

PDF签名检查器:

 11、总结

IT-Tools 是一个开源、可私有化部署的在线开发者工具集。它通过集成上百种涵盖编码、加密、格式转换、网络分析等场景的高频工具，旨在极速解决开发与运维工作中的日常问题。其核心价值在于**所有工具均在本地浏览器中运行**，彻底杜绝了敏感数据泄露的风险，成为一款高效、安全、可随时取用的“技术瑞士军刀”。