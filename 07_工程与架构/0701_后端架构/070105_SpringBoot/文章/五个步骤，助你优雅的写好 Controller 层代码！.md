---
title: 五个步骤，助你优雅的写好 Controller 层代码！
author: 大侠学JAVA
date: 
url: http://mp.weixin.qq.com/s?__biz=MzA4MDMyODg4OQ==&mid=2649490858&idx=1&sn=4ed68f7093b728ab5a5c5317fecba597&chksm=87bd447cb0cacd6a61a8959750be8a9dfa618c42d4f755bab40a33ea70e4a71fb8b980648f83&mpshare=1&scene=24&srcid=0606JQyzgDk6enskFc5bsqLa&sharer_sharetime=1686047030558&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

# Controller 层逻辑

MVC架构下，我们的web工程结构会分为三层，自下而上是dao层，service层和controller层。controller层为控制层，主要处理外部请求，调用service层。

一般情况下，controller层不应该包含业务逻辑，controller的功能应该有以下五点：

⑴、接收请求并解析参数

⑵、业务逻辑执行成功做出响应

⑶、异常处理

⑷、转换业务对象

⑸、调用 Service 接口

# 普通写法

```
@RestController  
public class TestController {  
  
    @Autowired  
    private UserService userService;  
  
    @PostMapping("/test")  
 public Result service(@Validated  @RequesBody  UserRequestBo requestBo) throws Exception {  
        Result result = new Result();  
        // 参数校验  
        if (StringUtils.isNotEmpty(requestBo.getId())  
                || StringUtils.isNotEmpty(requestBo.getType())  
                || StringUtils.isNotEmpty(requestBo.getName())  
                || StringUtils.isNotEmpty(requestBo.getAge())) {  
            throw new Exception("必输项校验失败");  
        } else {  
            // 调用service更新user，更新可能抛出异常，要捕获  
            try {  
                int count = 0;  
                User user = userService.queryUser(requestBo.getId());  
                if (ObjectUtils.isEmpty(user)) {  
                    result.setCode("11111111111");  
                    result.setMessage("请求失败");  
                    return result;  
                }  
                // 转换业务对象  
                UserDTO userDTO = new UserDTO();  
                BeanUtils.copyProperties(requestBo, userDTO);  
                if ("02".equals(user.getType())) {// 退回修改的更新  
                    count = userService.updateUser(userDTO)  
                }else if ("03".equals(user.getType())) {// 已生效状态，新增一条待复核  
                    count = userService.addUser(userDTO);  
                }  
                // 组装返回对象  
                result.setData(count);  
                result.setCode("00000000");  
                result.setMessage("请求成功");  
            } catch (Exception ex) {  
                // 异常处理  
                result.setCode("111111111111");  
                result.setMessage("请求失败");  
            }  
        }  
        return result;  
    }  
}
```

# 优化思路

### 1、调用 Service 层接口

一般情况下，controller作为控制层调用service层接口，不应该包含任何业务逻辑，所有的业务操作，都放在service层实现，把controller层相关代码去掉

**controller层就变成了：**

```
@RestController  
public class TestController {  
  
@Autowired  
private UserService userService;  
  
@PostMapping("/test")  
public Result service(@Validated  @RequesBody  UserRequestBo requestBo) throws Exception {  
    Result result = new Result();  
    // 参数校验  
    if (StringUtils.isNotEmpty(requestBo.getId())  
            || StringUtils.isNotEmpty(requestBo.getType())  
            || StringUtils.isNotEmpty(requestBo.getName())  
            || StringUtils.isNotEmpty(requestBo.getAge())) {  
        throw new Exception("必输项校验失败");  
    } else {  
        // 调用service更新user，更新可能抛出异常，要捕获  
        try {  
         // 转换业务对象  
            UserDTO userDTO = new UserDTO();  
            BeanUtils.copyProperties(requestBo, userDTO);  
            int count = userService.updateUser(userDTO);  
            // 组装返回对象  
            result.setData(count);  
            result.setCode("00000000");  
            result.setMessage("请求成功");  
        } catch (Exception ex) {  
            // 异常处理  
            result.setCode("EEEEEEEE");  
            result.setMessage("请求失败");  
        }  
    }  
    return result;  
}
```

### 2、参数校验

其实大多数的参数校验就是判空或者空字符串，那么我们可以用`@NotBlank`等注解。在UserRequestBo类中name属性上加上`@NotBlank`注解

**优化后如下：**

```
@Data  
public class UserRequestBo {  
  
    @NotBlank  
    private String id;  
  
    @NotBlank  
    private String type;  
  
    @NotBlank  
    private String name;  
  
    @NotBlank  
    private String age;  
}
```

**controller层就变成了：**

```
@RestController  
public class TestController {  
  
    @Autowired  
    private UserService userService;  
  
    @PostMapping("/test")  
    public Result service( @Validated  @RequesBody  UserRequestBo requestBo) throws Exception {  
        Result result = new Result();  
        // 调用service更新user，更新可能抛出异常，要捕获  
        try {  
         // 转换业务对象  
            UserDTO userDTO = new UserDTO();  
            BeanUtils.copyProperties(requestBo, userDTO);  
            int count = userService.updateUser(userDTO);  
            // 组装返回对象  
            result.setData(count);  
            result.setCode("00000000");  
            result.setMessage("请求成功");  
        } catch (Exception ex) {  
            // 异常处理  
            result.setCode("EEEEEEEE");  
            result.setMessage("请求失败");  
        }  
        return result;  
    }  
}
```

> 备注：`@NotNull`、`@NotBlank`、`@NotEmpty`的区别，也适用于代码中的校验方法

* **@NotNull：** 平常用于基本数据的包装类(Integer，Long，Double等等)，如果@NotNull 注解被使用在 String 类型的数据上，则表示该数据不能为 Null，但是可以为空字符串(“”)，空格字符串（“ ”）等。
* **@NotEmpty：** 平常用于 String、Collection集合、Map、数组等等，`@NotEmpty` 注解的参数不能为 Null 或者 长度为 0，如果用在String类型上，则字符串也不能为空字符串(“”), 但是空格字符串（“ ”）不会被校验住。
* **@NotBlank：** 平常用于 String 类型的数据上，加了`@NotBlank` 注解的参数不能为 Null ，不能为空字符串(“”), 也不能会空格字符串（“ ”），多了一个`trim()`得到效果。

### 3、统一封装返回对象

代码中无论是业务成功或者失败，都需要封装返回对象，目前代码中都是哪里用到就在哪里进行封装

我们可以统一封装返回对象

**优化后如下：**

```
@Data  
public class Result<T> {  
  
    private String code;  
  
    private String message;  
  
    private T data;  
  
 // 请求成功，指定data  
    public static <T> Result<T> success(T data) {  
        return new Result<>(ResultEnum.SUCCESS.getCode(), ResultEnum.SUCCESS.getMessage(), data);  
    }  
      
 // 请求成功，指定data和指定message  
    public static <T> Result<T> success(String message, T data) {  
        return new Result<>(ResultEnum.SUCCESS.getCode(), message, data);  
    }  
      
 // 请求失败  
    public static Result<?> failed() {  
        return new Result<>(ResultEnum.COMMON_FAILED.getCode(), ResultEnum.COMMON_FAILED.getMessage(), null);  
    }  
      
 // 请求失败，指定message  
    public static Result<?> failed(String message) {  
        return new Result<>(ResultEnum.COMMON_FAILED.getCode(), message, null);  
    }  
      
    // 请求失败，指定code和message  
    public static Result<?> failed(String code, String message) {  
        return new Result<>(code, message, null);  
    }  
}
```

**controller层就变成了：**

```
@RestController  
public class TestController {  
  
    @Autowired  
    private UserService userService;  
  
    @PostMapping("/test")  
    public Result service(@Validated  @RequesBody  UserRequestBo requestBo) throws Exception {  
        // 调用service更新user，更新可能抛出异常，要捕获  
        try {  
         // 转换业务对象  
            UserDTO userDTO = new UserDTO();  
            BeanUtils.copyProperties(requestBo, userDTO);  
            int count = userService.updateUser(userDTO);  
            // 组装返回对象  
            Result.success(count);  
        } catch (Exception ex) {  
            // 异常处理  
            Result.failed(ex.getMessage());  
        }  
    }  
} 
```

### 4、统一的异常捕获

Controller层和service存在大量的try-catch，都是重复代码并且看起来也不优雅。可以给controller层的方法加上切面来统一处理异常。

用`@ControllerAdvice`注解(`@RestControllerAdvice`也可以)，用来定义controller层的切面，添加`@Controller`注解的类中的方法执行都会进入该切面，同时我们可以使用`@ExceptionHandler`来对不同的异常进行捕获和处理，对于捕获的异常，我们可以进行日志记录，并且封装返回对象。

**优化后如下：**

```
// @RestControllerAdvice(basePackages = "com.ruoyi.web.controller.demo.test"), 指定包路径进行切面  
// @RestControllerAdvice(basePackageClasses = TestController.class) , 指定Contrller.class进行切面  
// @RestControllerAdvice 不带参数默认覆盖所有添加@Controller注解的类  
@RestControllerAdvice(basePackageClasses = TestController.class)  
public class TestControllerAdvice {  
  
    @Autowired  
    HttpServletRequest httpServletRequest;  
  
    private void logErrorRequest(Exception e){  
        // 组装日志内容  
        String logInfo = String.format("报错API URL: %S, error = ", httpServletRequest.getRequestURI(), e.getMessage());  
        // 打印日志  
        System.out.println(logInfo);  
    }  
  
    /**  
     * {@code @RequestBody} 参数校验不通过时抛出的异常处理  
     */  
    @ExceptionHandler({MethodArgumentNotValidException.class})  
    public Result handleMethodArgumentNotValidException(MethodArgumentNotValidException ex) {  
        // 打印日志  
        logErrorRequest(ex);  
        // 组织异常信息，可能存在多个参数校验失败  
        BindingResult bindingResult = ex.getBindingResult();  
        StringBuilder sb = new StringBuilder("校验失败:");  
        for (FieldError fieldError : bindingResult.getFieldErrors()) {  
       sb.append(fieldError.getField()).append("：").append(fieldError.getDefaultMessage()).append(", ");  
        }  
        return Result.failed(ResultEnum.VALIDATE_FAILED.getCode(), sb.toString());  
    }  
  
    /**  
     * 业务层异常，如果项目中有自定义异常则使用自定义业务异常，如果没有，可以和其他异常一起处理  
     *  
     * @param exception  
     * @return  
     */  
    @ExceptionHandler(RuntimeException.class)  
    protected Result serviceException(RuntimeException exception) {  
        logErrorRequest(exception);  
        return Result.failed(exception.getMessage());  
    }  
  
    /**  
     * 其他异常  
     *  
     * @param exception  
     * @return  
     */  
    @ExceptionHandler({HttpClientErrorException.class, IOException.class, Exception.class})  
    protected Result serviceException(Exception exception) {  
        logErrorRequest(exception);  
        return Result.failed(exception.getMessage());  
    }  
}
```

**controller层就变成了：**

```
@RestController  
public class TestController {  
  
    @Autowired  
    private UserService userService;  
  
    @PostMapping("/test")  
    public Result service( @Validated  @RequesBody  UserRequestBo requestBo) throws Exception {  
        UserDTO userDTO = new UserDTO();  
        BeanUtils.copyProperties(requestBo, userDTO);  
        // 调用service层接口  
        int count = userService.updateUser(userDTO);  
        //组装返回对象  
        return Result.success(count);  
    }  
}
```

### 5、转换业务对象

代码中可能有很多个地方转换同一个业务对象，入参`UserRequestBo`可以转换为userDTO，可以理解为这是`UserRequestBo`的一个特性或者能力，我们可以参考充血模式的思想，在`UserRequestBo`中定义`convertToUserDTO`方法，我们的目的是转换业务对象，至于使用什么方式转换，调用方并不关心，现在使用的`BeanUtils.copyProperties()`，如果有一天想修改成使用Mapstruct来进行对象转换，只需要修改`UserRequestBo`的`convertToUserDTO`方法即可，不会涉及到业务代码的修改。

**优化后代码：**

```
@Data  
public class UserRequestBo {  
  
    @NotBlank  
    private String id;  
  
    @NotBlank  
    private String type;  
  
    @NotBlank  
    private String name;  
  
    @NotBlank  
    private String age;  
  
    /**  
     * UserRequestBo对象为UserDTO  
     * */  
    public UserDTO convertToUserDTO(){  
        UserDTO userDTO = new UserDTO();  
        // BeanUtils.copyProperties要求字段名和字段类型都要保持一致，如果有不一样的字段，需要单独set  
        BeanUtils.copyProperties(this, userDTO);  
        userDTO.setType(this.getType());  
        return userDTO;  
    }  
}
```

**controller层就变成了：**

```
@RestController  
public class TestController {  
  
    @Autowired  
    private UserService userService;  
  
    @PostMapping("/test")  
    public Result service(@Validated  @RequesBody  UserRequestBo requestBo) throws Exception {  
        return Result.success(userService.updateUser(requestBo.convertToUserDTO()));  
    }  
}
```

优化结束，打完收工。

> ### *来源：blog.csdn.net/weixin\_44271364/article/*
>
> ### *details/129157011*