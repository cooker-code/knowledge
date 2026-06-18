---
title: Apache Commons EnumUtils：让枚举操作从繁琐到优雅的蜕变
author: Java程序员-晴天Y
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzk1NzU1NDIzMA==&mid=2247484351&idx=1&sn=fda39d6dcea0141cf605e434a52aca3d&chksm=c213874046d5dd968f6f680f269575ee7be4204e5fcb38de34531bd5c74a9bf1718d7006662a&mpshare=1&scene=24&srcid=0909SWnKRpTDaYSOhnlVPOPX&sharer_shareinfo=64222d9e0e9d1632ec88d4ecab0bb773&sharer_shareinfo_first=64222d9e0e9d1632ec88d4ecab0bb773#rd
---

开发中有遇到过系统状态码的定义问题吗？

开发中有遇到过业务类型的定义吗？

开发中对于这些定义是直接写成1、2、3还是有专门的定义呢？

那你在维护的时候是不是会感觉到头大？

虽然JDK 中对枚举已经是原生支持的，但是在使用的时候显得有些 "简陋"—— 字符串与枚举的互转、枚举值校验、集合化处理等常见操作，都需要开发者手写大量重复代码，不仅效率低下，还容易埋下空指针、逻辑疏漏等隐患。

巧了，Apache Commons 家族中的 EnumUtils 工具类，正是为解决这些痛点而来。

今天我们就来深入探索这个工具类，看看它如何让枚举操作实现从繁琐到优雅的蜕变。

## 一、枚举操作的 "手写困境"

先看一个典型场景：定义一个支付方式枚举，然后实现 "字符串转枚举"" 校验枚举合法性 ""获取所有枚举描述" 等功能。用原生 JDK 实现会是这样：

```
// 支付方式枚举public enum PayType {    ALIPAY("支付宝"),    WECHAT("微信支付"),    UNIONPAY("银联支付");    private final String desc;    PayType(String desc) {        this.desc = desc;    }    public String getDesc() {        return desc;    }    // 字符串转枚举（需处理null和不匹配情况）    public static PayType fromString(String name) {        if (name == null) {            return null;        }        for (PayType type : values()) {            if (type.name().equals(name)) {                return type;            }        }        throw new IllegalArgumentException("无效的支付方式：" + name);    }    // 校验字符串是否为有效枚举值    public static boolean isValid(String name) {        try {            fromString(name);            return true;        } catch (IllegalArgumentException e) {            return false;        }    }    // 获取所有枚举描述    public static List<String> getAllDescs() {        List<String> descs = new ArrayList<>();        for (PayType type : values()) {            descs.add(type.getDesc());        }        return descs;    }}
```

这段代码中，仅 3 个辅助方法就写了 30 多行，而且每个枚举类都要重复这套逻辑。更麻烦的是，手写代码容易出现疏漏：比如`fromString`未考虑大小写差异，`isValid`用异常控制流程影响性能，`getAllDescs`每次调用都创建新集合等。

而用 EnumUtils 处理这些需求，代码会变得异常简洁：

```
// 支付方式枚举（无需手写辅助方法）public enum PayType {    ALIPAY("支付宝"),    WECHAT("微信支付"),    UNIONPAY("银联支付");    private final String desc;    PayType(String desc) { this.desc = desc; }    public String getDesc() { return desc; }}// 字符串转枚举（一行搞定）PayType type = EnumUtils.getEnum(PayType.class, "ALIPAY");// 校验枚举合法性（无需try-catch）boolean valid = EnumUtils.isValidEnum(PayType.class, "WECHAT");// 获取所有枚举值（直接用于遍历）List<PayType> allTypes = EnumUtils.getEnumList(PayType.class);
```

无需在枚举类中编写任何辅助方法，通过 EnumUtils 的静态方法即可完成所有操作。代码量减少 70%，且经过 Apache Commons 的严格测试，稳定性远超手写逻辑。

## 二、EnumUtils 核心功能：枚举操作的 "全能工具箱"

EnumUtils 提供了 10 余种实用方法，覆盖枚举操作的全场景需求，按功能可分为五大类：

### 1. 安全的枚举值获取

从字符串或索引获取枚举时，JDK 的`valueOf`方法在匹配失败时会抛出异常，而 EnumUtils 提供了更灵活的获取方式：

#### （1）按名称获取枚举（支持 null 安全）

```
// 基础用法：名称匹配返回枚举，否则返回nullPayType alipay = EnumUtils.getEnum(PayType.class, "ALIPAY"); // 正常返回ALIPAYPayType invalid = EnumUtils.getEnum(PayType.class, "CASH"); // 返回null// 忽略大小写获取（适合处理用户输入）PayType wechat = EnumUtils.getEnumIgnoreCase(PayType.class, "wechat"); // 返回WECHAT
```

相比`PayType.valueOf("ALIPAY")`，`getEnum`在名称不匹配时返回 null 而非抛出异常，避免了繁琐的 try-catch 块，特别适合处理外部传入的不确定值。

#### （2）按索引获取枚举（数组式访问）

```
// 获取第一个枚举值（类似数组下标0）PayType first = EnumUtils.getEnum(PayType.class, 0); // ALIPAY// 获取枚举值的索引位置int index = EnumUtils.indexOf(PayType.class, PayType.UNIONPAY); // 返回2
```

### 2. 枚举合法性校验

在接收外部参数（如接口入参、数据库字段）时，往往需要先校验其是否为有效的枚举值：

```
// 校验字符串是否为有效枚举名称boolean isWechatValid = EnumUtils.isValidEnum(PayType.class, "WECHAT"); // trueboolean isCashValid = EnumUtils.isValidEnum(PayType.class, "CASH"); // false// 校验字符串是否为有效枚举名称（忽略大小写）boolean isAlipayValid = EnumUtils.isValidEnumIgnoreCase(PayType.class, "alipay"); // true// 校验枚举实例是否在指定枚举类中（防止跨枚举类比较）boolean contains = EnumUtils.contains(PayType.class, PayType.ALIPAY); // true
```

`isValidEnum`方法替代了手写的循环判断或 try-catch 校验，让参数校验逻辑更清晰。例如在接口层验证入参：

```
@PostMapping("/pay")public Result pay(@RequestParam String payType) {    if (!EnumUtils.isValidEnum(PayType.class, payType)) {        return Result.error("无效的支付方式：" + payType);    }    // 业务处理...}
```

### 3. 枚举集合化处理

EnumUtils 提供了将枚举转换为集合或映射的方法，方便进行批量操作：

#### （1）获取枚举值列表

```
// 获取所有枚举值的列表（ArrayList）List<PayType> allTypes = EnumUtils.getEnumList(PayType.class);// 遍历列表获取描述（配合Stream更灵活）List<String> allDescs = allTypes.stream()    .map(PayType::getDesc)    .collect(Collectors.toList()); // ["支付宝", "微信支付", "银联支付"]
```

`getEnumList`返回的是`ArrayList`实例，比枚举的`values()`方法（返回数组）更适合集合操作，尤其适合需要对枚举值进行过滤、转换的场景。

#### （2）创建名称到枚举的映射

```
// 创建名称到枚举的映射（Map<String, PayType>）Map<String, PayType> typeMap = EnumUtils.getEnumMap(PayType.class);// 快速查找（适合频繁查询场景）PayType unionpay = typeMap.get("UNIONPAY");
```

当需要多次通过名称查询枚举时，先创建映射可以避免重复遍历枚举数组，提升性能。

### 4. 带自定义字段的枚举处理

实际开发中，枚举常包含自定义字段（如编码、描述），EnumUtils 配合反射或 Stream 可轻松实现按自定义字段的操作：

```
// 定义带编码的枚举public enum OrderStatus {    PENDING(1, "待支付"),    PAID(2, "已支付"),    SHIPPED(3, "已发货");    private final int code;    private final String desc;    OrderStatus(int code, String desc) {        this.code = code;        this.desc = desc;    }    public int getCode() { return code; }    public String getDesc() { return desc; }}// 工具方法：通过编码查找枚举public static OrderStatus getByCode(int code) {    // 用EnumUtils获取所有枚举值，再过滤匹配的编码    return EnumUtils.getEnumList(OrderStatus.class).stream()        .filter(status -> status.getCode() == code)        .findFirst()        .orElse(null);}// 使用示例OrderStatus paid = getByCode(2); // 返回PAID
```

通过`EnumUtils.getEnumList`获取枚举列表后，配合 Stream API 可轻松实现按编码、描述等自定义字段的查找、过滤，避免为每个枚举类手写查找方法。

### 5. 枚举序列化辅助

在 JSON 序列化或数据库交互时，枚举的转换常需要特殊处理，EnumUtils 可简化这一过程：

```
// Jackson反序列化枚举时处理未知值public class OrderStatusDeserializer extends StdDeserializer<OrderStatus> {    public OrderStatusDeserializer() {        super(OrderStatus.class);    }    @Override    public OrderStatus deserialize(JsonParser p, DeserializationContext ctxt) throws IOException {        String value = p.getValueAsString();        // 用EnumUtils判断是否有效，无效时返回默认值        return EnumUtils.isValidEnum(OrderStatus.class, value)             ? EnumUtils.getEnum(OrderStatus.class, value)            : OrderStatus.PENDING; // 未知状态默认待支付    }}
```

通过 EnumUtils 在反序列化时进行校验，能优雅地处理外部传入的未知枚举值，避免解析失败。

## 三、实战案例：用 EnumUtils 优化订单状态流转

以电商系统的订单状态流转功能为例，展示 EnumUtils 如何简化业务代码。

**业务需求**：

1. 验证状态转换是否合法（如待支付→已支付是合法的，已取消→已发货是非法的）
2. 接收前端传入的状态字符串，转换为枚举值
3. 提供所有状态的下拉框选项（包含编码、名称、描述）

**传统实现（无 EnumUtils）**：

```
@Servicepublic class OrderService {    // 定义合法的状态转换规则    private static final Map<OrderStatus, List<OrderStatus>> VALID_TRANSITIONS;    static {        VALID_TRANSITIONS = new HashMap<>();        VALID_TRANSITIONS.put(OrderStatus.PENDING, Arrays.asList(OrderStatus.PAID, OrderStatus.CANCELLED));        VALID_TRANSITIONS.put(OrderStatus.PAID, Arrays.asList(OrderStatus.SHIPPED, OrderStatus.REFUNDED));        // 其他状态转换规则...    }    // 转换订单状态    public void changeStatus(Long orderId, String statusStr) {        // 字符串转枚举（手写循环）        OrderStatus newStatus = null;        for (OrderStatus status : OrderStatus.values()) {            if (status.name().equals(statusStr)) {                newStatus = status;                break;            }        }        if (newStatus == null) {            throw new IllegalArgumentException("无效的状态：" + statusStr);        }        // 查询当前状态（省略DAO操作）        OrderStatus currentStatus = getCurrentStatus(orderId);        // 校验转换是否合法（手写包含判断）        List<OrderStatus> allowed = VALID_TRANSITIONS.getOrDefault(currentStatus, Collections.emptyList());        boolean canTransition = false;        for (OrderStatus status : allowed) {            if (status == newStatus) {                canTransition = true;                break;            }        }        if (!canTransition) {            throw new IllegalStateException("不允许从" + currentStatus.getDesc() + "转换到" + newStatus.getDesc());        }        // 执行状态更新...    }    // 获取状态下拉框选项    public List<Map<String, Object>> getStatusOptions() {        List<Map<String, Object>> options = new ArrayList<>();        for (OrderStatus status : OrderStatus.values()) {            Map<String, Object> option = new HashMap<>();            option.put("code", status.getCode());            option.put("name", status.name());            option.put("desc", status.getDesc());            options.add(option);        }        return options;    }}
```

**用 EnumUtils 重构后**：

```
@Servicepublic class OrderService {    // 定义合法的状态转换规则（保持不变）    private static final Map<OrderStatus, List<OrderStatus>> VALID_TRANSITIONS;    static {        VALID_TRANSITIONS = new HashMap<>();        VALID_TRANSITIONS.put(OrderStatus.PENDING, Arrays.asList(OrderStatus.PAID, OrderStatus.CANCELLED));        VALID_TRANSITIONS.put(OrderStatus.PAID, Arrays.asList(OrderStatus.SHIPPED, OrderStatus.REFUNDED));        // 其他状态转换规则...    }    // 转换订单状态（用EnumUtils简化核心逻辑）    public void changeStatus(Long orderId, String statusStr) {        // 字符串转枚举（一行代码替代循环）        OrderStatus newStatus = EnumUtils.getEnum(OrderStatus.class, statusStr);        if (newStatus == null) {            throw new IllegalArgumentException("无效的状态：" + statusStr);        }        // 查询当前状态（省略DAO操作）        OrderStatus currentStatus = getCurrentStatus(orderId);        // 校验转换是否合法（用EnumUtils.contains简化判断）        List<OrderStatus> allowed = VALID_TRANSITIONS.getOrDefault(currentStatus, Collections.emptyList());        if (!EnumUtils.contains(allowed, newStatus)) {            throw new IllegalStateException("不允许从" + currentStatus.getDesc() + "转换到" + newStatus.getDesc());        }        // 执行状态更新...    }    // 获取状态下拉框选项（用EnumUtils+Stream简化）    public List<Map<String, Object>> getStatusOptions() {        return EnumUtils.getEnumList(OrderStatus.class).stream()            .map(status -> {                Map<String, Object> option = new HashMap<>();                option.put("code", status.getCode());                option.put("name", status.name());                option.put("desc", status.getDesc());                return option;            })            .collect(Collectors.toList());    }}
```

重构后的代码变化：

1. 字符串转枚举用`EnumUtils.getEnum`替代手动循环，代码更简洁
2. 状态包含判断用`EnumUtils.contains`替代手动遍历，语义更清晰
3. 枚举列表处理用`EnumUtils.getEnumList`配合 Stream API，更符合函数式编程风格
4. 整体代码量减少 30%，可读性和可维护性显著提升

## 四、最佳实践与避坑指南

### 1. 依赖引入

EnumUtils 位于`commons-lang3`包中，需在项目中引入依赖：

```
<dependency>    <groupId>org.apache.commons</groupId>    <artifactId>commons-lang3</artifactId>    <version>3.12.0</version> <!-- 推荐最新稳定版 --></dependency>
```

注意使用`commons-lang3`（3.x 版本），而非旧版的`commons-lang`（2.x）。3.x 版本支持泛型，修复了多项 bug，且兼容 Java 8 及以上版本。

### 2. 性能优化

* **缓存枚举列表**：`EnumUtils.getEnumList`每次调用都会创建新列表，频繁使用时建议缓存：

  ```
  // 缓存枚举列表（单例模式）private static final List<OrderStatus> STATUS_LIST = EnumUtils.getEnumList(OrderStatus.class);
  ```
* **优先使用映射查询**：当枚举值较多（超过 20 个）时，`getEnum`的遍历操作可能有性能损耗，建议先创建映射缓存：

  ```
  // 缓存名称到枚举的映射private static final Map<String, OrderStatus> STATUS_MAP = EnumUtils.getEnumMap(OrderStatus.class);// 后续查询直接从映射获取OrderStatus status = STATUS_MAP.get(statusStr);
  ```

### 3. 常见误区

* **忽略枚举的线程安全性**：枚举本身是线程安全的，但`EnumUtils.getEnumList`返回的是普通 ArrayList，多线程修改时需加锁或使用同步集合。
* **混淆`getEnum`和`valueOf`**：`getEnum`在名称不匹配时返回 null，`valueOf`则抛出异常，需根据场景选择。简单的内部调用用`valueOf`更高效，处理外部不确定值时用`getEnum`更安全。
* 过度使用工具类：对于简单的枚举操作（如直接通过名称获取），无需刻意使用 EnumUtils，原生方法更高效。

## 结语：细节处的效率提升

枚举操作看似微不足道，却是日常开发中的高频场景。一个项目中如果充斥着重复的枚举转换、校验代码，不仅会增加维护成本，还会因手写逻辑的不一致性埋下隐患。

作为 commons-lang3 包的核心组件，它将枚举操作的通用逻辑封装成直观易用的静态方法，用一行代码替代多行模板代码，成为处理枚举的 "效率神器"。

看完之后发现是不是非常赞！

如果有什么不同的见解，那就在评论区发表一下吧！

还有更多好看的内容点击这里：