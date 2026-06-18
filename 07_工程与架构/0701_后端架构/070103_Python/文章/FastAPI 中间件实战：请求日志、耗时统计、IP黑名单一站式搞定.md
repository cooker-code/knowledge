---
title: FastAPI 中间件实战：请求日志、耗时统计、IP黑名单一站式搞定
author: Ssoul肥鱼
date: 
url: https://mp.weixin.qq.com/s?chksm=cf7f6138f808e82ef1f5b7b3c4897fcbc923ebd6b558842ce105fedb0849e38a8ff61c9117c1&exptype=unsubscribed_card_recommend_article_u2i_mainprocess_coarse_sort_tlfeeds&ranksessionid=1776648604_2&req_id=1776648034534582&mid=2247484205&sn=1e90a2b3c35f762ce38262bbeed79053&idx=1&__biz=Mzg3OTg1MTM4Ng%3D%3D&scene=169&subscene=200&sessionid=1776648687&flutter_pos=20&clicktime=1776648791988&enterid=1776648791988&finder_biz_enter_id=5&key=daf9bdc5abc4e8d045a3ee3f5d0e502af84bafab7a461b6159b2b24dfb5fafbd660001cd4988c6fea1c21c1b3e5d7679b9f8e84124fb89512f268bf6c0b9dd3613050ded6711aab2a3380d854e9b3bb4f4300a4bda29859a7bed582c808ec1645e359e3edd080d6ca9a52ba989f87640eea6e757feb30cf914ae7e1419db5e5d&ascene=0&uin=MjAwNTI2NjEyMw%3D%3D&devicetype=OHOS-22&version=f3801037&abtest_cookie=AAACAA%3D%3D&lang=zh_CN&countrycode=CN&exportkey=n_ChQIAhIQ2JtxW831qpP8NXKpHXawNRLfAQIE97dBBAEAAAAAAMZVJjA5fosAAAAOpnltbLcz9gKNyK89dVj0LQkLJft1n%2F%2Fx7vpgolzD72IzzWO47c4ZTsV0LWLkHFotFo5fH%2BwgxON1ciExptFrrtvOi%2FMRG6nn5Dx31B1bTjC1oNCFNo8Gnf3q%2BnuqjuUrjJ9c%2BJEeO29PQ0BlydaSXbBDc8BUG6i6knmePKc5o0Zt19zRcYXMPYlNSmC2VeXtmMfokYautziOMiiTRER%2Byn0SGdIuXmpxaZ%2FPxXaf237UW2OwDYoSIJzehauTOSNnUNrj5tpPPto%3D&pass_ticket=J3NavZJ%2BxknVb4GqrfINn%2BLO4bLg4BTK02DUjYBzpOcEPyNw3xm3NEDkvqZcRYdO&wx_header=3
---

# **FastAPI 中间件实战**

> 写接口写多了，你会发现一个问题：每次出问题都要翻日志，翻半天还找不到是哪次请求出的错。今天教你用 FastAPI 中间件，把请求日志、耗时统计、IP 黑名单一次性搞定。

## 中间件是什么鬼

简单说，中间件就是每个请求进来和出去时都要经过的一道关卡。你可以在这里记录日志、统计时间、拦截黑名单，甚至加点骚操作。

FastAPI 的中间件基于 Starlette，写法很简洁：

```
1

2

3

4

5

6

  

@app.middleware("http")  
async def my_middleware(request: Request, call_next):  
    # 请求进来时做的事  
    response = await call_next(request)  
    # 请求出去时做的事  
    return response
```

看到没，就这几行。`call_next` 是核心，它负责把请求传给后面的处理函数。

## 实战一：请求日志中间件

最基础的需求，记录每个请求的详细信息。谁来了、要干啥、花了多久。

```
1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

  

import time  
from fastapi import Request  
import logging  
  
logging.basicConfig(level=logging.INFO)  
logger = logging.getLogger(__name__)  
  
@app.middleware("http")  
async def log_requests(request: Request, call_next):  
    start_time = time.time()  
      
    # 记录请求信息  
    logger.info(f"→ {request.method} {request.url.path}")  
      
    response = await call_next(request)  
      
    # 计算耗时  
    process_time = time.time() - start_time  
      
    # 记录响应信息  
    logger.info(f"← {response.status_code} | {process_time:.3f}s")  
      
    return response
```

跑起来你会发现，每次请求都有清晰的日志输出。接口挂了？先看状态码是不是 500。慢了？看耗时就知道。

## 实战二：耗时统计中间件

上面的代码已经带耗时了，但我们可以做得更专业一点。比如超过 1 秒的请求标红警告。

```
1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

  

@app.middleware("http")  
async def timing_middleware(request: Request, call_next):  
    start = time.perf_counter()  
    response = await call_next(request)  
    elapsed = time.perf_counter() - start  
      
    # 超过 1 秒标红  
    if elapsed > 1.0:  
        logger.warning(f"⚠️ 慢请求: {request.url.path} 耗时 {elapsed:.3f}s")  
    else:  
        logger.info(f"✓ {request.url.path} 耗时 {elapsed:.3f}s")  
      
    # 把耗时加到响应头里，方便前端调试  
    response.headers["X-Process-Time"] = str(elapsed)  
    return response
```

这里用了 `perf_counter()`，比 `time()` 更精准。还把耗时塞进了响应头，前端同学调接口时一眼就能看到。

## 实战三：IP 黑名单中间件

有些 IP 就是来搞事情的，直接拦在外面最省事。

```
1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

  

# 黑名单列表，实际项目中可以放在配置文件或 Redis  
BLACKLIST = {"192.168.1.100", "10.0.0.50"}  
   
@app.middleware("http")  
async def blacklist_middleware(request: Request, call_next):  
    client_ip = request.client.host  
      
    if client_ip in BLACKLIST:  
        logger.warning(f"🚫 拦截黑名单IP: {client_ip}")  
        return JSONResponse(  
            status_code=403,  
            content={"detail": "Access denied"}  
        )  
      
    return await call_next(request)
```

生产环境别写死 IP，放 Redis 里，动态更新。被攻击了直接加黑名单，不用重启服务。

## 实战四：三合一完整版

把上面三个功能整合起来，一个中间件搞定所有事：

```
1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

26

27

28

29

30

31

32

33

34

35

36

37

38

  

import time  
from fastapi import Request  
from fastapi.responses import JSONResponse  
import logging  
  
logger = logging.getLogger(__name__)  
BLACKLIST = set()  # 从配置文件读取  
  
@app.middleware("http")  
async def unified_middleware(request: Request, call_next):  
    # 1. IP 黑名单检查  
    client_ip = request.client.host  
    if client_ip in BLACKLIST:  
        return JSONResponse(status_code=403, content={"detail": "Forbidden"})  
      
    # 2. 开始计时  
    start = time.perf_counter()  
      
    # 3. 记录请求  
    logger.info(f"[{client_ip}] → {request.method} {request.url.path}")  
      
    # 4. 执行请求  
    try:  
        response = await call_next(request)  
    except Exception as e:  
        logger.error(f"💥 异常: {e}")  
        raise  
      
    # 5. 计算耗时  
    elapsed = time.perf_counter() - start  
      
    # 6. 记录响应  
    logger.info(f"[{client_ip}] ← {response.status_code} | {elapsed:.3f}s")  
      
    # 7. 添加响应头  
    response.headers["X-Process-Time"] = f"{elapsed:.3f}"  
      
    return response
```

这个中间件涵盖了：IP 拦截、请求记录、异常捕获、耗时统计、响应头注入。一行代码解决五个问题。

## 进阶：多个中间件的执行顺序

FastAPI 支持多个中间件，执行顺序是**从下往上**（后注册的先执行）。

```
1

2

3

4

5

  

# 先注册 A  
app.add_middleware(MiddlewareA)  
# 再注册 B  
app.add_middleware(MiddlewareB)  
# 执行顺序：B → A → 路由 → A → B
```

记住这个顺序，别把自己搞晕了。黑名单中间件应该放最外层，先拦截再处理。

## 写在最后

中间件是 FastAPI 里被低估的神器。很多人觉得写个装饰器也能实现，但中间件的优势在于**全局统一处理**，不用每个接口都加装饰器。

而且中间件可以访问完整的 Request 和 Response 对象，能做的比装饰器多得多。日志、监控、限流、鉴权、CORS，统统可以扔中间件里。

下次写接口的时候，先想想哪些事是每次请求都要做的，把它们抽成中间件。代码会干净很多，排查问题也快很多。

毕竟，写代码不只是为了跑起来，更是为了出问题的时候能快速定位。中间件就是你的第一道防线。

你还知道哪些好的编程学习方法？欢迎在评论区分享！

开发必备神器

1. **[强大的编程神器：IntelliJ IDEA 免费激活](https://mp.weixin.qq.com/s?__biz=Mzg3OTg1MTM4Ng==&mid=2247483749&idx=1&sn=d99a8ae313f28a42358c3fb165b62814&scene=21#wechat_redirect)**
2. [轻松安装 MySQL 数据库](https://mp.weixin.qq.com/s?__biz=Mzg3OTg1MTM4Ng==&mid=2247483725&idx=1&sn=820635253a6dfd02b46a01f30f9286e4&scene=21#wechat_redirect)
3. **[数据库管理神器 - Navicat 的奇妙之旅](https://mp.weixin.qq.com/s?__biz=Mzg3OTg1MTM4Ng==&mid=2247483742&idx=1&sn=99c4a63592a4220b0325fada3e7cd014&scene=21#wechat_redirect)**
4. **[宝藏级的开发手册，前后端开发语言都有，自己也在用](https://mp.weixin.qq.com/s?__biz=Mzg3OTg1MTM4Ng==&mid=2247483761&idx=1&sn=7841984e811cd32951c9deb47f925476&scene=21#wechat_redirect)**

·················END·················