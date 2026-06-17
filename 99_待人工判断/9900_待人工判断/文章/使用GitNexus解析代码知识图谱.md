---
title: 使用GitNexus解析代码知识图谱
author: 智慧胶囊新时代
date: 
url: https://mp.weixin.qq.com/s?__biz=MzcwNjEyNDI5NA==&mid=2247483854&idx=1&sn=14d0350b6385847af4754ce73ddd140c&chksm=f5fa7628637118699e1fee251e30ef59ab96c4992576992cc72f8ba818793d80fb006ae052cc&mpshare=1&scene=24&srcid=0419k4PsGdnADcknumtstggj&sharer_shareinfo=0a18b0e47bf5ab3b8ba45cf856cc7076&sharer_shareinfo_first=0a18b0e47bf5ab3b8ba45cf856cc7076#rd
---

鉴于近期 LLM Wiki 的爆火，接触到 GitNexus 这个项目，挺不错的。

能解析代码仓库、并结合大语言模型生成项目说明。实操走起：

```
cd ~/Applications npm install gitnexus~/Applications/node_modules/.bin/gitnexus  ~/Documents/gitnexus
```

生成维基：

```
~/Applications/node_modules/.bin/gitnexus wiki ~/Documents/GitNexus --provider openai --model Qwen3-Coder:30B --concurrency 4 --base-url http://llm.service.api/v1 --api-key 'GitNexus' --verbose
```

生成的偏技术实现的项目文档，预览如下：

---

把陈年项目也搬出来试试手：

* github.com/cheetahmobile/tsunami-udp

突然就有活力了，按需修改提示词，比如换用中文编写或换风格：

$(npm root -g)/gitnexus/dist/core/wiki/prompts.js

---

GitNexus自带Web页面，更方查看和查询关联关系。（目前需要下载项目源码编译使用）

```
git clone https://github.com/abhigyanpatwari/GitNexus.git ~/Documentscd ~/Documents/GitNexus/gitnexus-sharednpm run buildcd ~/Documents/GitNexus/gitnexus-webnpm installnpm run dev# 或 编译使用npm run buildnpx serve -l 5173 ~/Documents/GitNexus/gitnexus-web/dist# 要输出中文需要手动修改安装包里的提示词： node_modules/gitnexus/dist/core/wiki/prompts.jsgitnexus wiki ~/Documents/GitNexus --provider openai --model Qwen3-Coder:30B --concurrency 4 --base-url http://llm.service.api/v1 --api-key 'GitNexus' --verbose
```

命令行启动 Web 服务：

GitNexus-Web 页面展示如下，需要再启动本地的 GitNexus-Serve 服务

本地 GitNexus-Serve 启动后能自动连接并加载最近分析过的项目：

选择 tsunami-udp 之后，可以看到详细的代码图谱：

还内置了了AI功能，可连本地推理服务：