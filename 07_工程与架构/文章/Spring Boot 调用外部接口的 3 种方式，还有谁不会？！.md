---
title: Spring Boot 调用外部接口的 3 种方式，还有谁不会？！
author: Java高性能架构
date: 
url: http://mp.weixin.qq.com/s?__biz=MzkwODMzOTY1NA==&mid=2247525856&idx=1&sn=7c53ea510be948b6ad33561601f81277&chksm=c0c95549f7bedc5f662ab7ca77475bcfc80c05e36e6713446eca269b8e68f616df001dfb62af&mpshare=1&scene=24&srcid=062191IfVpCMFURWkYycmqiK&sharer_sharetime=1687326683102&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

关注上方蓝色“Java高性能架构”，设为星标⭐

回复“**资料**”获取整理好的**面试资料**

大家好,我是 风哥

## 1、简介

SpringBoot不仅继承了Spring框架原有的优秀特性，而且还通过简化配置来进一步简化了Spring应用的整个搭建和开发过程。

在Spring-Boot项目开发中，存在着本模块的代码需要访问外面模块接口，或外部url链接的需求, 比如在apaas开发过程中需要封装接口在接口中调用apaas提供的接口（像发起流程接口submit等等）下面也是提供了三种方式（不使用dubbo的方式）供我们选择

推荐一个开源免费的 Spring Boot 实战项目：

> https://github.com/javastacks/spring-boot-best-practice

## 2、方式一：使用原始httpClient请求

```
/*  
 * @description get方式获取入参，插入数据并发起流程  
 * @author lyx  
 * @date 2022/8/24 16:05  
 * @params documentId  
 * @return String  
 */  
//  
@RequestMapping("/submit/{documentId}")  
public String submit1(@PathVariable String documentId) throws ParseException {  
    //此处将要发送的数据转换为json格式字符串  
    Map<String,Object> map =task2Service.getMap(documentId);  
    String jsonStr = JSON.toJSONString(map, SerializerFeature.WRITE_MAP_NULL_FEATURES,SerializerFeature.QuoteFieldNames);  
    JSONObject jsonObject = JSON.parseObject(jsonStr);  
    JSONObject sr = task2Service.doPost(jsonObject);  
    return sr.toString();  
}  
/*  
 * @description 使用原生httpClient调用外部接口  
 * @author lyx  
 * @date 2022/8/24 16:08  
 * @params date  
 * @return JSONObject  
 */  
public static JSONObject doPost(JSONObject date) {  
    String assessToken="eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJ4ZGFwYXBwaWQiOiIzNDgxMjU4ODk2OTI2OTY1NzYiLCJleHAiOjE2NjEyMjY5MDgsImlhdCI6MTY2MTIxOTcwOCwieGRhcHRlbmFudGlkIjoiMzAwOTgxNjA1MTE0MDUyNjA5IiwieGRhcHVzZXJpZCI6IjEwMDM0NzY2MzU4MzM1OTc5NTIwMCJ9.fZAO4kJSv2rSH0RBiL1zghdko8Npmu_9ufo6Wex_TI2q9gsiLp7XaW7U9Cu7uewEOaX4DTdpbFmMPvLUtcj_sQ";  
    CloseableHttpClient client = HttpClients.createDefault();  
    // 要调用的接口url  
    String url = "http://39.103.201.110:30661 /xdap-open/open/process/v1/submit";  
    HttpPost post = new HttpPost(url);  
    JSONObject jsonObject = null;  
    try {  
        //创建请求体并添加数据  
        StringEntity s = new StringEntity(date.toString());  
        //此处相当于在header里头添加content-type等参数  
        s.setContentType("application/json");  
        s.setContentEncoding("UTF-8");  
        post.setEntity(s);  
        //此处相当于在Authorization里头添加Bear token参数信息  
        post.addHeader("Authorization", "Bearer " +assessToken);  
        HttpResponse res = client.execute(post);  
        String response1 = EntityUtils.toString(res.getEntity());  
        if (res.getStatusLine()  
                .getStatusCode() == HttpStatus.SC_OK) {  
            // 返回json格式：  
            String result = EntityUtils.toString(res.getEntity());  
            jsonObject = JSONObject.parseObject(result);  
        }  
    } catch (Exception e) {  
        throw new RuntimeException(e);  
    }  
    return jsonObject;  
}
```

## 3、方式二：使用RestTemplate方法

Spring-Boot开发中，`RestTemplate`同样提供了对外访问的接口API，这里主要介绍Get和Post方法的使用。

#### Get请求

提供了`getForObject` 、`getForEntity`两种方式，其中`getForEntity`如下三种方法的实现：

`Get--getForEntity`，存在以下两种方式重载

```
1.getForEntity(Stringurl,Class responseType,Object…urlVariables)  
2.getForEntity(URI url,Class responseType)
```

**`Get--getForEntity(URI url,Class responseType)`**

```
//该方法使用URI对象来替代之前的url和urlVariables参数来指定访问地址和参数绑定。URI是JDK java.net包下的一个类，表示一个统一资源标识符(Uniform Resource Identifier)引用。参考如下：  
RestTemplate restTemplate=new RestTemplate();  
UriComponents   
uriComponents=UriComponentsBuilder.fromUriString("http://USER-SERVICE/user?name={name}")  
.build()  
.expand("dodo")  
.encode();  
URI uri=uriComponents.toUri();  
ResponseEntityresponseEntity=restTemplate.getForEntity(uri,String.class).getBody();
```

**`Get--getForEntity(Stringurl,Class responseType,Object…urlVariables)`**

```
//该方法提供了三个参数，其中url为请求的地址，responseType为请求响应body的包装类型，urlVariables为url中的参数绑定，该方法的参考调用如下：  
// http://USER-SERVICE/user?name={name)  
RestTemplate restTemplate=new RestTemplate();  
Mapparams=new HashMap<>();  
params.put("name","dada"); //  
ResponseEntityresponseEntity=restTemplate.getForEntity("http://USERSERVICE/user?name={name}",String.class,params);
```

Get--getForObject，存在以下三种方式重载

```
1.getForObject(String url,Class responseType,Object...urlVariables)  
2.getForObject(String url,Class responseType,Map urlVariables)  
3.getForObject(URI url,Class responseType)
```

getForObject方法可以理解为对getForEntity的进一步封装，它通过`HttpMessageConverterExtractor`对HTTP的请求响应体body内容进行对象转换，实现请求直接返回包装好的对象内容。

另外，如果你近期准备面试跳槽，建议在Java面试库小程序在线刷题，涵盖 2000+ 道 Java 面试题，几乎覆盖了所有主流技术面试题。

#### Post 请求

Post请求提供有`postForEntity`、`postForObject`和`postForLocation`三种方式，其中每种方式都有三种方法，下面介绍`postForEntity`的使用方法。

Post--postForEntity，存在以下三种方式重载

```
1.postForEntity(String url,Object request,Class responseType,Object...  uriVariables)   
2.postForEntity(String url,Object request,Class responseType,Map  uriVariables)   
3.postForEntity(URI url,Object request，Class responseType)
```

如下仅演示第二种重载方式

```
/*  
 * @description post方式获取入参，插入数据并发起流程  
 * @author lyx  
 * @date 2022/8/24 16:07  
 * @params  
 * @return  
 */  
@PostMapping("/submit2")  
public Object insertFinanceCompensation(@RequestBody JSONObject jsonObject) {  
    String documentId=jsonObject.get("documentId").toString();  
    return task2Service.submit(documentId);  
}  
/*  
 * @description 使用restTimeplate调外部接口  
 * @author lyx  
 * @date 2022/8/24 16:02  
 * @params documentId  
 * @return String  
 */  
public String submit(String documentId){  
    String assessToken="eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJ4ZGFwYXBwaWQiOiIzNDgxMjU4ODk2OTI2OTY1NzYiLCJleHAiOjE2NjEyMjY5MDgsImlhdCI6MTY2MTIxOTcwOCwieGRhcHRlbmFudGlkIjoiMzAwOTgxNjA1MTE0MDUyNjA5IiwieGRhcHVzZXJpZCI6IjEwMDM0NzY2MzU4MzM1OTc5NTIwMCJ9.fZAO4kJSv2rSH0RBiL1zghdko8Npmu_9ufo6Wex_TI2q9gsiLp7XaW7U9Cu7uewEOaX4DTdpbFmMPvLUtcj_sQ";  
    RestTemplate restTemplate = new RestTemplate();  
    //创建请求头  
    HttpHeaders httpHeaders = new HttpHeaders();  
    //此处相当于在Authorization里头添加Bear token参数信息  
    httpHeaders.add(HttpHeaders.AUTHORIZATION, "Bearer " + assessToken);  
    //此处相当于在header里头添加content-type等参数  
    httpHeaders.add(HttpHeaders.CONTENT_TYPE,"application/json");  
    Map<String, Object> map = getMap(documentId);  
    String jsonStr = JSON.toJSONString(map);  
    //创建请求体并添加数据  
    HttpEntity<Map> httpEntity = new HttpEntity<Map>(map, httpHeaders);  
    String url = "http://39.103.201.110:30661/xdap-open/open/process/v1/submit";  
    ResponseEntity<String> forEntity = restTemplate.postForEntity(url,httpEntity,String.class);//此处三个参数分别是请求地址、请求体以及返回参数类型  
    return forEntity.toString();  
}
```

## 4、方式三：使用Feign进行消费

在maven项目中添加依赖

```
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-feign</artifactId>  
    <version>1.2.2.RELEASE</version>  
</dependency>
```

启动类上加上`@EnableFeignClients`

```
@SpringBootApplication  
@EnableFeignClients  
@ComponentScan(basePackages = {"com.definesys.mpaas", "com.xdap.*" ,"com.xdap.*"})  
public class MobilecardApplication {  
   
    public static void main(String[] args) {  
        SpringApplication.run(MobilecardApplication.class, args);  
    }  
   
}
```

**此处编写接口模拟外部接口供feign调用外部接口方式使用**

定义controller

```
@Autowired  
PrintService printService;  
  
@PostMapping("/outSide")  
public String test(@RequestBody TestDto testDto) {  
    return printService.print(testDto);  
}
```

定义service

```
@Service  
public interface PrintService {  
    public String print(TestDto testDto);  
}
```

定义serviceImpl

```
public class PrintServiceImpl implements PrintService {  
   
    @Override  
    public String print(TestDto testDto) {  
        return "模拟外部系统的接口功能"+testDto.getId();  
    }  
}
```

**构建Feigin的Service**

定义service

```
//此处name需要设置不为空，url需要在.properties中设置  
@Service  
@FeignClient(url = "${outSide.url}", name = "service2")  
public interface FeignService2 {  
    @RequestMapping(value = "/custom/outSide", method = RequestMethod.POST)  
    @ResponseBody  
    public String getMessage(@Valid @RequestBody TestDto testDto);  
}
```

定义controller

```
@Autowired  
FeignService2 feignService2;  
//测试feign调用外部接口入口  
@PostMapping("/test2")  
public String test2(@RequestBody TestDto testDto) {  
    return feignService2.getMessage(testDto);  
}
```

**postman测试**

此处因为我使用了所在项目，所以需要添加一定的请求头等信息，关于Feign的请求头添加也会在后续补充。

补充如下：

#### 添加Header解决方法

将token等信息放入Feign请求头中，主要通过重写`RequestInterceptor`的apply方法实现

定义config

```
@Configuration  
public class FeignConfig implements RequestInterceptor {  
    @Override  
    public void apply(RequestTemplate requestTemplate) {  
        //添加token  
        requestTemplate.header("token", "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJ4ZGFwYXBwaWQiOiIzNDgxMjU4ODk2OTI2OTY1NzYiLCJleHAiOjE2NjEyMjY5MDgsImlhdCI6MTY2MTIxOTcwOCwieGRhcHRlbmFudGlkIjoiMzAwOTgxNjA1MTE0MDUyNjA5IiwieGRhcHVzZXJpZCI6IjEwMDM0NzY2MzU4MzM1OTc5NTIwMCJ9.fZAO4kJSv2rSH0RBiL1zghdko8Npmu_9ufo6Wex_TI2q9gsiLp7XaW7U9Cu7uewEOaX4DTdpbFmMPvLUtcj_sQ");  
    }  
}
```

定义service

```
@Service  
@FeignClient(url = "${outSide.url}",name = "feignServer", configuration = FeignDemoConfig.class)  
public interface TokenDemoClient {  
    @RequestMapping(value = "/custom/outSideAddToken", method = RequestMethod.POST)  
    @ResponseBody  
    public String getMessage(@Valid @RequestBody TestDto testDto);  
}
```

定义controller

```
//测试feign调用外部接口入口，加上token  
@PostMapping("/testToken")  
public String test4(@RequestBody TestDto testDto) {  
    return tokenDemoClient.getMessage(testDto);  
}
```

> 版权声明：本文为CSDN博主「Chelsea」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。原文链接：https://blog.csdn.net/Chelsea/article/details/126689495

# 

End

风哥联合一线大厂朋友花费8个月的时间，录制了一份**Java入门+进阶视频教程**

**课程特色：**

1. 总共88G，时常高达365小时，覆盖所有主流技术栈
2. 均为同一人录制，不是东拼西凑的
3. 对标线下T0级别的培训课，讲师大厂架构师，多年授课经验，通俗易懂
4. 内容丰富，每一个技术点除了视频，还有课堂源码、笔记、PPT、图解
5. 五大实战项目（视频+源码+笔记+SQL+软件）
6. 一次付费，持续更新，永无二次费用

**点击下方超链接查看详情****（或者点击文末阅读原文）：**

[***(点击查看)  88G，超全技术栈的Java入门+进阶+实战！***](http://mp.weixin.qq.com/s?__biz=MzkwODMzOTY1NA==&mid=2247514708&idx=1&sn=8082086f1ae166b6dd9037753cb76aba&chksm=c0c98afdf7be03eb203b114bcc9e78b5b977a95659b40aa8723225dbdf11b4a87f5e5f51e490&scene=21#wechat_redirect)