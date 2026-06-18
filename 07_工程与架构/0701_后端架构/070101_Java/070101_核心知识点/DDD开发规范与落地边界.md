# DDD 开发规范与落地边界

## 目标

这个知识点要回答：

> Java 什么时候应该用 DDD，使用 DDD 时每类对象怎么写，什么时候不应该过度设计。

## 什么时候使用 DDD

| 场景 | 是否建议 DDD | 判断 |
|---|---|---|
| 简单 CRUD 后台 | 不建议完整 DDD | 领域规则少，映射成本大于收益 |
| 核心交易、订单、库存、计费 | 建议 | 状态、规则、约束、生命周期复杂 |
| 规则平台、审批流、治理平台 | 建议 | 业务概念需要长期演进 |
| 纯数据搬运、报表查询 | 谨慎 | 多数是流程和查询，不一定有丰富领域行为 |
| 原型验证 | 不建议完整 DDD | 先用清晰分层，等规则稳定再抽领域 |

## DDD 对象职责

| 对象 | 写什么 | 不写什么 |
|---|---|---|
| Entity | 有唯一标识和生命周期的业务对象 | 不写数据库访问和 HTTP 调用 |
| Value Object | 无身份、不可变、表达业务约束的值 | 不做可变状态流转 |
| Aggregate | 一组必须保持一致性的对象边界 | 不把所有对象塞进一个大聚合 |
| Aggregate Root | 聚合唯一入口，控制内部一致性 | 不暴露内部对象随意修改 |
| Domain Service | 不自然属于某个实体的领域规则 | 不编排事务和外部调用 |
| Repository | 领域对象的保存和获取接口 | 不暴露 JPA/MyBatis 细节 |
| Application Service | 用例编排、事务、调用领域和端口 | 不承载核心领域规则 |
| Domain Event | 领域状态变化后产生的业务事实 | 不直接等同 MQ 消息格式 |

## 推荐包结构

```text
domain/
  model/
    aggregate/
    entity/
    valueobject/
  service/
  repository/
  event/
application/
  command/
  query/
  service/
  port/
    incoming/
    outgoing/
infrastructure/
  persistence/
  external/
  messaging/
interfaces/
  rest/
```

## 代码书写规则

| 规则 | 说明 |
|---|---|
| 领域对象表达业务语言 | 类名和方法名应该是业务动作，不是数据库动作 |
| 聚合维护不变量 | 状态变更必须经过聚合根方法 |
| 应用服务开启事务 | 事务边界放用例层，不放 Controller |
| 仓储接口属于领域或应用层 | 实现放基础设施层 |
| DTO 不进入领域层 | 外部请求先转 Command，再调用领域模型 |
| 数据库 Entity 和领域 Entity 默认分开 | 核心系统优先隔离，简单系统可合并但要知道代价 |

## 示例

```java
public class Order {
    private OrderId id;
    private OrderStatus status;
    private List<OrderItem> items;

    public static Order create(UserId userId, List<OrderItem> items) {
        if (items.isEmpty()) {
            throw new DomainException("订单明细不能为空");
        }
        return new Order(new OrderId(), OrderStatus.CREATED, items);
    }

    public void cancel() {
        if (!status.canCancel()) {
            throw new DomainException("当前状态不能取消订单");
        }
        this.status = OrderStatus.CANCELED;
    }
}
```

## 当前文章补充

| 文章 | 补充位置 | 处理 |
|---|---|---|
| SpringBoot 的 3 种六边形架构应用方式 | DDD + 六边形的分层位置 | 补充：DDD 是复杂业务系统的一档选择，不是所有项目默认选择 |
| SPI 文章 | 扩展点思想 | 不直接属于 DDD，但可补“领域能力通过接口扩展”的意识 |
| 动态 jar 文章 | 插件化生命周期 | 不属于 DDD 主线，作为平台型系统扩展机制 |

## 还缺什么

| 缺口 | 影响 |
|---|---|
| 聚合边界案例 | 还不能稳定判断哪些对象应该在一个聚合里 |
| 领域事件实践 | 还不能指导事务内事件、事务后事件和 MQ 事件的关系 |
| 防腐层 | 还不能指导外部系统模型如何隔离 |
| 查询模型 | 还不能指导 DDD 下复杂查询是否绕过领域模型 |
