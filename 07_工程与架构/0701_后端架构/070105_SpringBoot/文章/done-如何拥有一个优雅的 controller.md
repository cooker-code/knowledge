> 已吸收至：[[07_工程与架构/0701_后端架构/070105_SpringBoot/070105_核心知识点/SpringBoot接口集成与运行治理边界|SpringBoot接口集成与运行治理边界]]
---
title: 如何拥有一个优雅的 controller
author: 业余草
date:
url: http://mp.weixin.qq.com/s?__biz=MzIyODE5NjUwNQ==&mid=2653371922&idx=1&sn=7744eb25710da5c03bf6b0827028b6f0&chksm=f38637e4c4f1bef21805d6f091dbef57af8538aff91a97c2dd441c246de7555774d10b79a7b8&mpshare=1&scene=24&srcid=0508uLqiR6FNNd0Igq9c0xvg&sharer_shareinfo=5ff65b52a256ace7f3b895906de2b5c5&sharer_shareinfo_first=5ff65b52a256ace7f3b895906de2b5c5#rd
---

来源：juejin.cn/post/7357172505961578511

推荐：https://t.zsxq.com/4LrDL

在开始之前，推荐一下我个墨问，可以扫码关注哈。

如果感兴趣，也可以微信私我进水群，每天闲聊，寻找乐子。

## 前言

见过几千行代码的 [controller](http://mp.weixin.qq.com/s?__biz=MzIyODE5NjUwNQ==&mid=2653356760&idx=1&sn=3941ed90ed401ee9ef88ac1991c4b95e&chksm=f3860b2ec4f182387d77600f3d83bc1a1ee40d0cfc38f8fe8cf326a34e5f9e408f76cb2366eb&scene=21#wechat_redirect)吗？我见过。

见过全是 try catch 的 controller 吗，我见过。

见过全是字段校验的 controller 吗，我见过。

见过全是业务代码的 [controller](http://mp.weixin.qq.com/s?__biz=MzIyODE5NjUwNQ==&mid=2653352012&idx=1&sn=76f479033e722d7eed38f92edc09e7b5&chksm=f387f9bac4f070aca918dca8a0832c3f5dd6496e58327c1fb250afd6451b43724c01e485c89f&scene=21#wechat_redirect) 吗？不好意思，我们公司很多业务写在 controller 的。

看见这些我真的血压高。

下面我们开始正文！

### 不优雅的 controller

```
@RestController
@RequestMapping("/user/test")
public class UserController {
    private static Logger logger = LoggerFactory.getLogger(UserController.class);

    @Autowired
    private UserService userService;

    @Autowired
    private AuthService authService;

    @PostMapping
    public CommonResult userRegistration(@RequestBody UserVo userVo) {
        if (StringUtils.isBlank(userVo.getUsername())){
            return CommonResult.error("用户名不能为空");
        }
        if (StringUtils.isBlank(userVo.getPassword())){
            return CommonResult.error("密码不能为空");
        }
        logger.info("注册用户：{}" , userVo.getUsername());
        try {
            userService.registerUser(userVo.getUsername());
            return CommonResult.ok();
        }catch (Exception e){
            logger.error("注册用户失败：{}", userVo.getUsername(), e);
            return CommonResult.error("注册失败");
        }
    }

    @PostMapping("/login")
    @PermitAll
    @ApiOperation("使用账号密码登录")
    public CommonResult<AuthLoginRespVO> login(@RequestBody AuthLoginReqVO reqVO) {
        if (StringUtils.isBlank(reqVO.getUsername())){
            return CommonResult.error("用户名不能为空");
        }
        if (StringUtils.isBlank(reqVO.getPassword())){
            return CommonResult.error("密码不能为空");
        }
        try {
            return success(authService.login(reqVO));
        }catch (Exception e){
            logger.error("注册用户失败：{}", reqVO.getUsername(), e);
            return CommonResult.error("注册失败");
        }
    }

}
```

### 优雅的controller

```
@RestController
@RequestMapping("/user/test")
public class UserController1 {
    private static Logger logger = LoggerFactory.getLogger(UserController1.class);

    @Autowired
    private UserService userService;

    @Autowired
    private AuthService authService;

    @PostMapping("/userRegistration")
    public CommonResult userRegistration(@RequestBody @Valid UserVo userVo) {
        userService.registerUser(userVo.getUsername());
        return CommonResult.ok();
    }

    @PostMapping("/login")
    @PermitAll
    @ApiOperation("使用账号密码登录")
    public CommonResult<AuthLoginRespVO> login(@RequestBody @Valid AuthLoginReqVO reqVO) {
        return success(authService.login(reqVO));
    }

}
```

> ❝
>
> 代码量直接减一半呀，这还不算上有些直接把业务逻辑写在 controller 的，看到这些我真的直接吐血

## 改造流程

### 校验方式

> ❝
>
> 这个 if 校验看得我哪哪都不爽。好歹给我写一个断言吧。Assert.notNull(userVo.getUsername(), "用户名不能为空");
>
> 这不香吗？确实不香。
>
> 使用 spring 提供的@Valid

* 在入参时使用@Valid注解，并且在 vo 中使用校验注解，如AuthLoginReqVO

```
@ApiModel(value = "管理后台 - 账号密码登录 Request VO")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class AuthLoginReqVO {

    @ApiModelProperty(value = "账号", required = true, example = "user")
    @NotEmpty(message = "登录账号不能为空")
    @Length(min = 4, max = 16, message = "账号长度为 4-16 位")
    @Pattern(regexp = "^[A-Za-z0-9]+$", message = "账号格式为数字以及字母")
    private String username;

    @ApiModelProperty(value = "密码", required = true, example = "password")
    @NotEmpty(message = "密码不能为空")
    @Length(min = 4, max = 16, message = "密码长度为 4-16 位")
    private String password;

}
```

### @Valid

在[SpringBoot](http://mp.weixin.qq.com/s?__biz=MzIyODE5NjUwNQ==&mid=2653371792&idx=1&sn=222c614010da923e9fe249a7daafcd3c&chksm=f3863466c4f1bd707286ded3006fee8561b22c7ed7b13cd62b81d3378ac88eda415e93954d7b&scene=21#wechat_redirect)中，@Valid是一个非常有用的注解，主要用于数据校验。以下是关于@Valid的一些详细信息：

1. `为什么使用 @Valid 来验证参数`：在编写接口时，我们经常需要验证请求参数。通常，我们可能会写大量的 if 和 if else 代码来进行判断。但这样的代码不仅不优雅，而且如果存在大量的验证逻辑，这会使代码看起来混乱，大大降低代码可读性。为了简化这个过程，我们可以使用 @Valid 注解来帮助我们简化验证逻辑。
2. `@Valid 注解的作用`：@Valid 的主要作用是用于数据效验，可以在定义的`实体中的属性上`，添加不同的注解来完成不同的校验规则，而在接口类中的接收数据参数中添加 @valid 注解，这时你的实体将会开启一个校验的功能。
3. `@Valid 的相关注解`：在实体类中不同的属性上添加不同的注解，就能实现不同数据的效验功能。
4. `使用 @Valid 进行参数效验步骤`：整个过程如下，用户访问接口，然后进行参数效验，因为 @Valid 不支持平面的参数效验（直接写在参数中字段的效验）所以基于 [GET](http://mp.weixin.qq.com/s?__biz=MzIyODE5NjUwNQ==&mid=2653339473&idx=2&sn=397d070259297f6e770a349f3454b078&chksm=f387b6a7c4f03fb12612b069e2ebe4e2a93adbfe7c565a8644cf786db18e10165b8fe0415dbb&scene=21#wechat_redirect) 请求的参数还是按照原先方式进行效验，而 POST 则可以以实体对象为参数，可以使用 @Valid 方式进行效验。如果效验通过，则进入业务逻辑，否则抛出异常，交由全局异常处理器进行处理。
5. `@Validated与@Valid的区别`：`@Validated` 是 `@Valid` 的变体。通过声明实体中属性的 `groups` ，再搭配使用 `@Validated` ，就能决定哪些属性需要校验，哪些不需要校验。

### 全局异常处理

* 这个全局异常处理，可以根据自己的异常，自定义异常处理，并设置一个兜底的异常处理

```
@ResponseBody
@RestControllerAdvice
public class ExceptionHandlerAdvice {
    protected Logger logger = LoggerFactory.getLogger(getClass());

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public CommonResult<Object> handleValidationExceptions(MethodArgumentNotValidException ex) {
        logger.error("[handleValidationExceptions]", ex);
        StringBuilder sb = new StringBuilder();
        ex.getBindingResult().getAllErrors().forEach(error -> {
            String fieldName = ((org.springframework.validation.FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            sb.append(fieldName).append(":").append(errorMessage).append(";");
        });
        return CommonResult.error(sb.toString());
    }

    /**
     * 处理系统异常，兜底处理所有的一切
     */
    @ExceptionHandler(value = Exception.class)
    public CommonResult<?> defaultExceptionHandler(Throwable ex) {
        logger.error("[defaultExceptionHandler]", ex);
        // 返回 ERROR CommonResult
        return CommonResult.error(INTERNAL_SERVER_ERROR.getCode(), INTERNAL_SERVER_ERROR.getMsg());
    }
}
```

> ❝
>
> 就这么多，搞定，这样就拥有了漂流优雅的 controller 了。

注意哈，做项目是可以这样用的。但是，如果是 [saas](http://mp.weixin.qq.com/s?__biz=MzIyODE5NjUwNQ==&mid=2653365572&idx=1&sn=3b9d01462e4159cccf71b8c962d4b2c2&chksm=f3862cb2c4f1a5a48a075c124c51ec84759c12dedbe6447b8d4c4194ced6639b580abdad7096&scene=21#wechat_redirect) 系统，或者是 paas 应用。在灵活性上就会被受到限制。在这种情况下，固定字段的校验就不行了。得根据配置来，所以，需要一个拦截器，根据对于模型，页面配置等对用到的字段进行合法校验。

这种校验方式，是在请求进入 Controller 之前就做了，同样的也精简了 [Controller](http://mp.weixin.qq.com/s?__biz=MzIyODE5NjUwNQ==&mid=2653333232&idx=2&sn=f03857381c1874029dc3b07d4ff9dd74&chksm=f387af06c4f026101f355725e92464e9a1ef8e51b4edd86164bf553eea132a5bdb9710ddaacf&scene=21#wechat_redirect) 里的校验逻辑。

另外，即使要校验，也可以多用用充血模型。

## 在日常开发中，还有那些血压飙升瞬间

* 我拿出下图阁下如何面对

* 这个阁下又如何面对，我不说，你能知道这个什么吗【狗头】

## 总结

* 不是很明白为什么有些喜欢在 controller 写业务逻辑的，曾经有个同事问我（就是喜欢在 controller 写业务的），你这个接口写在那里，我需要调一下你这个接口。我满脸问号？？不是隔壁的模块吗，为什么要调我的接口？直接引用的我的 [service](http://mp.weixin.qq.com/s?__biz=MzIyODE5NjUwNQ==&mid=2653324090&idx=1&sn=f629dda2c8a87c52c431613e7b8bb53c&chksm=f3878accc4f003da8d92fb29f4c93c662b7dc7894d10de1f63f6e6770df380c290e7cdb5165e&scene=21#wechat_redirect) 去调方法就好了。
* 这个就是痛点，各写各的，冗余代码一堆。
* 曾经看到一个同事写一个保存的方法，虽然逻辑挺多，我滑动了好久都还没有方法还没有结束。一个方法整整几百行……
* 看过 spring 源码都知道，spring 源码难啃，就是因为 [spring](http://mp.weixin.qq.com/s?__biz=MzIyODE5NjUwNQ==&mid=2653371372&idx=1&sn=e2a398a8d494f8c285a194b8dbefb2d2&chksm=f386321ac4f1bb0cc87417b675ef530d6cbdb45e5659232dc323855e4988a83181ab998b6492&scene=21#wechat_redirect) 无限往下套娃，基本每个方法干每个方法的事情。比如我保存用户时，就只是保存用户，至于什么校验丢给校验的方法处理，什么发送消息丢给发送消息处理，这些就不能耦合在一起。
* 对于看到一些 if 下面一丢逻辑，然后 if 再一丢逻辑，看代码时很多情况不需要知道这个逻辑怎么实现的，知道入参出参就大概这里做什么了。即使想知道详细情况点进去就知道了。突出这个当前方法要做的事情就好了。
* 阿里的开发手册就推荐一个方法不能超过 80 行，超过可以根据业务具体调整一下。