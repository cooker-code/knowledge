---
title: 用于实时仪表盘的 10 个 FastAPI 流式 API 实践
author: AI大模型观察站
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzMjkwMjk3Mw==&mid=2247487314&idx=1&sn=5bcd5ecbe0b52c0f7667a9bdb9fd23c7&chksm=c31d2471e7660d45627d64f2b196354d15cf9f34bf27b6f260556e755a8b0ffc39b3f8dbd81b&mpshare=1&scene=24&srcid=0114qlyblr2vNPWmieERBm6t&sharer_shareinfo=048b2d4e9232b8039ccb0913c10ba90e&sharer_shareinfo_first=048b2d4e9232b8039ccb0913c10ba90e#rd
---

十个可直接复制粘贴的模式，用 FastAPI 向浏览器推送数据——顺滑、安全、低延迟。

*用 FastAPI 构建实时看板。十种流式模式——SSE、WebSocket、NDJSON、chunked responses、backpressure、fan-out、caching 和 security——配套可运行代码。*

---

看板不是被“一次刷新”杀死的，而是死于无数个刷新按钮。实时不是“更快的轮询（poll）”，而是关于**push**（推送）。服务器端用 FastAPI，对端是现代浏览器，你可以流式传输数值而不是整页——无需重构技术栈。下面是我常用的十个实用模式，让图表现在就动起来，而不是等 30 秒。

---

## 1) Server-Sent Events (SSE) - 最简单的“单向推送”

当你的看板只需要从服务器→浏览器的更新时，SSE 是最轻量的选择。运行在 HTTP/1.1 之上，对代理友好，并且自带自动重连。

```
# server.py  
from fastapi import FastAPI  
from fastapi.responses import StreamingResponse  
import asyncio, json, time  
  
app = FastAPI()  
  
asyncdefsse_gen():  
    whileTrue:  
        payload = {"t": time.time(), "cpu": 0.37}  
        yieldf"data: {json.dumps(payload)}\n\n"  
        await asyncio.sleep(1)  
  
@app.get("/metrics")  
asyncdefmetrics():  
    return StreamingResponse(sse_gen(), media_type="text/event-stream")
```

```
// client.js  
const ev = new EventSource("/metrics");  
ev.onmessage = e => {  
  const { t, cpu } = JSON.parse(e.data);  
  updateChart(t, cpu);  
};
```

使用场景：你想以极低成本向图表和计数器做推送。

---

## 2) WebSockets - 双向控制通道

对于筛选、实时搜索或协作式看板，你会希望浏览器能说话（回传）。

```
# server.py  
from fastapi import FastAPI, WebSocket  
app = FastAPI()  
  
@app.websocket("/ws")  
async def ws(ws: WebSocket):  
    await ws.accept()  
    await ws.send_json({"hello": "client"})  
    while True:  
        msg = await ws.receive_json()  
        # e.g., { "cmd": "subscribe", "ticker": "AAPL" }  
        await ws.send_json({"ok": True, "echo": msg})
```

提示：用 WebSocket 传控制消息；把重数据通过 SSE 或 NDJSON（见下）来流式传输，以分离关注点。

---

## 3) NDJSON over chunked HTTP - “平民级 Streaming API”

无需花哨协议；只是在一个无界响应里用换行分隔的 JSON。

```
# server.py  
from fastapi.responses import StreamingResponse  
import asyncio, json  
  
asyncdefndjson():  
    for i inrange(5_000):  
        yield json.dumps({"i": i}) + "\n"  
        await asyncio.sleep(0.01)  
  
@app.get("/stream")  
asyncdefstream():  
    return StreamingResponse(ndjson(), media_type="application/x-ndjson")
```

```
// client (modern Fetch streams)  
const res = awaitfetch("/stream");  
const reader = res.body.getReader();  
const dec = newTextDecoder();  
  
let buf = "";  
while (true) {  
const { value, done } = await reader.read();  
if (done) break;  
  buf += dec.decode(value, { stream: true });  
let idx;  
while ((idx = buf.indexOf("\n")) >= 0) {  
    const line = buf.slice(0, idx); buf = buf.slice(idx + 1);  
    handle(JSON.parse(line));  
  }  
}
```

很适合：表格数据流、日志尾部（log tail）、向图表回填数据（backfill）。

---

## 4) BackgroundTasks + Stream - 不要阻塞事件循环

快速推送；慢慢计算。在保持流响应的同时把重活卸载出去。

```
from fastapi import BackgroundTasks  
  
asyncdefcompute_and_queue(q):  
    # put rows into an asyncio.Queue without blocking client  
    for row inawait slow_db_scan():  
        await q.put(row)  
    await q.put(None)  # poison pill  
  
@app.get("/orders")  
asyncdeforders(bg: BackgroundTasks):  
    q: asyncio.Queue = asyncio.Queue(maxsize=1000)  
  
    asyncdefgen():  
        whileTrue:  
            item = await q.get()  
            if item isNone: break  
            yield json.dumps(item) + "\n"  
    bg.add_task(compute_and_queue, q)  
    return StreamingResponse(gen(), media_type="application/x-ndjson")
```

收益：通过队列大小实现稳定延迟与 backpressure。

---

## 5) Redis Pub/Sub fan-out - 一处生产，多处看板

让 worker 向某个通道发布；每个已连接客户端都能收到同一份数据包，无需重复工作。

```
import aioredis, asyncio, json  
from fastapi import FastAPI  
from fastapi.responses import StreamingResponse  
  
app = FastAPI()  
  
@app.get("/prices")  
asyncdefprices():  
    asyncdefgen():  
        r = aioredis.from_url("redis://localhost")  
        pub = r.pubsub()  
        await pub.subscribe("ticks")  
        asyncfor msg in pub.listen():  
            if msg["type"] == "message":  
                yieldf"data: {msg['data'].decode()}\n\n"  
    return StreamingResponse(gen(), media_type="text/event-stream")
```

模式：生产者只写一次；由 Redis fan-out 到 N 个客户端。

---

## 6) Postgres LISTEN/NOTIFY - 数据库原生事件

非常适合绑定到关系事件（新订单、作业状态）的看板。

```
# server.py  
import asyncpg, asyncio, json  
from fastapi.responses import StreamingResponse  
  
asyncdefpg_events():  
    conn = await asyncpg.connect(dsn="postgres://user:pass@localhost/db")  
    await conn.add_listener("order_updates", lambda *a: None)  
    try:  
        whileTrue:  
            msg = await conn.connection.notifies.get()  # async queue  
            yieldf"data: {json.dumps({'payload': msg.payload})}\n\n"  
    finally:  
        await conn.close()  
  
@app.get("/orders/sse")  
asyncdeforders_sse():  
    return StreamingResponse(pg_events(), media_type="text/event-stream")
```

SQL 侧：

```
NOTIFY order_updates, json_build_object('id', NEW.id, 'status', NEW.status)::text;
```

好处：无需轮询表；数据库直接把事件推出来。

---

## 7) Backpressure & rate limiting - 让图表更顺滑

你的浏览器画不动每秒 5 万个点。要么在服务器端做节流（throttle），要么合并事件。

```
# throttle generator  
async def rate_limited(gen, max_hz=20):  
    interval = 1.0 / max_hz  
    last = 0  
    async for item in gen:  
        now = asyncio.get_event_loop().time()  
        if now - last < interval:  
            continue  
        last = now  
        yield item
```

可与客户端小型 ring buffer 结合，只绘制最近的 N 个点。Smooth > raw（顺滑胜过原始）。

---

## 8) Heartbeats, retries, and idle timeouts

流式传输会断。要让问题显性化，并自动恢复。

```
# server heartbeat  
async def sse_gen():  
    try:  
        while True:  
            yield "event: ping\ndata: {}\n\n"  
            await asyncio.sleep(10)  
    finally:  
        # metrics, cleanup, etc.  
        pass
```

```
// client  
ev.addEventListener("ping", () => markHealthy());  
ev.onerror = () => markUnhealthy(); // EventSource auto-reconnects
```

注意：把代理超时设置得高于心跳间隔。

---

## 9) Security & multi-tenant streams

一次认证，按 topic 授权。短生命周期的 JWT + 服务器端校验。

```
from fastapi import Depends, HTTPException  
from fastapi.security import OAuth2PasswordBearer  
  
oauth2 = OAuth2PasswordBearer(tokenUrl="token")  
  
defuser_from_token(token=Depends(oauth2)):  
    # verify & decode; return user/tenancy  
    return {"sub": "u1", "org": "acme"}  
  
@app.get("/tenant/{topic}")  
asyncdeftenant_stream(topic: str, user=Depends(user_from_token)):  
    ifnot can_read(user, topic):  
        raise HTTPException(403, "forbidden")  
    return StreamingResponse(sse_gen_for(topic, user["org"]),  
                             media_type="text/event-stream")
```

实践：按 org/project 限定订阅范围，避免数据泄露。

---

## 10) Edge caching for initial state; stream the deltas

用缓存的 snapshot 即刻“注水”看板，然后通过 SSE/WebSocket 推送变更。

```
# initial snapshot route (cacheable)  
@app.get("/snapshot")  
async def snapshot():  
    data = await read_aggregate()  
    return JSONResponse(data, headers={"Cache-Control": "public, max-age=5"})
```

客户端流程：先请求 /snapshot → 立即渲染 → 打开 /metrics 的 SSE 获取实时更新。

结果：更好的 LCP，同时具备真正的实时行为。

---

## Micro-checklist（让你的流有“高级感”）

* 按需选择协议：SSE（单向）、WebSocket（双向）、NDJSON（批量流）。
* 保持数据包小且有类型：倾向固定的 JSON 结构；不要“大杂烩”的 “any” blob。
* 保护事件循环：把重活卸载到任务或 worker；使用 backpressure。
* 让失败显性化：心跳 + UI 健康指示。
* 度量指标：记录 TTFB（time-to-first-byte）、包速率、断连次数和 p95 延迟。

---

## 一个快速、真实的案例

某团队上线了一个交易看板，每 2 秒轮询一次，每次都重渲整张表。我们改为 Redis Pub/Sub → SSE，在客户端为图表加入 ring buffer，并在首次渲染时提供缓存的 snapshot。中位“首个数字出现”时间降至 300 ms 以下，中端笔记本的 CPU 占用下降 40%，带宽降低 60%——同时更新更快。用户不再问“现在是实时的吗？”

---

## 结语

实时不是一个特性，而是一种交付风格。用 FastAPI，你可以从很小开始——SSE 做推送、一个心跳和一个 snapshot——然后随着产品成长，逐步升级到 WebSocket、fan-out 和数据库原生事件。选择能带来顺滑运动和可预测成本的“最少原语（primitives）”。你的图表（以及你的运维团队）都会感谢你。

号召行动（CTA）：想要一个最小的起步模板，包含 SSE、WebSocket 控制通道、Redis fan-out，以及带 Fetch streams 的 React 客户端吗？评论 “stream-kit”，我会分享仓库链接。