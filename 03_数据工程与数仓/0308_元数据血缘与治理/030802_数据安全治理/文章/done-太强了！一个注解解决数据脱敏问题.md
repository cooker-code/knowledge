> 已吸收至：[[03_数据工程与数仓/0308_元数据血缘与治理/030802_数据安全治理/030802_核心知识点/敏感字段加密脱敏与查询边界|敏感字段加密脱敏与查询边界]]
---
title: 太强了！一个注解解决数据脱敏问题
author: 芋道源码
date:
url: http://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247574017&idx=2&sn=f8bff5f2e9d19bc20b1ee98af9da6fd1&chksm=fa4bddb0cd3c54a6540f53f172036886d030200c40dc828d8e1585576093822ee7de7c2f3bd9&mpshare=1&scene=24&srcid=0614oArhfrb8PO0yoMrITwxx&sharer_sharetime=1686719344662&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

点击上方“芋道源码”，选择“[设为星标](http://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247486188&idx=3&sn=f160d91ea23e5113e6077c500a2e30c4&chksm=fa49755dcd3efc4bf4f566fbbbf74c191d0b79f2d3222fd211bc52d80b5ef127f52b1158ed71&scene=21#wechat_redirect)”

管她前浪，还是后浪？

能浪的浪，才是好浪！

每天 **10:33** 更新文章，每天掉亿点点头发...

源码精品专栏

* [原创 | Java 2021 超神之路，很肝~](http://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21#wechat_redirect)
* [中文详细注释的开源项目](http://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247486264&idx=1&sn=475ac3f1ef253a33daacf50477203c80&chksm=fa497489cd3efd9f7298f5da6aad0c443ae15f398436aff57cb2b734d6689e62ab43ae7857ac&scene=21#wechat_redirect)
* [RPC 框架 Dubbo 源码解析](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247484647&idx=1&sn=9eb7e47d06faca20d530c70eec3b8d5c&chksm=fa497b56cd3ef2408f807e66e0903a5d16fbed149ef7374021302901d6e0260ad717d903e8d4&scene=21#wechat_redirect)
* [网络应用框架 Netty 源码解析](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247485054&idx=2&sn=9f3b85f7b8454634da6c5c2ded9b4dba&chksm=fa4979cfcd3ef0d9d2dd92d8d1bd8f1553abc6e2095a5d743e0b2c2afe4955ea2bbbd7a4b79d&token=55862109&lang=zh_CN&scene=21#wechat_redirect)
* [消息中间件 RocketMQ 源码解析](http://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247486256&idx=1&sn=81daccd3fcd2953456c917630636fb26&chksm=fa497481cd3efd97d9239f5eab060e49dea9876a6046eadba0effb878d2fb51f3ba5733e4c0b&scene=21#wechat_redirect)
* [数据库中间件 Sharding-JDBC 和 MyCAT 源码解析](http://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247486257&idx=1&sn=4d3c9c675f8833157641a2e0b48e498c&chksm=fa497480cd3efd96fe17975b0b8b141e87fd0a62673e6a30b501460de80b3eb997056f09de08&scene=21#wechat_redirect)
* [作业调度中间件 Elastic-Job 源码解析](http://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247486258&idx=1&sn=ae5665ae9c3002b53f87cab44948a096&chksm=fa497483cd3efd950514da5a37160e7fd07f0a96f39265cf7ba3721985e5aadbdcbe7aafc34a&scene=21#wechat_redirect)
* [分布式事务中间件 TCC-Transaction 源码解析](http://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247486259&idx=1&sn=b023cf3dbf97e5da59db2f4ee632f5a6&chksm=fa497482cd3efd9402d71469f71863f71a6998b27e12ca2e00446b8178d79dcef0721d8e570a&scene=21#wechat_redirect)
* [Eureka 和 Hystrix 源码解析](http://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247486260&idx=1&sn=8f14c0c191d6f8df6eb34202f4ad9708&chksm=fa497485cd3efd93937143a648bc1b530bc7d1f6f8ad4bf2ec112ffe34dee80b474605c22db0&scene=21#wechat_redirect)
* [Java 并发源码](http://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247486261&idx=1&sn=bd69f26aadfc826f6313ffbb95e44ee5&chksm=fa497484cd3efd92352d6fb3d05ccbaebca2fafed6f18edbe5be70c99ba088db5c8a7a8080c1&scene=21#wechat_redirect)

[来源：juejin.cn/post/](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

[7225218026097852472](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

* [什么是数据脱敏](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
* [开胃菜](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)

+ [使用 Hutool 工具类实现数据掩码](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
+ [使用 Jackson 进行数据序列化脱敏](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)

* [注解实现数据脱敏](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)

+ [1、定义一个注解](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
+ [2、创建一个枚举类](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
+ [3、创建我们的自定义序列化类](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
+ [4、测试](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)

* [后记](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)

---

本文主要分享什么是数据脱敏，如何优雅的在项目中运用一个注解实现数据脱敏，为项目进行赋能。希望能给你们带来帮助。

## [什么是数据脱敏](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

数据脱敏是一种通过去除或替换敏感数据中的部分信息，以保护数据隐私和安全的技术。其主要目的是确保数据仍然可以在各种场景中使用，同时保护敏感信息，防止数据泄露和滥用。数据脱敏通常用于处理包含个人身份信息和其他敏感信息的数据集，如手机号、姓名、地址、银行卡、身份证号、车牌号等等。

在数据脱敏过程中，通常会采用不同的算法和技术，以根据不同的需求和场景对数据进行处理。例如，对于身份证号码，可以使用掩码算法（masking）将前几位数字保留，其他位用“X”或"\*"代替；对于姓名，可以使用伪造（pseudonymization）算法，将真实姓名替换成随机生成的假名。

下面我讲为大家带来数据脱敏掩码操作，让我们一起学起来吧。

> 基于 Spring Boot + MyBatis Plus + Vue & Element 实现的后台管理系统 + 用户小程序，支持 RBAC 动态权限、多租户、数据权限、工作流、三方登录、支付、短信、商城等功能
>
> * 项目地址：https://github.com/YunaiV/ruoyi-vue-pro
> * 视频教程：https://doc.iocoder.cn/video/

## [开胃菜](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

下面给大家介绍的是使用两种不同的工具类进行数据脱敏，而我们今天的主题使用一个注解解决数据脱敏问题的主要两个工具类。来跟着我学习吧。

### [使用 Hutool 工具类实现数据掩码](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

比喻说我们现在要对手机号进行数据脱敏，前三后四不掩码，其他全部用 \* 进行掩码

如下图代码所示，

我们定义了一个手机号：17677772345，需要进行数据脱敏。

调用的 Hutool 的信息脱敏工具类。

我们运行一下看看结果。一个简单的数据脱敏就实现了。

#### [Hutool 信息脱敏工具类](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

根据上面的一个 Demo，大家可以看到我使用了 Hutool 的信息脱敏工具类进行对手机号掩码脱敏。那么让我们一起看看 Hutool 信息脱敏的工具类吧。

官网文档：

https://hutool.cn/docs/#/core/工具类/信息脱敏工具-DesensitizedUtil

看一下官网的介绍，支持多种脱敏数据类型，满足我们大部分需求，如果需要自定义还提供了自定义的方法实现。

下面是里面定义号的脱敏规则，直接调用就可以实现简单的数据脱敏，这里给大家介绍是因为我们今天要给大家带来的注解实现数据脱敏核心就是利用我们的 Hutool 提供的工具类实现，支持自定义隐藏。

### [使用 Jackson 进行数据序列化脱敏](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

首先创建一个实体类，此实体类只有一个测试的手机号。

注解的讲解：

* @Data：lombok 的注解生成 get,set 等等方法。
* @JsonSerialize(using = TestJacksonSerialize.class)：该注解的作用就是可自定义序列化，可以用在注解上，方法上，字段上，类上，运行时生效等等，根据提供的序列化类里面的重写方法实现自定义序列化。可以看下下面的源码，有兴趣的朋友可以去了解一下，也能解决我们日常开发中很多场景。

```
@Data
public class TestDTO implements Serializable {
    /**
     * 手机号
     */
    @JsonSerialize(using = TestJacksonSerialize.class)
    private String phone;
}
```

然后创建一个 TestJacksonSerialize 类实现自定义序列化。

此类主要继承 JsonSerializer，因为我们这里需要序列化的类型是 String 泛型就选择 String。注意如果你使用此注解作用在类上的话，这里就是你要序列化的类。

重写序列化方法，里面的实现很简单就是调用我们的 Hutool 工具类进行手机号数据脱敏。

```
public class TestJacksonSerialize extends JsonSerializer<String> {

    @Override
    @SneakyThrows
    public void serialize(String str, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) {
        // 使用我们的hutool工具类进行手机号脱敏
        jsonGenerator.writeString(DesensitizedUtil.fixedPhone(String.valueOf(str)));
    }
}
```

让我们测试一下吧，因为此注解是运行时生效，我们定义一个接口来测试。

```
@RestController
@RequestMapping("/test")
public class TestApi {

    @GetMapping
    public TestDTO test(){
        TestDTO testDTO = new TestDTO();
        testDTO.setPhone("17677772345");
        return testDTO;
    }
}
```

可以看到测试成功，经过上面的两个工具类的介绍，联想一下我们怎么通过两个工具类定义一个自己的注解实现数据脱敏呢。

> 基于 Spring Cloud Alibaba + Gateway + Nacos + RocketMQ + Vue & Element 实现的后台管理系统 + 用户小程序，支持 RBAC 动态权限、多租户、数据权限、工作流、三方登录、支付、短信、商城等功能
>
> * 项目地址：https://github.com/YunaiV/yudao-cloud
> * 视频教程：https://doc.iocoder.cn/video/

## [注解实现数据脱敏](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

我们考虑一下，工具类现在有了，那么我们怎么去实现一个注解优雅的解决数据脱敏呢？

请看下文，让我带大家一起学习。

### [1、定义一个注解](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

定义一个 Desensitization 注解。

* @Retention(RetentionPolicy.RUNTIME)：运行时生效。
* @Target(ElementType.FIELD)：可用在字段上。
* @JacksonAnnotationsInside：此注解可以点进去看一下是一个元注解，主要是用户打包其他注解一起使用。
* @JsonSerialize：上面说到过，该注解的作用就是可自定义序列化，可以用在注解上，方法上，字段上，类上，运行时生效等等，根据提供的序列化类里面的重写方法实现自定义序列化。

```
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
@JacksonAnnotationsInside
@JsonSerialize(using = DesensitizationSerialize.class)
public @interface Desensitization {
    /**
     * 脱敏数据类型，只要在CUSTOMER的时候，startInclude和endExclude生效
     */
    DesensitizationTypeEnum type() default DesensitizationTypeEnum.CUSTOMER;

    /**
     * 开始位置（包含）
     */
    int startInclude() default 0;

    /**
     * 结束位置（不包含）
     */
    int endExclude() default 0;
}
```

可以看到此注解有三个值，一个是枚举类定义了我们的脱敏数据类型。一个开始位置，一个结束位置。

枚举类待会给大家讲解，如果选择了自定义类型，下面的开始位置，结束位置才生效。

开始结束位置是我们 Hutool 工具提供的自定义脱敏实现需要的参数。可以看此方法，需要提出一点的是此方法硬编码了掩码值。如果我们的场景需要其他掩码值的话实现也很简单，把 Hutool 的源码拷出来，代替他的硬编码，就可以实现。

### [2、创建一个枚举类](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

此枚举类是我们数据脱敏的类型，包括了大部分场景。以及可以满足我们日常开发咯。

```
public enum DesensitizationTypeEnum {
    //自定义
    CUSTOMER,
    //用户id
    USER_ID,
    //中文名
    CHINESE_NAME,
    //身份证号
    ID_CARD,
    //座机号
    FIXED_PHONE,
    //手机号
    MOBILE_PHONE,
    //地址
    ADDRESS,
    //电子邮件
    EMAIL,
    //密码
    PASSWORD,
    //中国大陆车牌，包含普通车辆、新能源车辆
    CAR_LICENSE,
    //银行卡
    BANK_CARD
}
```

### [3、创建我们的自定义序列化类](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

此类是我们数据脱敏的关键。主要是继承了我们的 JsonSerializer，实现了我的ContextualSerializer。重写了它俩的方法。

* @NoArgsConstructor：Lombok 无参构造生成。
* @AllArgsConstructor：Lombok 有参生成。
* ContextualSerializer：这个类是序列化上下文类，主要是解决我们这个地方获取字段的一些信息，可以看一下源码，他的实现类有很多，Jackson 提供的 @JsonFormat 注解也是实现此类，获取字段的一些信息进行序列化的。有兴趣的朋友可以看一下，多看源码，才能学到 Jackson 的实现方法，才能有今天我们的实现。

两个重写的方法解读：

* serialize：重写，实现我们的序列化自定义。
* createContextual：序列化上下文方法重写，获取我们的字段一些信息进行判断，然后返回实例。具体代码可以看下面代码，都有注释噢。

```
@NoArgsConstructor
@AllArgsConstructor
public class DesensitizationSerialize extends JsonSerializer<String> implements ContextualSerializer {
    private DesensitizationTypeEnum type;

    private Integer startInclude;

    private Integer endExclude;
    @Override
    public void serialize(String str, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
        switch (type) {
            // 自定义类型脱敏
            case CUSTOMER:
                jsonGenerator.writeString(CharSequenceUtil.hide(str,startInclude,endExclude));
                break;
            // userId脱敏
            case USER_ID:
                jsonGenerator.writeString(String.valueOf(DesensitizedUtil.userId()));
                break;
            // 中文姓名脱敏
            case CHINESE_NAME:
                jsonGenerator.writeString(DesensitizedUtil.chineseName(String.valueOf(str)));
                break;
            // 身份证脱敏
            case ID_CARD:
                jsonGenerator.writeString(DesensitizedUtil.idCardNum(String.valueOf(str), 1, 2));
                break;
            // 固定电话脱敏
            case FIXED_PHONE:
                jsonGenerator.writeString(DesensitizedUtil.fixedPhone(String.valueOf(str)));
                break;
            // 手机号脱敏
            case MOBILE_PHONE:
                jsonGenerator.writeString(DesensitizedUtil.mobilePhone(String.valueOf(str)));
                break;
            // 地址脱敏
            case ADDRESS:
                jsonGenerator.writeString(DesensitizedUtil.address(String.valueOf(str), 8));
                break;
            // 邮箱脱敏
            case EMAIL:
                jsonGenerator.writeString(DesensitizedUtil.email(String.valueOf(str)));
                break;
            // 密码脱敏
            case PASSWORD:
                jsonGenerator.writeString(DesensitizedUtil.password(String.valueOf(str)));
                break;
            // 中国车牌脱敏
            case CAR_LICENSE:
                jsonGenerator.writeString(DesensitizedUtil.carLicense(String.valueOf(str)));
                break;
            // 银行卡脱敏
            case BANK_CARD:
                jsonGenerator.writeString(DesensitizedUtil.bankCard(String.valueOf(str)));
                break;
            default:
        }

    }

    @Override
    public JsonSerializer<?> createContextual(SerializerProvider serializerProvider, BeanProperty beanProperty) throws JsonMappingException {
        if (beanProperty != null) {
            // 判断数据类型是否为String类型
            if (Objects.equals(beanProperty.getType().getRawClass(), String.class)) {
                // 获取定义的注解
                Desensitization desensitization = beanProperty.getAnnotation(Desensitization.class);
                // 为null
                if (desensitization == null) {
                    desensitization = beanProperty.getContextAnnotation(Desensitization.class);
                }
                // 不为null
                if (desensitization != null) {
                    // 创建定义的序列化类的实例并且返回，入参为注解定义的type,开始位置，结束位置。
                    return new DesensitizationSerialize(desensitization.type(), desensitization.startInclude(),
                            desensitization.endExclude());
                }
            }

            return serializerProvider.findValueSerializer(beanProperty.getType(), beanProperty);
        }
        return serializerProvider.findNullValueSerializer(null);
    }
}
```

### [4、测试](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

创建一个测试注解的 DTO，此测试如下。

```
@Data
public class TestAnnotationDTO implements Serializable {
    /**
     * 自定义
     */
    @Desensitization(type = DesensitizationTypeEnum.CUSTOMER,startInclude = 5,endExclude = 10)
    private String custom;
    /**
     * 手机号
     */
    @Desensitization(type = DesensitizationTypeEnum.MOBILE_PHONE)
    private String phone;
    /**
     * 邮箱
     */
    @Desensitization(type = DesensitizationTypeEnum.EMAIL)
    private String email;
    /**
     * 身份证
     */
    @Desensitization(type = DesensitizationTypeEnum.ID_CARD)
    private String idCard;
}
```

新增测试接口：

```
@GetMapping("/test-annotation")
public TestAnnotationDTO testAnnotation(){
    TestAnnotationDTO testAnnotationDTO = new TestAnnotationDTO();
    testAnnotationDTO.setPhone("17677772345");
    testAnnotationDTO.setCustom("111111111111111111");
    testAnnotationDTO.setEmail("1433926101@qq.com");
    testAnnotationDTO.setIdCard("4444199810015555");
    return testAnnotationDTO;
}
```

测试一下看看效果。如下图所示，完美！

#### [项目 pom 文件](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

```
<?xml version="1.0" encoding="utf-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.10</version>
        <relativePath/>
        <!-- lookup parent from repository -->
    </parent>
    <groupId>com.jiaqing</groupId>
    <artifactId>tool-desensitization</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>tool-desensitization</name>
    <description>数据脱敏</description>
    <properties>
        <java.version>1.8</java.version>
        <hutool.version>5.8.5</hutool.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-core</artifactId>
            <version>${hutool.version}</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!--json模块-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-json</artifactId>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

## [后记](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

今天给大家带来的是如何实现一个注解进行数据脱敏。

* 如何使用 hutool 工具类进行数据脱敏。
* 如何使用 @JsonSerialize 注解实现自定义序列化。
* 如何使用 hutool 工具+Jackson 实现自己的脱敏注解。

---

---

欢迎加入我的知识星球，一起探讨架构，交流源码。加入方式，**长按下方二维码噢**：

已在知识星球更新源码解析如下：

最近更新《芋道 SpringBoot 2.X 入门》系列，已经 101 余篇，覆盖了 MyBatis、Redis、MongoDB、ES、分库分表、读写分离、SpringMVC、Webflux、权限、WebSocket、Dubbo、RabbitMQ、RocketMQ、Kafka、性能测试等等内容。

提供近 3W 行代码的 SpringBoot 示例，以及超 4W 行代码的电商微服务项目。

获取方式：点“**在看**”，关注公众号并回复 **666** 领取，更多内容陆续奉上。

```
文章有帮助的话，在看，转发吧。

谢谢支持哟 (*^__^*）
```