> 已吸收至：[[07_工程与架构/0701_后端架构/070103_Python/070103_核心知识点/Python来源校准与降权准则|Python来源校准与降权准则]]、[[07_工程与架构/0701_后端架构/070103_Python/070103_知识地图|070103_Python知识地图]]

---
title: tenacity，建议直接封神！
author: 折页的风
date:
url: https://mp.weixin.qq.com/s?__biz=MzA3ODk1Mzg0Mg==&mid=2649860914&idx=1&sn=c89f57f40aae881d6f5653d8b14582c0&chksm=864710980a6f9bbd20a0d2b5dcbeb143c8326aecee17ec25a95f3c6b97da0032c8bfc922280f&mpshare=1&scene=24&srcid=11197gLKuz32MOlLxgj7rkQI&sharer_shareinfo=2c9f7d4161d01ffd6fb1db7addaac5b9&sharer_shareinfo_first=2c9f7d4161d01ffd6fb1db7addaac5b9#rd
---

在日常的代码开发过程中，重试逻辑往往扮演着比较重要的角色。

特别是在业务超时、请求阻塞等业务场景需要对某一块逻辑加上重试机制以保持业务的连贯性。

在我们的Python编码中就有这样一个非常便捷的工具库；它可以帮助我们很简单的就能实现一个代码块中重试机制。

你可以选择pip或者conda命令安装这个工具库，它就是 Tenacity 模块。

```
pip install tenacity
```

下面我们通过一个简单的代码块来演示一下它的用法。

在安装完成之后可以直接使用注解的方式，将 @retry注解直接添加在某个函数的前面；

可以在注解中通过参数设置的方式直接设置业务逻辑重试的次数、重试间隔时间等相关信息，自动帮助我们完成重试机制；

```
from tenacity import retry, stop_after_attempt, wait_exponential, retry_if_exception_type
import requests  # 用网络请求模拟易失败操作

# 1. 给函数加装饰器，定义重试规则
@retry(
    stop=stop_after_attempt(3),  # 最多重试3次
    wait=wait_exponential(multiplier=1, min=1, max=5),  # 重试间隔：1s → 2s → 4s（最多5s）
    retry=retry_if_exception_type(requests.exceptions.RequestException)  # 只有请求异常时才重试
)
deffetch_url(url):
    response = requests.get(url, timeout=3)
    response.raise_for_status()# 触发4xx/5xx错误的异常
return response.text

# 2. 调用函数（失败会自动重试）
try:
    result = fetch_url("https://example.com/nonexistent")# 故意访问不存在的地址
print("请求成功：", result[:50])
exceptExceptionas e:
print(f"3次重试后仍失败：{e}")
```

最后在调用这个添加了重试机制的函数时就会触发重试机制，这种方式避免我们对每一个函数去开发复杂的重试机制。

tenacity 模块更多的说明及用途，欢迎大家在留言讨论！

**推荐阅读**：

* • [bowler，一个超好用的 python 工具！](https://mp.weixin.qq.com/s?__biz=MzA3ODk1Mzg0Mg==&mid=2649860908&idx=1&sn=6a3667df3254e45bf16c4ecab8142752&scene=21#wechat_redirect)
* • [magick，一个超酷的 python 工具！](https://mp.weixin.qq.com/s?__biz=MzA3ODk1Mzg0Mg==&mid=2649860903&idx=1&sn=1f77b3fd92b9f90faba8db6881617291&scene=21#wechat_redirect)
* • [轻到离谱的 fagr，热力图轻松玩！](https://mp.weixin.qq.com/s?__biz=MzA3ODk1Mzg0Mg==&mid=2649860893&idx=1&sn=46eb66868893dd00b45959ae7c49bb4d&scene=21#wechat_redirect)
* • [轻量级msnoise，无GPU也能透视地壳！](https://mp.weixin.qq.com/s?__biz=MzA3ODk1Mzg0Mg==&mid=2649860893&idx=2&sn=f17a4b85db6001b3867b09daf5337d3d&scene=21#wechat_redirect)