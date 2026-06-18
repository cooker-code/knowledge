> 已吸收至：[[07_工程与架构/0701_后端架构/070105_SpringBoot/070105_核心知识点/SpringBoot接口集成与运行治理边界|SpringBoot接口集成与运行治理边界]]
---
title: 和 if else说再见，SpringBoot 这样做参数校验才足够优雅！
author: 互联网架构师
date:
url: http://mp.weixin.qq.com/s?__biz=MzI2MTIzMzY3Mw==&mid=2247541045&idx=2&sn=1837bd45cdb677561ac6d35457ac9c4a&chksm=ea5fec53dd2865453eda9c9d2734101a88fd216367b1bd21fefc04933e860cd4b82ea92917e9&mpshare=1&scene=24&srcid=0614YJdmLIXIQhUHDHXbyDGo&sharer_sharetime=1686701644016&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

**点击**关注**公众号：**互联网架构师，后台回复[**2T**](https://mp.weixin.qq.com/s?__biz=MzI2MTIzMzY3Mw==&mid=2247487508&idx=1&sn=78cf235aa9ba5f988c6922ca98f8bfd6&chksm=ea5cdd72dd2b54647cf55b4a73dcafa69fc7228205ad39ecc98fe57b39cdecb21c238c6d6cb6&scene=21#wechat_redirect)**获取**[**2TB**](https://mp.weixin.qq.com/s?__biz=MzI2MTIzMzY3Mw==&mid=2247487508&idx=1&sn=78cf235aa9ba5f988c6922ca98f8bfd6&chksm=ea5cdd72dd2b54647cf55b4a73dcafa69fc7228205ad39ecc98fe57b39cdecb21c238c6d6cb6&scene=21#wechat_redirect)**学习资源！**

上一篇：[Alibaba开源内网高并发编程手册.pdf](http://mp.weixin.qq.com/s?__biz=MzI2MTIzMzY3Mw==&mid=2247523205&idx=1&sn=89b261f829ce6c3487ce8b2ccf3ed06b&chksm=ea5f56e3dd28dff5238989d3e07d775d34c04a4e80a166059e92e84379274819203d72d1dc1d&scene=21#wechat_redirect)

## 一、概述

当我们想提供可靠的 API 接口，对参数的校验，以保证最终数据入库的正确性，是 必不可少 的活。比如下图就是 我们一个项目里 新增一个菜单校验 参数的函数，写了一大堆的 if else 进行校验，非常的不优雅，比起枯燥的CRUD来说，参数校验更是枯燥。

这只是一个创建菜单的校验，只需要判断菜单，菜单url 以及菜单的父类id是否为空，上级菜单是否挂载正确，这样已经消耗掉了30，40行代码了，更不要说，管理后台创建商品这种参数贼多的接口。估计要写几百行校验代码了。

```
/**
  * 验证参数是否正确
  */
private void verifyForm(SysMenuEntity menu){
    if(StringUtils.isBlank(menu.getName())){
        throw new RRException("菜单名称不能为空");
    }

    if(menu.getParentId() == null){
        throw new RRException("上级菜单不能为空");
    }

    //菜单
    if(menu.getType() == Constant.MenuType.MENU.getValue()){
        if(StringUtils.isBlank(menu.getUrl())){
            throw new RRException("菜单URL不能为空");
        }
    }

    //上级菜单类型
    int parentType = Constant.MenuType.CATALOG.getValue();
    if(menu.getParentId() != 0){
        SysMenuEntity parentMenu = sysMenuService.getById(menu.getParentId());
        parentType = parentMenu.getType();
    }

    //目录、菜单
    if(menu.getType() == Constant.MenuType.CATALOG.getValue() ||
       menu.getType() == Constant.MenuType.MENU.getValue()){
        if(parentType != Constant.MenuType.CATALOG.getValue()){
            throw new RRException("上级菜单只能为目录类型");
        }
        return ;
    }

    //按钮
    if(menu.getType() == Constant.MenuType.BUTTON.getValue()){
        if(parentType != Constant.MenuType.MENU.getValue()){
            throw new RRException("上级菜单只能为菜单类型");
        }
        return ;
    }
}
```

可能小伙伴会说不加参数校验行不行？或者把参数校验放到前端不就行了？那你可是想的太简单了，世界比我们想象中的不安全，可能有“黑客”会绕过浏览器，直接使用 HTTP 工具，模拟请求向后端 API 接口传入违法的参数，以达到它们 “不可告人” 的目的。比如 sql 注入攻击，我相信，很多时候并不是我们不想添加，而是没有统一方便的方式，让我们快速的添加实现参数校验的功能。

世界上大多数碰到的困难，大多已经有了解决方案，特别是开发生态非常完整的java来说，早在 Java 2009 年就提出了 Bean Validation 规范，并且已经历经 JSR303、JSR349、JSR380 三次标准的置顶，发展到了 2.0 。

有细心的小伙伴可能发现 Jakarta Bean Validation 3.0 里，这里3.0变化并不大，只是更改了一下包名 和命名空间而已。实际上还是对 Bean Validation 2.0 的实现。

Bean Validation 和我们很久以前学习过的 JPA 一样，只提供规范，不提供具体的实现，目前实现 Bean Validation 规范的数据校验框架，主要有：

* Hibernate Validator
* Apache BVal

可能有小伙伴就要说 Hibernate 不就是个老掉牙的ORM框架吗？现在不都没人用了吗？其实不然 Hibernate 可是打着 Everything data 口号的，它还提供了 Hibernate Search、Hibernate OGM 等等解决方案的。

Hibernate 只是在国内的用的少了，国内主要是还是用 mybatis 这种 半orm框架的多。我们可以通过 google 的 trends 来看一下：

在中国的 mybatis， jpa， Hibernate 搜索热度：

在全球的 mybatis， jpa， Hibernate 搜索热度：

由于国内的开发环境可以说 99.99% 的开发者肯定都在用 spring，且正好 Spring Validation 提供了对 Bean Validation 的内置封装支持，可以使用 @Validated 注解，实现声明式校验，而无需直接调用 Bean Validation 提供的 API 方法。

而在实现原理上，也是基于 Spring AOP 拦截，最终还是调用不同的 Bean Validation 的实现框架。例如说，Hibernate Validator 。实现校验相关的操作这一点，类似 Spring Transaction 事务，通过 @Transactional 注解，实现声明式事务。下面，让我们开始学习如何在 Spring Boot 中，实现参数校验。

## 二、注解

在开始入门之前，我们先了解下本文可能会涉及到的注解。javax.validation.constraints 包下，定义了一系列的约束( constraint )注解。共 22个，如下：

大致可以分为以下几类：

### 2.1 空和非空检查

* `@NotBlank` ：只能用于字符串不为 null ，并且字符串 `#trim()` 以后 length 要大于 0 。
* `@NotEmpty` ：集合对象的元素不为 0 ，即集合不为空，也可以用于字符串不为 null 。
* `@NotNull` ：不能为 null 。
* `@Null` ：必须为 null 。

### 2.2 数值检查

* `@DecimalMax(value)` ：被注释的元素必须是一个数字，其值必须小于等于指定的最大值。
* `@DecimalMin(value)` ：被注释的元素必须是一个数字，其值必须大于等于指定的最小值。
* `@Digits(integer, fraction)` ：被注释的元素必须是一个数字，其值必须在可接受的范围内。
* `@Positive` ：判断正数。
* `@PositiveOrZero`：判断正数或 0 。
* `@Max(value)` ：该字段的值只能小于或等于该值。
* `@Min(value)` ：该字段的值只能大于或等于该值。-`@Negative` ：判断负数。
* `@NegativeOrZero` ：判断负数或 0 。

### 2.3 Boolean 值检查

* `@AssertFalse` ：被注释的元素必须为 true 。
* `@AssertTrue` ：被注释的元素必须为 false 。

### 2.4 长度检查

* `@Size(max, min)` ：检查该字段的 size 是否在 min 和 max 之间，可以是字符串、数组、集合、Map 等。

### 2.5 日期检查

* `@Future` ：被注释的元素必须是一个将来的日期。
* `@FutureOrPresent` ：判断日期是否是将来或现在日期。
* `@Past` ：检查该字段的日期是在过去。
* `@PastOrPresent` ：判断日期是否是过去或现在日期。

### 2.6 其它检查

* `@Email` ：被注释的元素必须是电子邮箱地址。
* `@Pattern(value)` ：被注释的元素必须符合指定的正则表达式。

### 2.7 Hibernate Validator 附加的约束注解

`org.hibernate.validator.constraints` 包下，定义了一系列的约束( constraint )注解。如下：

* `@Range(min=, max=)` ：被注释的元素必须在合适的范围内。
* `@Length(min=, max=)` ：被注释的字符串的大小必须在指定的范围内。
* `@URL(protocol=,host=,port=,regexp=,flags=)` ：被注释的字符串必须是一个有效的 URL 。
* `@SafeHtml` ：判断提交的 HTML 是否安全。例如说，不能包含 javascript 脚本等等。

### 2.8 @Valid 和 @Validated

`@Valid` 注解，是 Bean Validation 所定义，可以添加在普通方法、构造方法、方法参数、方法返回、成员变量上，表示它们需要进行约束校验。

`@Validated` 注解，是 Spring Validation 锁定义，可以添加在类、方法参数、普通方法上，表示它们需要进行约束校验。同时，@Validated 有 value 属性，支持分组校验。属性如下：

```
@Target({ElementType.TYPE, ElementType.METHOD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Validated {

    /**
  * Specify one or more validation groups to apply to the validation step
  * kicked off by this annotation.
  * <p>JSR-303 defines validation groups as custom annotations which an application declares
  * for the sole purpose of using them as type-safe group arguments, as implemented in
  * {@link org.springframework.validation.beanvalidation.SpringValidatorAdapter}.
  * <p>Other {@link org.springframework.validation.SmartValidator} implementations may
  * support class arguments in other ways as well.
  */
    Class<?>[] value() default {};

}
```

对于初学的胖友来说，很容易搞混 @Valid（`javax.validation`包下） 和 @Validated （`org.springframework.validation.annotation`包下）注解。两者大致有以下的区别：

| 名称 | spring Validation是否实现了声明式校验 | 是否支持嵌套校验 | 是否支持分组校验 |
| --- | --- | --- | --- |
| @Validated | 是 | 否 | 是 |
| @Valid | 否 | 是 | 否 |

@Valid 有嵌套对象校验功能 例如说：如果不在 `User.profile` 属性上，添加 @Valid 注解，就会导致 `UserProfile.nickname` 属性，不会进行校验。

```
// User.java
public class User {

    private String id;

    @Valid
    private UserProfile profile;

}

// UserProfile.java
public class UserProfile {

    @NotBlank
    private String nickname;

}
```

总的来说，绝大多数场景下，我们使用 @Validated 注解即可。而在有嵌套校验的场景，我们使用 @Valid 注解添加到成员属性上。

## 三、快速入门

### 3.1 引入依赖

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>


    <groupId>com.ratel</groupId>
    <artifactId>java-validation</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>java-validation</name>
    <description>java validation action</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>


    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>


        <!--在一些高版本springboot中默认并不会引入这个依赖，需要手动引入-->
        <!--        <dependency>
            <groupId>org.hibernate.validator</groupId>
            <artifactId>hibernate-validator</artifactId>
            <scope>compile</scope>
        </dependency>-->

        <!-- 保证 Spring AOP 相关的依赖包 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
        </dependency>


        <!--lombok相关 方便开发-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <scope>provided</scope>
        </dependency>

        <!--knife4j接口文档 方便待会进行接口测试-->
        <dependency>
            <groupId>com.github.xiaoymin</groupId>
            <artifactId>knife4j-spring-boot-starter</artifactId>
            <version>3.0.3</version>
        </dependency>

    </dependencies>


    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

在 Spring Boot 体系中，也提供了 `spring-boot-starter-validation` 依赖。在这里，我们并没有引入。为什么呢？因为 `spring-boot-starter-web` 已经引入了 `spring-boot-starter-validation` ，而 `spring-boot-starter-validation` 中也引入了 `hibernate-validator` 依赖，所以无需重复引入。三者的依赖引入关系可见下图

### 3.2 创建基本的类

UserAddDTO 实体类：

```
package com.ratel.validation.entity;

import lombok.Data;
import org.hibernate.validator.constraints.Length;

import javax.validation.constraints.NotEmpty;
import javax.validation.constraints.Pattern;

/**
 * @Description
 * @Date 2023/04/07
 * @Version 1.0
 */
@Data
public class UserAddDTO {

    /**
     * 账号
     */
    @NotEmpty(message = "登录账号不能为空")
    @Length(min = 5, max = 16, message = "账号长度为 5-16 位")
    @Pattern(regexp = "^[A-Za-z0-9]+$", message = "账号格式为数字以及字母")
    private String username;
    /**
     * 密码
     */
    @NotEmpty(message = "密码不能为空")
    @Length(min = 4, max = 16, message = "密码长度为 4-16 位")
    private String password;
}
```

UserController 用来写接口，在类上，添加 @Validated 注解，表示 UserController 是所有接口都需要进行参数校验。

```
package com.ratel.validation.cotroller;

import com.ratel.validation.entity.UserAddDTO;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;

import javax.validation.Valid;
import javax.validation.constraints.Min;

@RestController
@RequestMapping("/users")
@Validated
public class UserController {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @GetMapping("/get")
    public UserAddDTO get(@RequestParam("id") @Min(value = 1L, message = "编号必须大于 0") Integer id) {
        logger.info("[get][id: {}]", id);
        UserAddDTO userAddDTO = new UserAddDTO();
        userAddDTO.setUsername("张三");
        userAddDTO.setPassword("123456");
        return userAddDTO;
    }

    @PostMapping("/add")
    public void add(@Valid @RequestBody UserAddDTO addDTO) {
        logger.info("[add][addDTO: {}]", addDTO);
    }

}
```

### 3.3 启动程序，进行测试

启动程序，然后再浏览器里我们就可以进行输入: swagger访问地址：`http://localhost:8080/doc.html#/home` 打开swagger文档 就可以进行测试了：

首先我们访问 `http://localhost:8080/users/get?id=-1` 进行测试，查看返回结果，果然对我们的 id 进行校验。

接下来我们访问 `http://localhost:8080/users/add` 进行新增用户的校验：请求体我们写成：

```
{
  "password": "233",
  "username": "33"
}
```

然后返回结果的如下：

```
{
  "timestamp": "2023-04-09T13:33:58.864+0000",
  "status": 400,
  "error": "Bad Request",
  "errors": [
    {
      "codes": [
        "Length.userAddDTO.password",
        "Length.password",
        "Length.java.lang.String",
        "Length"
      ],
      "arguments": [
        {
          "codes": [
            "userAddDTO.password",
            "password"
          ],
          "arguments": null,
          "defaultMessage": "password",
          "code": "password"
        },
        16,
        4
      ],
      "defaultMessage": "密码长度为 4-16 位",
      "objectName": "userAddDTO",
      "field": "password",
      "rejectedValue": "233",
      "bindingFailure": false,
      "code": "Length"
    },
    {
      "codes": [
        "Length.userAddDTO.username",
        "Length.username",
        "Length.java.lang.String",
        "Length"
      ],
      "arguments": [
        {
          "codes": [
            "userAddDTO.username",
            "username"
          ],
          "arguments": null,
          "defaultMessage": "username",
          "code": "username"
        },
        16,
        5
      ],
      "defaultMessage": "账号长度为 5-16 位",
      "objectName": "userAddDTO",
      "field": "username",
      "rejectedValue": "33",
      "bindingFailure": false,
      "code": "Length"
    }
  ],
  "message": "Validation failed for object='userAddDTO'. Error count: 2",
  "path": "/users/add"
}
```

返回结果的json串中的 errors 字段，参数错误明细数组。每一个数组元素，对应一个参数错误明细。这里，username 违背了 账号长度为 5-16 位规定。password违反了 密码长度为 4-16 位的规定。

返回结果示意图：

### 3.3 一些疑问

在这里 细心的小伙伴可能会有几个疑问：

#### 3.3.1 疑问一

`#get(id)` 方法上，我们并没有给 id 添加 @Valid 注解，而 `#add(addDTO)` 方法上，我们给 addDTO 添加 @Valid 注解。这个差异，是为什么呢？

因为 `UserController` 使用了 @Validated 注解，那么 Spring Validation 就会使用 AOP 进行切面，进行参数校验。而该切面的拦截器，使用的是 `MethodValidationInterceptor` 。

* 对于 `#get(id)` 方法，需要校验的参数 id ，是平铺开的，所以无需添加 @Valid 注解。
* 对于 `#add(addDTO)` 方法，需要校验的参数 addDTO ，实际相当于嵌套校验，要校验的参数的都在 addDTO 里面，所以需要添加 @Valid （其实实测加@Validated也行，暂时不知道为啥 为了好区分就先用 @Valid 吧 ）注解。

#### 3.3.2 疑问二

`#get(id)` 方法的返回的结果是 `status = 500` ，而`#add(addDTO)` 方法的返回的结果是 `status = 400` 。

* 对于 `#get(id)` 方法，在 `MethodValidationInterceptor` 拦截器中，校验到参数不正确，会抛出 `ConstraintViolationException` 异常。
* 对于 `#add(addDTO)` 方法，因为 addDTO 是个 POJO 对象，所以会走 SpringMVC 的 DataBinder 机制，它会调用 `DataBinder#validate(Object... validationHints)` 方法，进行校验。在校验不通过时，会抛出 `BindException` 。

在 SpringMVC 中，默认使用 `DefaultHandlerExceptionResolver` 处理异常。

* 对于 `BindException` 异常，处理成 400 的状态码。
* 对于 `ConstraintViolationException` 异常，没有特殊处理，所以处理成 500 的状态码。

这里，我们在抛个问题，如果 `#add(addDTO)` 方法，如果参数正确，在走完 DataBinder 中的参数校验后，会不会在走一遍 `MethodValidationInterceptor` 的拦截器呢？思考 100 毫秒…

答案是会。这样，就会导致浪费。所以 Controller 类里，如果 只有 类似的 `#add(addDTO)` 方法的 嵌套校验，那么我可以不在 Controller 类上添加 @Validated 注解。从而实现，仅使用 DataBinder 中来做参数校验。

#### 3.3.3 返回提示很不友好，太长了

第三点，无论是 `#get(id)` 方法，还是 `#add(addDTO)` 方法，它们的返回提示都非常不友好，那么该怎么办呢？我们将在 第四章节通过 处理校验异常 进行处理。

## 四、处理校验异常

### 4.1 校验不通过的枚举类

```
package com.ratel.validation.enums;

/**
 * 业务异常枚举
 */
public enum ServiceExceptionEnum {

    // ========== 系统级别 ==========
    SUCCESS(0, "成功"),
    SYS_ERROR(2001001000, "服务端发生异常"),
    MISSING_REQUEST_PARAM_ERROR(2001001001, "参数缺失"),
    INVALID_REQUEST_PARAM_ERROR(2001001002, "请求参数不合法"),

    // ========== 用户模块 ==========
    USER_NOT_FOUND(1001002000, "用户不存在"),

    // ========== 订单模块 ==========

    // ========== 商品模块 ==========
    ;

    /**
     * 错误码
     */
    private final int code;
    /**
     * 错误提示
     */
    private final String message;

    ServiceExceptionEnum(int code, String message) {
        this.code = code;
        this.message = message;
    }

    public int getCode() {
        return code;
    }

    public String getMessage() {
        return message;
    }

}
```

### 4.2 统一返回结果实体类

```
package com.ratel.validation.common;

import com.fasterxml.jackson.annotation.JsonIgnore;
import org.springframework.util.Assert;

import java.io.Serializable;

/**
 * 通用返回结果
 *
 * @param <T> 结果泛型
 */
public class CommonResult<T> implements Serializable {

    public static Integer CODE_SUCCESS = 0;

    /**
     * 错误码
     */
    private Integer code;
    /**
     * 错误提示
     */
    private String message;
    /**
     * 返回数据
     */
    private T data;

    /**
     * 将传入的 result 对象，转换成另外一个泛型结果的对象
     *
     * 因为 A 方法返回的 CommonResult 对象，不满足调用其的 B 方法的返回，所以需要进行转换。
     *
     * @param result 传入的 result 对象
     * @param <T> 返回的泛型
     * @return 新的 CommonResult 对象
     */
    public static <T> CommonResult<T> error(CommonResult<?> result) {
        return error(result.getCode(), result.getMessage());
    }

    public static <T> CommonResult<T> error(Integer code, String message) {
        Assert.isTrue(!CODE_SUCCESS.equals(code), "code 必须是错误的！");
        CommonResult<T> result = new CommonResult<>();
        result.code = code;
        result.message = message;
        return result;
    }

    public static <T> CommonResult<T> success(T data) {
        CommonResult<T> result = new CommonResult<>();
        result.code = CODE_SUCCESS;
        result.data = data;
        result.message = "";
        return result;
    }

    public Integer getCode() {
        return code;
    }

    public void setCode(Integer code) {
        this.code = code;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }

    @JsonIgnore
    public boolean isSuccess() {
        return CODE_SUCCESS.equals(code);
    }

    @JsonIgnore
    public boolean isError() {
        return !isSuccess();
    }

    @Override
    public String toString() {
        return "CommonResult{" +
            "code=" + code +
            ", message='" + message + '\'' +
            ", data=" + data +
            '}';
    }

}
```

### 4.3 增加全局异常处理类 GlobalExceptionHandler

```
package com.ratel.validation.exception;


import com.ratel.validation.common.CommonResult;
import com.ratel.validation.enums.ServiceExceptionEnum;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.validation.BindException;
import org.springframework.validation.ObjectError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.MissingServletRequestParameterException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.HttpServletRequest;
import javax.validation.ConstraintViolation;
import javax.validation.ConstraintViolationException;

@ControllerAdvice(basePackages = "com.ratel.validation.cotroller")
public class GlobalExceptionHandler {

    private Logger logger = LoggerFactory.getLogger(getClass());

    /**
     * 处理 MissingServletRequestParameterException 异常
     *
     * SpringMVC 参数不正确
     */
    @ResponseBody
    @ExceptionHandler(value = MissingServletRequestParameterException.class)
    public CommonResult missingServletRequestParameterExceptionHandler(HttpServletRequest req, MissingServletRequestParameterException ex) {
        logger.error("[missingServletRequestParameterExceptionHandler]", ex);
        // 包装 CommonResult 结果
        return CommonResult.error(ServiceExceptionEnum.MISSING_REQUEST_PARAM_ERROR.getCode(),
                                  ServiceExceptionEnum.MISSING_REQUEST_PARAM_ERROR.getMessage());
    }

    @ResponseBody
    @ExceptionHandler(value = ConstraintViolationException.class)
    public CommonResult constraintViolationExceptionHandler(HttpServletRequest req, ConstraintViolationException ex) {
        logger.error("[constraintViolationExceptionHandler]", ex);
        // 拼接错误
        StringBuilder detailMessage = new StringBuilder();
        for (ConstraintViolation<?> constraintViolation : ex.getConstraintViolations()) {
            // 使用 ; 分隔多个错误
            if (detailMessage.length() > 0) {
                detailMessage.append(";");
            }
            // 拼接内容到其中
            detailMessage.append(constraintViolation.getMessage());
        }
        // 包装 CommonResult 结果
        return CommonResult.error(ServiceExceptionEnum.INVALID_REQUEST_PARAM_ERROR.getCode(),
                                  ServiceExceptionEnum.INVALID_REQUEST_PARAM_ERROR.getMessage() + ":" + detailMessage.toString());
    }

    @ResponseBody
    @ExceptionHandler(value = BindException.class)
    public CommonResult bindExceptionHandler(HttpServletRequest req, BindException ex) {
        logger.info("========进入了 bindException======");
        logger.error("[bindExceptionHandler]", ex);
        // 拼接错误
        StringBuilder detailMessage = new StringBuilder();
        for (ObjectError objectError : ex.getAllErrors()) {
            // 使用 ; 分隔多个错误
            if (detailMessage.length() > 0) {
                detailMessage.append(";");
            }
            // 拼接内容到其中
            detailMessage.append(objectError.getDefaultMessage());
        }
        // 包装 CommonResult 结果
        return CommonResult.error(ServiceExceptionEnum.INVALID_REQUEST_PARAM_ERROR.getCode(),
                                  ServiceExceptionEnum.INVALID_REQUEST_PARAM_ERROR.getMessage() + ":" + detailMessage.toString());
    }

    @ResponseBody
    @ExceptionHandler(value = MethodArgumentNotValidException.class)
    public CommonResult MethodArgumentNotValidExceptionHandler(HttpServletRequest req, MethodArgumentNotValidException ex) {
        logger.info("-----------------进入了 MethodArgumentNotValidException-----------------");
        logger.error("[MethodArgumentNotValidException]", ex);
        // 拼接错误
        StringBuilder detailMessage = new StringBuilder();
        for (ObjectError objectError : ex.getBindingResult().getAllErrors()) {
            // 使用 ; 分隔多个错误
            if (detailMessage.length() > 0) {
                detailMessage.append(";");
            }
            // 拼接内容到其中
            detailMessage.append(objectError.getDefaultMessage());
        }
        // 包装 CommonResult 结果
        return CommonResult.error(ServiceExceptionEnum.INVALID_REQUEST_PARAM_ERROR.getCode(),
                                  ServiceExceptionEnum.INVALID_REQUEST_PARAM_ERROR.getMessage() + ":" + detailMessage.toString());
    }


    /**
     * 处理其它 Exception 异常
     * @param req
     * @param e
     * @return
     */
    @ResponseBody
    @ExceptionHandler(value = Exception.class)
    public CommonResult exceptionHandler(HttpServletRequest req, Exception e) {
        // 记录异常日志
        logger.error("[exceptionHandler]", e);
        // 返回 ERROR CommonResult
        return CommonResult.error(ServiceExceptionEnum.SYS_ERROR.getCode(),
                                  ServiceExceptionEnum.SYS_ERROR.getMessage());
    }
}
```

### 4.4 测试

访问：`http://localhost:8080/users/add` 可以看到异常结果已经被拼接成一个字符串，相比之前清新 易懂了不少。

> ###

最后，关注公众号互联网架构师，在后台回复：2T，可以获取我整理的 Java 系列面试题和答案，非常齐全。

### 正文结束 推荐阅读 ↓↓↓ 1.[JetBrains 如何看待自己的软件在中国被频繁破解?](http://mp.weixin.qq.com/s?__biz=MzI2MTIzMzY3Mw==&mid=2247537122&idx=1&sn=d7e44519a307465a94cd9c221388d60a&chksm=ea5f9c84dd281592913c9215cf9371141633753cff2c6deefad2c8c34fb73926278129dadfcf&scene=21#wechat_redirect) 2.[Alibaba开源内网高并发编程手册.pdf](http://mp.weixin.qq.com/s?__biz=MzI2MTIzMzY3Mw==&mid=2247526408&idx=2&sn=2477c56ea703763e74e502a48960dd62&chksm=ea5fa56edd282c78817792c420b39ad8384c2de55d2b48eb236af85424f0782372a1ec556796&scene=21#wechat_redirect) 3.[程序员一般可以从什么平台接私活？](http://mp.weixin.qq.com/s?__biz=MzI2MTIzMzY3Mw==&mid=2247488928&idx=1&sn=4c56dd675e1b32a73b698df3d5e8609f&chksm=ea5cd8c6dd2b51d05e6d6d715418241f471ccb3002719263b2d0e092763f68f3691249970a08&scene=21#wechat_redirect) 4.[40岁，刚被裁，想说点啥。](http://mp.weixin.qq.com/s?__biz=MzI2MTIzMzY3Mw==&mid=2247536860&idx=1&sn=b77e9a1953eefa90bfba0da7b73146d1&chksm=ea5f9dbadd2814ac870fab4e4c28f4e63ee07c19e8eaaa690501a4a023fd76083e750cd41ee7&scene=21#wechat_redirect) 5.[为什么国内 996 干不过国外的 955呢？](http://mp.weixin.qq.com/s?__biz=MzI2MTIzMzY3Mw==&mid=2247513283&idx=1&sn=3b3d8e893a1241cf5af2150370ae9d71&chksm=ea5f79a5dd28f0b3a7603b1fbdb306f77ac64223bd05c4b7f0dc0469f77a68359810c5de2a01&scene=21#wechat_redirect) 6.[中国的铁路订票系统在世界上属于什么水平？](http://mp.weixin.qq.com/s?__biz=MzI2MTIzMzY3Mw==&mid=2247528236&idx=1&sn=c675d5d3aede21b4b7ebbba21eadf96c&chksm=ea5fa24add282b5cfb13757cd534ec600fb910f9dcb7f3ee3dea5e7929e0fbbd22381625ff20&scene=21#wechat_redirect)                         7.[15张图看懂瞎忙和高效的区别！](http://mp.weixin.qq.com/s?__biz=MzI2MTIzMzY3Mw==&mid=2247488564&idx=1&sn=256862239b12a1313919ee3ea6ff0d1c&chksm=ea5cd952dd2b50443feec88eb893233d2f6cf33f89e2ce72192c98035fd26cb0eb797a86f8f0&scene=21#wechat_redirect)