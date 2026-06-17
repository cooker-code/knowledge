---
title: Hermes + Webhook：让工作流真正自动化
author: i龙虾
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI3MTk5OTc3Ng==&mid=2247484434&idx=1&sn=fefc5fd1ab6b30e708391a19c161b23c&chksm=eaf831ed140ccd30d854af6da69498a692a9caf123f3d5c068d740d6b8a857908e15bb999290&mpshare=1&scene=24&srcid=04238zg2MMuIwvq7EljRWk2R&sharer_shareinfo=6c2164de192f6df345165474f2a383b1&sharer_shareinfo_first=6c2164de192f6df345165474f2a383b1#rd
---

你有没有听过这种声音——"AI Agent 吹了这么久，到底能干啥？"

说实话，我自己以前也是这么想的。装了一堆工具，配了一堆技能，然后呢？打开聊天窗口，问一句，答一句，跟搜索引擎有啥区别？直到我开始玩 Webhook，才真正理解什么叫"Agent 在你睡觉的时候替你干活"。

今天我想聊的，就是把 Hermes Agent 和 Webhook 结合起来，怎么搭出一套真正自动化的流程。不是那种"我手动跑一下脚本"的伪自动化。外部事件一触发，Agent 自己就开工了——审查代码、推送通知、分析数据、回写结果。全程你不需要打开任何一个聊天窗口。

## 先搞清楚：Webhook 到底是什么

很多人觉得 Webhook 是个很技术的概念，其实特别简单。

打个比方。你家来了个快递，轮询（Polling）的方式是你每隔5分钟开门看一眼有没有人。Webhook 是装了个门铃——快递到了，一按，你立刻就知道了。

技术上说，就是一个外部服务在某件事发生时，往你指定的 URL 发一个 HTTP POST 请求，请求里带着事件详情的 JSON 数据。谁下了你的店铺订单、哪笔支付到账了、哪个 PR 被提交了——发生即通知，实时的。

GitHub、微信支付、支付宝、有赞……几乎所有主流平台都支持 Webhook。但问题是，大多数人拿到这个通知后，还得自己去处理。看一眼、手动归类、人工跟进。Webhook 变成了"高级通知"而已。

Agent 改变了这件事。

## Hermes Webhook 的核心：不只是接收通知

Hermes 的 Webhook 能力分两个方向：

**外部 -> Agent（入站）**：GitHub 提了个 PR、有赞来了新订单、微信支付收到一笔付款——事件进来，Agent 自动处理。这是最常用的模式。

**Agent -> 外部（出站）**：Agent 完成分析后，把结果 POST 到你的应用、飞书、企业微信，或者直接写回 GitHub 做评论。不是给你发消息让你自己去处理，而是 Agent 直接把活干了。

这两个方向组合起来，就不是一个"聊天机器人"了——它变成了一段后端服务。事件驱动、自动响应、跨平台协调。

## 配置实操：从零开始

好了，说干就干。以下所有命令和配置都来自 Hermes 官方文档，我尽量写得详细一些。

### 第一步：启用 Webhook 适配器

最简单的方式是用向导：

```
hermes gateway setup 
```

按提示启用 Webhook、设端口、设全局 HMAC 密钥就行。

如果你想手动配置，往 `~/.hermes/.env` 里加这几行：

WEBHOOK\_ENABLED=true WEBHOOK\_PORT=8644 # 默认端口 WEBHOOK\_SECRET=your-global-secret

`WEBHOOK_SECRET` 是用来验证请求是否真的来自你配置的那个服务的。后面会详细说安全性。

启动 Gateway 之后，验证一下服务是否跑起来了：

```
curl http://localhost:8644/health 
```

看到 `{"status": "ok", "platform": "webhook"}` 就说明没问题。

### 第二步：定义路由

路由是整个 Webhook 系统的核心。它决定了：什么事件进来、用什么提示词交给 Agent、Agent 处理完以后结果送去哪。

所有路由都写在 `~/.hermes/config.yaml` 里，放在 `platforms.webhook.extra.routes` 下面。看个完整的例子：

```
platforms:  webhook:    enabled: true    extra:      port: 8644      secret: "global-fallback-secret"      routes:        github-pr:          events: ["pull_request"]          secret: "github-webhook-secret"          prompt: |            Review this pull request:            Repository: {repository.full_name}            PR #{number}: {pull_request.title}            Author: {pull_request.user.login}            URL: {pull_request.html_url}            Diff URL: {pull_request.diff_url}            Action: {action}          skills: ["github-code-review"]          deliver: "github_comment"          deliver_extra:            repo: "{repository.full_name}"            pr_number: "{number}"
```

几个关键字段解释一下：

**events**：监听哪些事件类型。比如 GitHub 的 `pull_request`、`push`、`issues`。留空就接受所有事件。

**prompt**：这是给 Agent 的指令模板。注意 `{repository.full_name}` 这种花括号语法——它会自动从 Webhook 的 JSON payload 里取值，支持点号嵌套访问。还有一个特殊变量 `{__raw__}`，会把整个 payload 原样塞进去，适合处理结构不确定的通用 Webhook。

**skills**：给这次 Agent 运行注入哪些技能。比如上面的例子注入了 `github-code-review`，Agent 就会按照这个技能定义的流程去做代码审查。

**deliver**：Agent 处理完以后，结果送哪里去。支持的值包括 `github_comment`、`feishu`、`wecom`、`email`、`telegram`、`discord`、`slack` 等等。默认是 `log`，就是只打日志不推送。

**deliver\_extra**：投递的附加参数。比如往 GitHub 发评论需要指定仓库和 PR 编号，往飞书发消息需要指定 Webhook URL。这里的值同样支持花括号模板。

### 第三步：把外部服务的 Webhook 指向你的服务器

以 GitHub 为例：

1. 仓库 -> Settings -> Webhooks -> Add webhook

2. Payload URL 填：`http://your-server:8644/webhooks/github-pr`

3. Content type 选 `application/json`

4. Secret 填你路由配置里一样的值

5. 触发事件选 Pull requests

6. 保存

如果服务器在内网，需要做端口映射或者用 ngrok 暴露到公网。生产环境建议直接放在有公网 IP 的机器上，或者用反向代理（Nginx + SSL）。

### 第四步：确保投递目标已连接

如果你配了 `deliver: github_comment`，需要确保 Gateway 机器上装了 GitHub CLI 并已认证：

```
gh auth login 
```

如果配了 `deliver: feishu`，需要在飞书创建一个自定义机器人，拿到 Webhook URL，填到 `deliver_extra.webhook_url` 里。企业微信的群机器人同理，流程一样，拿到 key 填进去就行。

到这里，入站 Webhook 的完整流程就搭好了。PR 一提交 -> GitHub 发 POST -> Hermes 收到 -> Agent 审查代码 -> 自动在 PR 下发评论。全程不需要你碰一下。

## 四个真实场景

光讲配置有点枯燥，来看看实际能做什么。

### 场景一：新订单实时通知 + 每小时销售报告

假设你在运营一个有赞店铺，或者自己的电商后台接了微信支付。

**实时通知**：在路由里加一条 `new-order` 路由，deliver 到飞书机器人。每当有新订单，Agent 直接把买家信息、商品、金额推到你的飞书工作群。不需要开后台盯着。

**定时报告**：再配一个 cron job，每小时跑一次。用 Python 脚本调有赞 API 或者查自己的数据库，拉取最近一小时的订单数据，Agent 读取脚本输出后做摘要——新增多少笔订单、主要商品类目、有没有异常退款。

```
routes:  new-order:    events: ["trade.TradePaid"]    secret: "youzan-webhook-secret"    prompt: |      有赞新订单到账！      订单号：{order_no}      买家：{buyer_nick}      商品：{orders[0].title}      实付金额：{payment} 元      下单时间：{created}    deliver: "feishu"    deliver_extra:      webhook_url: "https://open.feishu.cn/open-apis/bot/v2/hook/your-token"
```

然后 cron job 这样配：

hermes cron create \

  --schedule "every 1h" \

  --prompt "根据脚本输出，总结最近一小时有赞店铺的销售情况。先说新增订单数，再说主要商品类目，最后标注有没有异常退款。" \

  --script youzan\_stats.py

更有意思的是，如果你的 Agent 同时接了微信支付的回调，你可以交叉分析——是不是发了某条推文后订单量涨了？是不是某个活动页上线后转化变好了？Agent 自动做这些对比，结果直接推到飞书。

### 场景二：GitHub PR 自动代码审查

这个可能是最实用的场景了。

流程很简单：有人提 PR -> GitHub 触发 Webhook -> Agent 拉取 diff、分析代码、评估风险 -> 自动在 PR 下面留一条审查评论。

配置就是上面那个完整的 `github-pr` 路由。Agent 不是简单地复制粘贴 PR 标题，而是真的会去看改动了什么，给出正确性判断、改进建议和风险等级。

从实际体验来看，一个中等复杂度的 PR，从 Webhook 触发到评论发布，大概一分钟左右。对于团队协作来说，这意味着即使是半夜两点提的 PR，也会立刻有一份初步审查意见等着你。不是要替代人类审查，而是先帮你看一遍，把明显的低级错误拦住。

几个踩坑点：

• GitHub CLI 必须在 Gateway 运行的环境里安装和认证，光 Windows 上装了没用，WSL 或 Linux 环境里也要装。

• Gateway 必须在运行状态，否则 Webhook 打过来是 502。

• ngrok 免费版的 URL 每次重启会变，记得更新 GitHub 里的配置。

• 第一次设置完以后，GitHub 会发一个 ping 请求来测试，确保你的服务能正确响应。

### 场景三：Agent 分析数据后回写应用

这个稍微高级一些，用到了出站方向。

假设你有一个新闻分析的 Web 应用。你配一个 cron job，让 Agent 定时跑一个 Python 脚本去拉 RSS 新闻源，然后 Agent 阅读这些新闻，分析出最可能成为热点的主题，最后把分析结果 POST 回你的应用 API。

```
platforms:  webhook:    enabled: true    extra:      routes:        news-analysis:          secret: "app-secret"          prompt: "Analysis complete. See agent response."          deliver: "log"
```

cron job 配置：

hermes cron create \

  --schedule "0 9 \* \* \*" \

  --script poll\_news\_rss.py \

  --prompt "你在分析新闻热点趋势。文章内容在脚本输出里。提取出最可能成为热点的前10个关键词，返回符合 API schema 的合法 JSON。生成完 JSON 之后，用 terminal 工具把结果 POST 到 https://your-app.com/api/agent-updates，带上 secret header。" \

  --deliver feishu

注意这个场景里，Agent 不只是"通知"你那么简单。它是整个工作流的核心环节。脚本负责原始数据采集，Agent 负责理解和分析，最后 Agent 自己把结果写回应用。你唯一要做的事，就是第一次配置好，然后看看最终结果是不是靠谱。

而且出站模式不需要 ngrok 或任何公网暴露，因为是 Agent 主动发 HTTP 请求出去。

### 场景四：微信公众号新关注者自动运营

这个场景对做内容的人来说可能是最有感的。

微信公众号开启开发者模式后，用户关注、发消息、点击菜单等事件全部会以 POST 请求的形式推送到你配置的服务器 URL。官方文档里叫"消息与事件推送"，本质就是 Webhook。个人订阅号也能用，门槛是这几个场景里最低的。

**配置公众号服务器端**

先在公众号后台开启开发者模式：设置与开发 -> 基本配置 -> 服务器配置。需要填三个东西：

• **服务器地址（URL）**：填 `http://your-server:8644/webhooks/wechat-mp`

• **令牌（Token）**：随便设一个字符串，记住，后面路由里要一样

• **消息加解密方式**：先选明文模式，调通之后再考虑加密

有一个细节：微信在保存配置的时候会发一个 GET 请求到你的 URL 做验证（带 `signature`、`timestamp`、`nonce`、`echostr` 四个参数），必须原样返回 `echostr` 才算验证通过。Hermes 的 Webhook 适配器会自动处理这个握手，不用自己写。

**路由配置**

微信推过来的是 XML 格式，Hermes 会自动解析成 JSON 再塞进 prompt 模板。关注事件的 payload 字段是 `Event: subscribe`，普通文本消息是 `MsgType: text`。

```
routes:  wechat-mp-subscribe:    events: ["subscribe"]    secret: "your-mp-token"    prompt: |      微信公众号新增关注者！      用户 OpenID：{FromUserName}      关注时间：{CreateTime}      请根据这个信息，生成一段热情但不夸张的欢迎语，      提醒用户可以发送关键词获取往期精华内容，      最后附上最近发布的三篇文章标题。    skills: ["mp-welcome"]    deliver: "wechat_mp_reply"    deliver_extra:      to_user: "{FromUserName}"      from_user: "{ToUserName}"
```

`deliver: "wechat_mp_reply"` 会让 Agent 把回复内容直接通过公众号客服消息接口发给用户。需要公众号有客服消息权限——订阅号默认有，服务号认证后也有。

**不只是发欢迎语**

更有意思的用法是接用户的消息。配一条 `text-message` 路由监听 `MsgType: text` 事件，用户发什么，Agent 读到之后直接回复。不是固定关键词匹配，而是真正理解意图。

```
  wechat-mp-message:    events: ["text"]    secret: "your-mp-token"    prompt: |      公众号用户发来消息：      内容：{Content}      用户 OpenID：{FromUserName}      请用简短友好的语气回复，如果是问技术问题就认真解答，      如果是闲聊就简单回应，不要超过200字。    deliver: "wechat_mp_reply"    deliver_extra:      to_user: "{FromUserName}"      from_user: "{ToUserName}"
```

配合 `skills` 注入一个了解你公众号内容定位的技能文件，Agent 的回复就会有明确的"人设"，不会答得驴唇不对马嘴。

踩坑点：

• 微信要求服务器在 5 秒内响应，超时会重试三次。Agent 推理时间如果超了，用 `deliver_only` 先回一句"正在处理"，再异步推送结果。

• 公众号 Token 就是验证签名用的字符串，跟 HMAC secret 的作用一样，路由配置里填到 `secret` 字段就行。

• 未认证订阅号每天只能发 1 条模板消息，但客服消息（用户主动发消息后 48 小时内）没有这个限制，大部分场景够用。

• 本地调试同样需要 ngrok 把端口暴露出去，微信不接受内网地址。

我的VPS到期了，目前没有启用。

## 直投模式：不需要 Agent 的时候

有些场景根本不需要 Agent 去"思考"。观测云告警了、Supabase 数据库有变动、后台任务跑完了——你只是想把一条消息推到飞书或企业微信，不需要花 LLM 的 token。

Hermes 支持 `deliver_only: true` 模式：

```
routes:  deploy-notify:    events: ["push"]    secret: "deploy-secret"    prompt: "New push to {repository.full_name} branch {ref}: {head_commit.message}"    deliver: "wecom"    deliver_only: true
```

这个模式下，prompt 模板渲染完就直接投递，Agent 根本不被调用。零 token 消耗，亚秒级延迟。对于高频事件特别有用——你不会希望每次数据库写入都跑一次 Agent 推理。

也可以用 CLI 动态创建直投订阅：

hermes webhook subscribe order-matches 

  --deliver wecom \

  --deliver-webhook-key "your-wecom-bot-key" \

  --deliver-only \

  --prompt "新订单：{order.buyer\_nick} 购买了 {order.title}！" \

  --description "有赞订单企业微信通知"

不需要重启 Gateway，创建即生效。

## 安全：不能忽略的事

Webhook 本质上是让互联网上的服务往你的机器发请求。安全措施不到位，就等于把门敞开。

**HMAC 签名验证**：每个请求进来，Hermes 会验证签名。

GitHub 用 `X-Hub-Signature-256`（HMAC-SHA256）

有赞用 `Authorization`（HMAC-SHA256，base64 编码）

通用格式用 `X-Webhook-Signature`

签名不对的请求直接 401 拒掉。

**每个路由必须有 secret**：要么在路由级别配，要么用全局 secret 兜底。没配 secret 的路由在启动时会直接报错。

**限流**：默认每个路由 30 请求/分钟。超了返回 429。可以通过配置调整：

```
platforms:  webhook:    extra:      rate_limit: 60  # 每分钟 60 次
```

**幂等性**：重复的投递（比如 GitHub 重试）会被自动去重，不会重复触发 Agent。投递 ID 缓存 1 小时。

**请求体大小限制**：默认 1MB，超过直接拒掉。可以调整：

```
platforms:  webhook:    extra:      max_body_bytes: 2097152  # 2MB
```

**特别提醒**：Webhook payload 里包含攻击者可控的数据——PR 标题、commit message、issue 描述，这些都可能被注入恶意指令。如果你的 Hermes 实例暴露在公网上，强烈建议在 Docker 或虚拟机里跑网关（Gateway），用沙箱环境隔离风险。

## 回到那个老问题

"AI Agent 到底能干啥？"

我现在每天收到的信息流——GitHub 的 PR 更新、有赞店铺订单数据、定时新闻分析结果——都是 Agent 在处理。盯着屏幕等通知？不存在的。手动跑脚本？不用了。自己去分析一堆原始数据然后写摘要？Agent 搞定了。

Webhook 把 Agent 从"你问它答"的被动模式，变成了"事件驱动、主动响应"的服务节点。说白了就是把已经存在的东西连接起来了——事件触发、Agent 处理、结果投递。

自动化的门槛从来不是"有没有工具"，而是"你愿不愿意花一个小时把管道接好"。接好之后，每天省下来的可能是一个小时，也可能更多。

Hermes 的 Webhook 文档写得很清楚，配置也不复杂。如果你已经在用 Agent，没理由不试试。如果你还没开始用，这可能是最能体现 Agent 价值的切入点——不是让它跟你聊天，而是让它替你跑腿。

Hermes Webhooks 官方文档：

https://hermes-agent.nousresearch.com/docs/user-guide/messaging/webhooks