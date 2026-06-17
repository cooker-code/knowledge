---
title: 更轻更自由，还要什么自行车：别急上Supabase，先试试它的内核PostgREST
author: 云引
date: 
url: https://mp.weixin.qq.com/s?__biz=MzE5OTE1MjE5MA==&mid=2247484551&idx=1&sn=afaac7bd49c8905707491d8d0d89381c&chksm=9771ac18308bf9d725b1c5cb194108329b42bbe1676ac4e83239f6786f7b0680bcf3725fc793&mpshare=1&scene=24&srcid=1202GeFsQFos33EQCTDEj0QK&sharer_shareinfo=a3b6245c91f75ff517518a00bc844b45&sharer_shareinfo_first=a3b6245c91f75ff517518a00bc844b45#rd
---

## 引言

在现代应用开发中，“胖数据库”架构正重新焕发生机。这种模式将复杂的业务逻辑、数据验证和访问权限控制尽可能地下沉到数据库层，让后端应用更多地扮演一个轻量的胶水层或直接通过HTTP暴露数据库能力。这种架构特别适用于内部管理系统、数据中台、以及业务模型相对稳定但查询需求多变的场景，它能极大地减少中间层的代码量，提升开发效率，并保证数据操作的性能与一致性。

在这一趋势下，PostgREST作为一个独立的开源项目，完美地践行了“胖数据库”理念。它不是一个ORM，也不是一个代码生成器，而是一个**将 PostgreSQL 数据库直接转换成 RESTful API 的 Web 服务器**。让我们深入了解一下它的核心能力。

### PostgREST 的核心能力

**1. 灵活且强大的查询参数**  
PostgREST 提供了一套极其丰富的查询语法，让前端能够精准地获取所需数据。

* • **条件过滤**：直接在 URL 参数中实现。

+ • 等值/不等值：`eq`, `neq` (例如 `/users?age=gt.18`)
+ • 范围比较：`gt`/`gte`, `lt`/`lte`
+ • 模糊匹配：`like`（区分大小写）, `ilike`（不区分大小写）(例如 `/users?name=ilike.*john*`)
+ • 集合判断：`in.(...)` (例如 `/users?id=in.(1,2,3)`)
+ • 空值判断：`is.null`, `not.is.null`

* • **逻辑组合**：支持 `and` 和 `or` 逻辑运算，构建复杂查询。

+ • `or=(a.eq.1,b.ilike.*x*)`

* • **选择与排序**：只返回需要的字段，并按指定方式排序。

+ • `select=id,name,created_at`
+ • `order=updated_at.desc.nullslast` (按更新时间降序，空值排在最后)

* • **标准分页**：使用 HTTP `Range` 头进行分页，符合 RFC 规范，并可通过 `Prefer: count=exact` 获取数据总数。

  ```
  // 请求头  
  Range-Unit: items  
  Range: 0-9  
  Prefer: count=exact  
    
  // 响应头  
  Content-Range: 0-9/123
  ```

**2. 直接调用 RPC / 存储过程**  
对于复杂计算或特定业务逻辑，可以直接将 PostgreSQL 函数作为 API 端点调用。

* • **POST 调用**：使用 `POST /rpc/function_name`，请求体为 JSON 格式的参数。使用 `Prefer: params=single-object` 可以简化单个 JSON 对象参数的传递。

  ```
  # 调用一个名为 "search_products" 的函数  
  POST /rpc/search_products HTTP/1.1  
  Prefer: params=single-object  
  Content-Type: application/json  
    
  {"query": "wireless", "max_price": 100}
  ```
* • **GET 调用**：如果函数被标记为 `stable` 或 `immutable`，可以使用 GET 请求并通过 Query String 传参。

**3. 事务与请求钩子**  
这是 PostgREST 的高级特性，允许你在数据库事务层面介入请求生命周期。

* • **`db-pre-request`**：这是一个在主体查询执行前被调用的 PostgreSQL 函数。你可以在这里：

+ • 设置数据库角色：`set local role to ...;`
+ • 设置搜索路径：`set local search_path to ...;`
+ • 写入响应头：`perform set_config('response.headers', '[{"X-Custom-Header": "Value"}]', true);`
+ • 验证权限并中断非法请求：`raise exception 'unauthorized';`

* • **`db-hoisted-tx-settings`**：确保某些配置（如 `role` 和 `search_path`）在整个事务中保持一致，避免因连接池复用导致的状态混乱。

**4. 安全与多租户基石**  
PostgREST 与 PostgreSQL 的原生安全机制深度集成。

* • **JWT 集成**：配置 `jwt-secret` 后，PostgREST 会自动解码 JWT 并将其声明（claims）注入到数据库会话中，可通过 `request.jwt.claims` 访问。
* • **行级安全策略（RLS）**：这是实现数据隔离的关键。在表上启用 RLS 后，可以创建策略，利用 JWT 中的信息进行过滤。

  ```
  -- 例如，一个多租户策略，确保用户只能访问自己租户的数据  
  create policy "用户仅可访问其租户数据" on my_table  
    for all using (  
      tenant_id::text = current_setting('request.jwt.claims', true)::json->>'tenant_id'  
    );  
    
  -- 或者，一个用户只能访问自己数据的策略  
  create policy "用户仅可访问自身数据" on my_table  
    for all using (  
      user_id::text = current_setting('request.jwt.claims', true)::json->>'sub'  
    );
  ```
* • **多 Schema 支持**：通过 `db-schemas` 配置，可以同时暴露多个业务 Schema 的 API，并通过 `search_path` 控制优先级，实现清晰的模块划分。

**5. 生态友好**

* • **纯 REST**：无任何侵入性，天然适配任何语言、任何框架的 HTTP 客户端。
* • **性能稳健**：基于预编译语句和连接池，性能开销极低。它将复杂查询和权限控制都留在 SQL 层解决，保证了效率和安全性。

### Supabase 与 PostgREST 的关系：引擎与整车

理解了 PostgREST 的能力后，我们再回头看 Supabase，就会发现它们的关系非常清晰。

**Supabase Database 的 HTTP 接入层，其核心就是 PostgREST。** 你在 Supabase 中使用的绝大部分数据查询、RPC 调用、分页和 RLS 能力，都直接由 PostgREST 提供。

Supabase 在其上做了一层优秀的“语法糖”包装。例如，它提供了 `auth.uid()` 和 `auth.jwt()` 这样的便捷函数。但这些函数的本质，仍然是查询 `request.jwt.claims`。你自己在纯 PostgREST 环境中可以轻松实现等价功能：

```
-- Supabase 的 auth.uid() 大致等价于：  
current_setting('request.jwt.claims', true)::json->>'sub'  
  
-- Supabase 的 auth.jwt() 大致等价于：  
current_setting('request.jwt.claims', true)::json
```

因此，结论是：**如果你的核心需求只是一个“数据库即 API”的轻量、高性能接入层，那么直接使用 PostgREST 就能获得 Supabase 在数据 API 方面 80% 以上的核心能力，而且没有额外的技术栈负担和供应商锁定风险。**

### 轻量使用 PostgREST 的实践指南

如何上手？以下是一些常见操作的示例：

1. 1. **查询数据**：

   ```
   # 查询状态为 ACTIVE 且姓名包含 "zhang" 的用户，只返回 id 和 name，按创建时间降序排列，取前50条  
   GET /users?status=eq.ACTIVE&name=ilike.*zhang*&select=id,name,created_at&order=created_at.desc.nullslast  
   Header:  
     Range-Unit: items  
     Range: 0-49  
     Prefer: count=exact
   ```
2. 2. **调用 RPC**：

   ```
   # 调用一个复杂的报表函数  
   POST /rpc/generate_sales_report HTTP/1.1  
   Prefer: params=single-object  
   Content-Type: application/json  
     
   {"tenant_id":"t01", "from_date":"2024-01-01", "to_date":"2024-12-31"}
   ```
3. 3. **配置权限/RLS**：

* • 为所有表启用 RLS：`ALTER TABLE your_table ENABLE ROW LEVEL SECURITY;`
* • 为每种操作（SELECT, INSERT, UPDATE, DELETE）创建策略，在 `USING` 子句中从 `request.jwt.claims` 读取用户身份进行过滤。

4. 4. **使用 `db-pre-request` 设置动态上下文**：

   ```
   CREATE OR REPLACE FUNCTION pre_request() RETURNS void AS $$  
   DECLARE  
     role text;  
   BEGIN  
     role := current_setting('request.jwt.claims', true)::json->>'role';  
     IF role IS NOT NULL THEN  
       EXECUTE format('set local role to %I', role);  
     END IF;  
     -- 设置租户特定的搜索路径  
     PERFORM set_config('search_path', 'tenant_schema, public', false);  
   END;  
   $$ LANGUAGE plpgsql;
   ```

   然后在 PostgREST 配置中指向此函数。

### 与遗留系统的平滑集成方案

对于已经存在 Spring Boot、.NET 或类似框架构建的遗留系统，推倒重来是不可行的。但我们可以采用一种“侧车”或“薄代理”模式，将 PostgREST 集成进来。

点击“阅读原文”，跳转GitHub查看简版pgrest-client项目源码。

**目标**：在不破坏现有业务和安全体系的前提下，为新的或特定的数据访问需求引入 PostgREST 的高效开发模式。

**参考 Supabase-JS 的设计，我们可以自建一个“三件套”：**

1. 1. **查询构建器**：在客户端（可以是前端，也可以是后端服务内部）创建一个链式调用的查询构建器，用于拼装 PostgREST 的查询参数。它可以将 `.eq('status', 'ACTIVE').ilike('name', '*zhang*').select('id,name').range(0, 49)` 这样的调用，最终转换成上述的 URL。
2. 2. **代理服务**：在现有的后端应用中，创建一个薄薄的 HTTP 代理层。这个层负责：

* • **鉴权桥接**：将现有的 Session 或 Token 机制，转换成 PostgREST 所需的 JWT。
* • **请求转发**：将来自客户端的请求（附带了查询参数和分页头）转发给 PostgREST。
* • **响应格式化**：可能需要对响应进行一些处理，比如将数据库返回的 `snake_case` 字段名转换为后端惯用的 `camelCase`。

3. 3. **客户端接口**：使用像 Feign（Java）、Axios（Node.js）等库，封装对 PostgREST 的常见操作（GET, POST, PATCH, DELETE），并确保能正确传递 `Prefer` 和 `Range` 等头部。

**集成效果**：

* • **最小改动**：原有业务代码和实体类基本不受影响。
* • **迭代加速**：所有简单的 CRUD 和复杂查询都可以通过 PostgREST 快速实现，后端只需编写 RPC 函数处理最复杂的业务逻辑。
* • **架构清晰**：数据权限和复杂过滤逻辑牢牢固定在数据库层，避免了业务服务层的代码膨胀和重复。

### 总结

Supabase 是一个非常优秀的一站式 BaaS 平台，但它的强大很大程度上源于其底层坚固的“引擎”——PostgREST。

当你评估需求后，发现核心诉求只是一个**高性能、无侵入、安全可靠的“数据库 RESTful API 网关”** 时，直接采用 PostgREST 是一个非常明智和高效的选择。它本身已经足够强大和成熟。

通过施加一些简单的工程化约束——合理地设计 RLS 策略、使用 JWT 进行身份传递、利用 `db-pre-request` 钩子管理上下文、以及遵循其分页规范——你可以用极轻量的方式，构建出一个既灵活又稳固的数据库应用层。

即便是面对庞大的遗留系统，也无需畏惧。仿照 supabase-js 的思路，封装一层薄薄的代理和查询构建器，就能在不伤筋动骨的情况下，为你的技术栈注入 PostgREST 所带来的“快、稳、准”的数据库操作体验。