> 已吸收至：[[07_工程与架构/0701_后端架构/070103_Python/070103_核心知识点/Python来源校准与降权准则|Python来源校准与降权准则]]、[[07_工程与架构/0701_后端架构/070103_Python/070103_知识地图|070103_Python知识地图]]

---
title: 搭建FastAPI服务要注意的问题（一）
author: 吃着火锅K着歌
date:
url: https://mp.weixin.qq.com/s?__biz=MzIyMTcyNzQ5Mg==&mid=2247487231&idx=1&sn=1dcf36bb7057d30d99072d0e68504f19&chksm=e96b0483d029ed18469567c930e7aba64ae8f0370a97cba0bedb9a4353a83fa352cc4135f667&mpshare=1&scene=24&srcid=0204BfRgpjkexxDwsrq7ItwM&sharer_shareinfo=166ed5279bc637e5a9eab649afdd5ec7&sharer_shareinfo_first=166ed5279bc637e5a9eab649afdd5ec7#rd
---

近些年来FastAPI已经成了Python后端开发的主选之一。它的易用性太香了。异步并发支持、Pydantic自动数据校验、中间件机制、统一异常处理、自动生成接口文档，还有灵活的依赖注入。正因为上手太简单，很多团队急于落地功能，往往跳过系统设计环节，后期随着项目复杂度提升，各种诡异问题也就出来了。今天我们就来看看搭建FastAPI服务时最基础、容易影响后期维护的问题。

对于新手来说搭建FastAPI的通病：优先实现功能，而忽略了结构设计。初期几行代码还好，一旦接口数量超过20个、涉及多业务模块，就会出现「找接口要10分钟」「重复代码遍地」「循环导入报错」的问题。其实可以按功能职责拆分目录，让我们看个适合大多数场景的结构：

```
# 项目目录结构（推荐）app/├── main.py          # 核心入口：创建app、生命周期配置、注册中间件/异常处理器├── validators/      # 通用校验：IDs、枚举、业务规则（供Pydantic复用）├── middleware/      # 中间件：全局拦截（如请求体大小限制、跨域、日志）├── error_handlers/  # 异常处理：自定义异常与HTTP响应映射├── utilities/       # 工具函数：响应封装、分页、格式化、加密等├── models/          # 数据模型：Pydantic请求/响应模型（统一数据格式）├── services/        # 业务逻辑：核心逻辑封装（与接口解耦）├── clients/         # 客户端：数据库、HTTP客户端、日志/追踪适配器（如Redis、Postgres）├── factories/       # 工厂类：依赖注入适配（统一管理客户端/服务实例）├── endpoints/       # 接口路由：按业务域分组（如accounts/、orders/）│   ├── accounts/    # 账户模块│   │   ├── root.py  # 账户主路由（/accounts）│   │   └── orders/  # 子路由（/accounts/{account_id}/orders）│   └── orders/      # 订单独立模块（可选）└── tests/           # 测试用例：单元测试/集成测试（依赖覆盖）
```

其中对于父子路由的关联设计，很多人会出现路由前缀混乱的问题，让我们举个实战的例子：需要实现下面的嵌套路由，确保访问订单必须关联账户ID

```
/accounts (账户列表)/accounts/{account_id} (单个账户)/accounts/{account_id}/orders （该账户的订单列表）/accounts/{account_id}/orders/{order_id} （单个订单）
```

首先看订单路由/app/endpoints/accounts/order/root.py

```
from fastapi import APIRouter, Pathfrom uuid import UUIDfrom typing import Annotated# 子路由：前缀为 /{account_id}/orders（依赖父路由前缀）orders_router = APIRouter(    prefix="/{account_id}/orders",    tags=["orders"]  # 接口文档分组)# 订单列表接口@orders_router.get("/")async def list_orders(    # 路径参数校验：确保account_id是UUID格式    account_id: Annotated[UUID, Path(description="账户ID")],    last_id: str | None = None,  # 分页参数（可选）    limit: int = 50  # 默认每页50条):    # 实际场景中，这里调用services层的订单查询逻辑    return {        "account_id": str(account_id),        "items": [],  # 订单列表（实际业务填充）        "next": None  # 下一页标识（分页用）    }# 创建订单接口@orders_router.post("/")async def create_order(    account_id: Annotated[UUID, Path(description="账户ID")]):    # 实际场景：校验账户存在，再创建订单    return {"account_id": str(account_id), "ok": True, "msg": "订单创建成功"}
```

账户路由:app/endpoints/accounts/root.py

```
from fastapi import APIRouter, Depends, Pathfrom uuid import UUIDfrom typing import Annotated# 导入子路由（订单路由）from .orders.root import orders_router# 父路由：前缀为 /accountsaccounts_router = APIRouter(    prefix="/accounts",    tags=["accounts"])# 依赖项：加载账户信息（所有子路由都会继承该依赖）async def load_account(    account_id: Annotated[UUID, Path(description="账户ID")]):    # 实际场景：从数据库查询账户，不存在则抛异常    return {"id": account_id, "status": "active"}# 账户列表接口@accounts_router.get("/")async def list_accounts():    # 实际场景：分页查询账户列表    return [{"id": "12345", "name": "测试账户1"}, {"id": "67890", "name": "测试账户2"}]# 单个账户详情接口@accounts_router.get("/{account_id}")async def get_account(account: Annotated[dict, Depends(load_account)]):    return account# 关联子路由（订单路由）# 关键：通过dependencies参数，让所有子路由都依赖load_account（自动校验账户存在）accounts_router.include_router(    orders_router,    dependencies=[Depends(load_account)])
```

如此设计，路由按业务拆分，找接口很方便；子路由依赖父路由的校验逻辑，避免了代码重复。子路由可以复用

本篇内容我们了解了对于FastAPI通用项目结构和父子路由的设计方式，避免在后期维护时查找接口混乱。下次我们讲看看FastAPI的依赖注入问题