---
title: AI工作流5：通过docker本地部署fireCrawl
author: 飞翔逆向
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg4NTYwODY5NQ==&mid=2247484606&idx=1&sn=65de65269f9c111d3467252ca7b8e60e&chksm=ce4fc4b7ba1f65b813eba9a6eaed207ff1fe19d461b8b099178cb319844f74a7b26d67ddbb8d&mpshare=1&scene=24&srcid=0919YDBrfPCXpkQUxhB2pTps&sharer_shareinfo=34810df5b60593e302a8d0d10524ea90&sharer_shareinfo_first=34810df5b60593e302a8d0d10524ea90#rd
---

1、什么是FireCrawl?

FireCrawl 是一个开源的网络爬虫框架，旨在帮助开发者高效、灵活地抓取和解析网页数据。我们可以在Dify和N8N的工作流里面添加FireCrawl 完成简单的爬虫任务。

Dify和N8N都是功能强大的开源工作流项目，FireCrawl 的官方api有额度限制，所以考虑是否可以本地部署FireCrawl服务，然后通过dify或者N8N调用本地的FireCrawl服务，这样子就可以无限制进行网页爬取。

2、什么是N8N？

简单说就是一个开源的工作流自动化工具。下面教大家如何在windows安装FireCrawl和如何在N8N使用fireCrawl。

3、使用docker部署FireCrawl

因为我是windows环境，所以直接安装Docker Desktop，再通过Docker Desktop安装FireCrawl

3.1 下载FireCrawl源码

解压源码包，在fireCrawl目录下面新增.env配置文件

3.2 copy.env

使用下面配置

修改下面2个配置即可

`## To turn on DB authentication, you need to set up supabase.``USE_DB_AUTHENTICATION=false``# 这里是设置 api key 的，但是内网使用的话可以随意设置的``TEST_API_KEY=fs-test`

```
# .env  
# ===== Required ENVS ======NUM_WORKERS_PER_QUEUE=8 PORT=3002HOST=0.0.0.0  
#for self-hosting using docker, use redis://redis:6379. For running locally, use redis://localhost:6379REDIS_URL=redis://redis:6379  
#for self-hosting using docker, use redis://redis:6379. For running locally, use redis://localhost:6379REDIS_RATE_LIMIT_URL=redis://redis:6379 PLAYWRIGHT_MICROSERVICE_URL=http://localhost:3000/scrape  
## To turn on DB authentication, you need to set up supabase.USE_DB_AUTHENTICATION=false  
# ===== Optional ENVS ======  
# Supabase Setup (used to support DB authentication, advanced logging, etc.)SUPABASE_ANON_TOKEN= SUPABASE_URL= SUPABASE_SERVICE_TOKEN=  
# Other Optionals# use if you've set up authentication and want to test with a real API key# 这里是设置 api key 的，但是内网使用的话可以随意设置的TEST_API_KEY=fs-test# set if you'd like to test the scraping rate limitRATE_LIMIT_TEST_API_KEY_SCRAPE=# set if you'd like to test the crawling rate limitRATE_LIMIT_TEST_API_KEY_CRAWL=# set if you'd like to use scraping Be to handle JS blockingSCRAPING_BEE_API_KEY=# add for LLM dependednt features (image alt generation, etc.)OPENAI_API_KEY=BULL_AUTH_KEY=@# use if you're configuring basic logging with logtailLOGTAIL_KEY=# set if you have a llamaparse key you'd like to use to parse pdfsLLAMAPARSE_API_KEY=# set if you'd like to send slack server health status messagesSLACK_WEBHOOK_URL=# set if you'd like to send posthog events like job logsPOSTHOG_API_KEY=# set if you'd like to send posthog events like job logsPOSTHOG_HOST=  
# set if you'd like to use the fire engine closed betaFIRE_ENGINE_BETA_URL=  
# Proxy Settings for Playwright (Alternative you can can use a proxy service like oxylabs, which rotates IPs for you on every request)PROXY_SERVER=PROXY_USERNAME=PROXY_PASSWORD=# set if you'd like to block media requests to save proxy bandwidthBLOCK_MEDIA=  
# Set this to the URL of your webhook when using the self-hosted version of FireCrawlSELF_HOSTED_WEBHOOK_URL=  
# Resend API Key for transactional emailsRESEND_API_KEY=  
# LOGGING_LEVEL determines the verbosity of logs that the system will output.# Available levels are:# NONE - No logs will be output.# ERROR - For logging error messages that indicate a failure in a specific operation.# WARN - For logging potentially harmful situations that are not necessarily errors.# INFO - For logging informational messages that highlight the progress of the application.# DEBUG - For logging detailed information on the flow through the system, primarily used for debugging.# TRACE - For logging more detailed information than the DEBUG level.# Set LOGGING_LEVEL to one of the above options to control logging output.LOGGING_LEVEL=INFO
```

3.3、使用docker-compose拉取和运行FireCrawl

在.env同目录下执行命令

```
docker-compose up -d
```

3.4、测试FireCrawl

通过apiFox或者postman测试FireCrawl服务是否正常：

如果出现下载go依赖无法访问外网的情况，需要修改/apps/api/目录，修改dockerFile文件，添加国内代理配置：

```
FROM golang:1.24 AS go-baseENV GOPROXY=https://goproxy.cn,direct   *******#加上这么一行设置国内代理  *******COPY sharedLibs/go-html-to-md /app/sharedLibs/go-html-to-md  
RUN cd /app/sharedLibs/go-html-to-md && \    go mod tidy && \    go build -o html-to-markdown.so -buildmode=c-shared html-to-markdown.go && \    chmod +x html-to-markdown.so
```

重新执行：docker-componse up -d

4.在N8N中使用FireCrawl获取网页数据

在N8N使用fireCrwal目前有2个办法，方法1如下图直接引入对应的fireCrawl节点，选择对应的acition即可。需要配置官方的appkey，有使用额度限制。

方法2：新增一个http节点，通过http节点调用本地部署的fireCrawl即可。

到此白嫖fireCrawl就完成了。

**如果你喜欢小蜜蜂的分析，请不吝给本文一个转发或者再看。谢谢。**

**下期预告：分享一个微信公众号自动发布的工作流。**