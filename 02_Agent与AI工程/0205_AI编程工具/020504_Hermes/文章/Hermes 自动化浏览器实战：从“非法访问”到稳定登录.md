---
title: Hermes 自动化浏览器实战：从“非法访问”到稳定登录
author: 冥破
date: 冥破冥破
url: https://mp.weixin.qq.com/s?__biz=MzIxOTgwNDA1OQ==&mid=2247484446&idx=1&sn=9270020ade87a35247670f3fb627b181&chksm=961c15ae104d9f4291950fd9f1133c4ebef4e2c004bebb3924c68f70059eb7c15c5c69103ead&mpshare=1&scene=24&srcid=0530tsgVfXaR9Pmfmqs2KBQW&sharer_shareinfo=6836065261c7d1321a825a469481c51f&sharer_shareinfo_first=6836065261c7d1321a825a469481c51f#rd
---

# Hermes 自动化浏览器实战：从“非法访问”到稳定登录

你有没有遇到过这种情况：

* 想让 Agent 帮你查网页，结果刚打开就提示“非法访问”。
* 扫码登录成功了，下次再用，登录态又没了。
* 自己浏览器能正常访问，换成自动化浏览器就被识别、拦截、空白页。

我最近折腾 Hermes 自动化浏览器，踩的就是这些坑。

最后我这边跑通了两条路线：

* 用 agent-browser 做本地 Chromium 自动化，加一点基础伪装。
* 用 camofox-browser 接管 Hermes 浏览器后端，走 Camoufox 这套反检测浏览器。

不是万能方案，但对“网页访问 + 登录保持 + 信息查询”这类任务，已经实用很多。

---

## 一、问题现场：默认浏览器为什么容易被拦

比如用 Hermes 默认浏览器访问闲鱼时，就可能遇到这种情况：

原因其实不复杂：

* navigator.webdriver = true，网站能看出浏览器被自动化工具控制。
* User-Agent 里可能带 "HeadlessChrome"，这是很明显的自动化特征。
* 默认浏览器会话不稳定，Cookie 和登录态不容易保留下来。

**真正麻烦的不是“打开一次网页”，而是后面还能不能像正常浏览器一样继续访问、登录和查询。**

---

## 二、第一条路线：先让 agent-browser 像个正常浏览器

Hermes 没有配置云端浏览器服务时，默认会走本地 agent-browser。

最直接的办法，是给它加一点基础伪装。

在 ~/.hermes/.env 里配置：

● ● ●bash

`AGENT_BROWSER_ARGS=--disable-blink-features=AutomationControlled  
AGENT_BROWSER_USER_AGENT="Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36"`

这两行主要做两件事：

* 降低 WebDriver 自动化特征。
* 避免 User-Agent 暴露 "HeadlessChrome"。

改完 .env 后，重启 Hermes Agent 才会生效。

我也把这套能力封装成了 browser-stealth Skill，可以直接拿走：

https://gitee.com/willis-song/willis-hermes-skills.git

把仓库里的 browser/browser-stealth 复制到：

● ● ●text

`~/.hermes/skills/browser`

使用时可以直接跟 Hermes 说：

> 请参考全局 skill browser-stealth，访问 xx 官网。

配置好之后，再访问闲鱼，效果会好很多：

需要登录时，让 Hermes 发截图过来，然后用手机 App 扫码：

---

## 三、第二条路线：用 Camofox 接管浏览器后端

agent-browser 加基础伪装后，已经能解决一部分问题。

但我实际用下来，遇到过一些不稳定情况。比如闲鱼页面能打开，但搜索框识别不稳，后续操作容易卡住。

这时候可以考虑 Camofox。

简单说：

* Camoufox 是基于 Firefox 的反检测浏览器。
* camofox-browser 把 Camoufox 包装成本地服务，默认端口是 9377。
* Hermes 配置 CAMOFOX\_URL 后，就可以把浏览器工具请求转到 Camofox。

怎么选？

| 场景 | 建议 |
| --- | --- |
| 普通网页访问 | 先用 agent-browser |
| 轻度拦截 | agent-browser + stealth 配置 |
| 反检测强、交互不稳 | 上 Camofox |
| 想长期保持登录态 | Camofox + managed\_persistence |

我自己的感受是：能用 agent-browser 跑通就先用它，成本最低；遇到反检测更强的网站，再上 Camofox。

---

## 四、实战配置：部署 Camofox 并接入 Hermes

Camofox 这条路线多一步部署成本，但好处是链路更清楚：本地先跑起 camofox-browser 服务，再让 Hermes 通过 CAMOFOX\_URL 去连接它。

### 1. Docker Compose 部署

我更推荐 Docker Compose 部署，目录结构清楚，后续也方便持久化浏览器数据。

目录可以自己定。我使用的是：

● ● ●text

`~/.hermes/options/camofox`

我的 compose 文件和 camofox 源码都放在这个目录下。

● ● ●bash

`mkdir -p ~/.hermes/options/camofox  
cd ~/.hermes/options/camofox`

先下载源码：

● ● ●bash

`git clone https://github.com/jo-inc/camofox-browser`

再编译：

● ● ●bash

`cd camofox-browser  
make build`

这个过程会有一点久，我这边大概半小时。

你也可以直接通过 make up 启动，Makefile 里有写。我没有试过，理论上是可行的。官方 README 也提示，Docker 方式建议用 make up 或 make fetch 后再 make build，不要直接裸跑 docker build。

build 之后，本地会构建好 Camofox 镜像。可以用下面命令查看：

● ● ●bash

`docker image ls`

如果能看到类似下面的镜像，就说明编译成功了：

● ● ●text

`IMAGE                            ID             DISK USAGE   CONTENT SIZE   EXTRA  
camofox-browser:135.0.1-x86_64   8ccffcf5c910       3.67GB         1.14GB  
...`

创建 docker-compose.yml：

● ● ●yaml

`version: "3.8"  
  
services:  
camofox:  
    image: camofox-browser:135.0.1-x86_64  
    container_name: camofox  
    restart: unless-stopped  
    ports:  
      - 9377:9377  
    environment:  
      - CAMOFOX_PORT=9377  
      - CAMOFOX_PROFILE_DIR=/home/node/.local/state/camofox/profiles  
      - CAMOFOX_COOKIES_DIR=/home/node/.local/state/camofox/cookies  
    volumes:  
      - ./camofox-data:/home/node/.local/state/camofox`

启动服务：

● ● ●bash

`docker compose -f docker-compose.yml up -d  
docker compose logs -f camofox`

第二条命令主要是看日志，确认服务是否正常启动。

验证服务：

● ● ●bash

`curl http://localhost:9377/health`

正常会返回类似：

● ● ●json

`{"status":"ok"}`

### 2. 手动部署

如果不想走 Docker Compose，也可以参考官方文档，用 npm install && npm start 手动启动 camofox-browser。

---

## 五、Hermes 里启用 Camofox

Camofox 服务启动后，还需要告诉 Hermes 去使用它。

在 ~/.hermes/.env 中添加：

● ● ●bash

`CAMOFOX_URL=http://localhost:9377`

这个环境变量优先级很高，一旦配置，就一定会走 Camofox。

查看当前用的是 Camofox 还是 agent-browser，可以使用 hermes tools 命令，进入 Browser Automation 看：

如果希望登录态、Cookie 能在多次任务之间保持，可以在 ~/.hermes/config.yaml 中添加：

● ● ●yaml

`browser:  
  camofox:  
    managed_persistence: true`

这个配置路径很重要。

managed\_persistence 必须写在 browser.camofox 下面。写在顶层会被忽略。

改完之后，完整重启 Hermes Agent，让配置生效。

启用 managed\_persistence: true 后，Hermes 会发送稳定的 userId 给 Camofox 服务端，让服务端复用同一个 Firefox Profile。

这带来几个效果：

* 登录态、Cookie 可以在多次任务间保持。
* userId 按 Hermes profile 隔离。
* 不同 profile 使用不同的浏览器数据，互不干扰。

状态数据大概在两个地方：

● ● ●text

`Hermes 侧：~/.hermes/browser_auth/camofox/  
Camofox 服务侧：容器内 /home/node/.local/state/camofox（如果有挂载）`

需要注意的是，Hermes 只是发送稳定的 userId，真正的 Profile 持久化还要看 Camofox 服务端是否按这个 userId 正确复用浏览器数据。

如果配置路径写错，或者服务端没有正确处理持久化，下一次任务还是可能掉登录态。

---

## 六、用闲鱼验证效果

下面是我拿闲鱼做的一次 demo。

先让 Hermes 访问闲鱼：

Camofox 正确“看到”了网页，并把截图发回来：

我让它帮我搜集二手沙发床相关商品，但这一步需要登录。页面弹出登录界面后，让我扫码：

扫码登录后，继续搜索，最终拿到了结果：

这只是一个 demo，我只是先抛了块砖。

对于想抓取网页信息的场景，可以通过多轮验证，慢慢告诉 Agent 要提取哪些字段、输出什么格式、怎么整理结果。等流程稳定后，就能形成一整套工作流。

一旦形成经验，Hermes 也会自己创建 Skill，后续再做类似网页信息抓取和整理时，它就可以直接参考已有 Skill。

淘宝我还没搞定。

这个站点的反检测和登录链路更复杂，我目前没有一个可以稳定复用的方案。你们如果有跑通的经验，不妨教教我。😄

---

## 参考资料

* Hermes Agent 浏览器文档：https://hermes-agent.nousresearch.com/docs/user-guide/features/browser
* Camoufox：https://camoufox.com/
* Camofox-browser：https://github.com/jo-inc/camofox-browser
* 我的 Hermes Skills 仓库：https://gitee.com/willis-song/willis-hermes-skills.git