> 已吸收至：[[07_工程与架构/0701_后端架构/070101_Java/070101_核心知识点/Java来源校准与降权准则|Java来源校准与降权准则]]、[[07_工程与架构/0701_后端架构/070101_Java/070101_知识地图|070101_Java知识地图]]

---
title: 自从用了 OkHttp，别的都完全不想用了！
author: 浪尖聊大数据
date:
url: http://mp.weixin.qq.com/s?__biz=MzUyMDA4OTY3MQ==&mid=2247506765&idx=2&sn=1f4a6113c0c487f135117955c263e77a&chksm=f9ed2265ce9aab7377ac1193d2822905f2dfa702e1506e79b5d00e637c24dcf44c269891ab83&mpshare=1&scene=24&srcid=0601eDnOql1lWn7na1D7GJ0j&sharer_sharetime=1654038352492&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

Java封装OkHttp3工具类，适用于Java后端开发者。

[说实在话，用过挺多网络请求工具，有过java原生的，HttpClient3和4，但是个人感觉用了OkHttp3之后，之前的那些完全不想再用了。](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247542384&idx=2&sn=4f1148b7fc0090760ba0e8ef4c299ac1&chksm=eb50a346dc272a50baca4f3f935323612c0ce4283599e87550276336dcc79b9bd6bde2f627df&scene=21#wechat_redirect)

[怎么说呢，代码轻便，使用起来很很很灵活，响应快，比起HttpClient好用许多。当然，这些是我个人观点，不喜勿喷。](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247542384&idx=2&sn=4f1148b7fc0090760ba0e8ef4c299ac1&chksm=eb50a346dc272a50baca4f3f935323612c0ce4283599e87550276336dcc79b9bd6bde2f627df&scene=21#wechat_redirect)

## **准备工作**

[Maven项目在pom文件中引入jar包](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247535563&idx=2&sn=2f28d811947f61f9690336b433831c19&chksm=eb50cefddc2747ebf647ce65467630ba9f010b1558f9bd5e5156700bcdc17baf788ad0c94a86&scene=21#wechat_redirect)

```
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>3.10.0</version>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.60</version>
</dependency>
```

[引入json是因为工具类中有些地方用到了，现在通信都流行使用json传输，也少不了要这个jar包。](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247535563&idx=2&sn=2f28d811947f61f9690336b433831c19&chksm=eb50cefddc2747ebf647ce65467630ba9f010b1558f9bd5e5156700bcdc17baf788ad0c94a86&scene=21#wechat_redirect)

## [**工具类代码**](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247542384&idx=2&sn=4f1148b7fc0090760ba0e8ef4c299ac1&chksm=eb50a346dc272a50baca4f3f935323612c0ce4283599e87550276336dcc79b9bd6bde2f627df&scene=21#wechat_redirect)

```
import com.alibaba.fastjson.JSON;
import okhttp3.*;

import javax.net.ssl.SSLContext;
import javax.net.ssl.SSLSocketFactory;
import javax.net.ssl.TrustManager;
import javax.net.ssl.X509TrustManager;
import java.io.IOException;
import java.net.URLEncoder;
import java.security.SecureRandom;
import java.security.cert.X509Certificate;
import java.util.LinkedHashMap;
import java.util.Map;
import java.util.concurrent.Semaphore;
import java.util.concurrent.TimeUnit;

public class OkHttpUtils {
    private static volatile OkHttpClient okHttpClient = null;
    private static volatile Semaphore semaphore = null;
    private Map<String, String> headerMap;
    private Map<String, String> paramMap;
    private String url;
    private Request.Builder request;

    /**
     * 初始化okHttpClient，并且允许https访问
     */
    private OkHttpUtils() {
        if (okHttpClient == null) {
            synchronized (OkHttpUtils.class) {
                if (okHttpClient == null) {
                    TrustManager[] trustManagers = buildTrustManagers();
                    okHttpClient = new OkHttpClient.Builder()
                            .connectTimeout(15, TimeUnit.SECONDS)
                            .writeTimeout(20, TimeUnit.SECONDS)
                            .readTimeout(20, TimeUnit.SECONDS)
                            .sslSocketFactory(createSSLSocketFactory(trustManagers), (X509TrustManager) trustManagers[0])
                            .hostnameVerifier((hostName, session) -> true)
                            .retryOnConnectionFailure(true)
                            .build();
                    addHeader("User-Agent", "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Safari/537.36");
                }
            }
        }
    }

    /**
     * 用于异步请求时，控制访问线程数，返回结果
     *
     * @return
     */
    private static Semaphore getSemaphoreInstance() {
        //只能1个线程同时访问
        synchronized (OkHttpUtils.class) {
            if (semaphore == null) {
                semaphore = new Semaphore(0);
            }
        }
        return semaphore;
    }

    /**
     * 创建OkHttpUtils
     *
     * @return
     */
    public static OkHttpUtils builder() {
        return new OkHttpUtils();
    }

    /**
     * 添加url
     *
     * @param url
     * @return
     */
    public OkHttpUtils url(String url) {
        this.url = url;
        return this;
    }

    /**
     * 添加参数
     * 
     * @param key   参数名
     * @param value 参数值
     * @return
     */
    public OkHttpUtils addParam(String key, String value) {
        if (paramMap == null) {
            paramMap = new LinkedHashMap<>(16);
        }
        paramMap.put(key, value);
        return this;
    }

    /**
     * 添加请求头
     *
     * @param key   参数名
     * @param value 参数值
     * @return
     */
    public OkHttpUtils addHeader(String key, String value) {
        if (headerMap == null) {
            headerMap = new LinkedHashMap<>(16);
        }
        headerMap.put(key, value);
        return this;
    }

    /**
     * 初始化get方法
     *
     * @return
     */
    public OkHttpUtils get() {
        request = new Request.Builder().get();
        StringBuilder urlBuilder = new StringBuilder(url);
        if (paramMap != null) {
            urlBuilder.append("?");
            try {
                for (Map.Entry<String, String> entry : paramMap.entrySet()) {
                    urlBuilder.append(URLEncoder.encode(entry.getKey(), "utf-8")).
                            append("=").
                            append(URLEncoder.encode(entry.getValue(), "utf-8")).
                            append("&");
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
            urlBuilder.deleteCharAt(urlBuilder.length() - 1);
        }
        request.url(urlBuilder.toString());
        return this;
    }

    /**
     * 初始化post方法
     *
     * @param isJsonPost true等于json的方式提交数据，类似postman里post方法的raw
     *                   false等于普通的表单提交
     * @return
     */
    public OkHttpUtils post(boolean isJsonPost) {
        RequestBody requestBody;
        if (isJsonPost) {
            String json = "";
            if (paramMap != null) {
                json = JSON.toJSONString(paramMap);
            } 
            requestBody = RequestBody.create(MediaType.parse("application/json; charset=utf-8"), json);
        } else {
            FormBody.Builder formBody = new FormBody.Builder();
            if (paramMap != null) {
                paramMap.forEach(formBody::add);
            }
            requestBody = formBody.build();
        }
        request = new Request.Builder().post(requestBody).url(url);
        return this;
    }

    /**
     * 同步请求
     *
     * @return
     */
    public String sync() {
        setHeader(request);
        try {
            Response response = okHttpClient.newCall(request.build()).execute();
            assert response.body() != null;
            return response.body().string();
        } catch (IOException e) {
            e.printStackTrace();
            return "请求失败：" + e.getMessage();
        }
    }

    /**
     * 异步请求，有返回值
     */
    public String async() {
        StringBuilder buffer = new StringBuilder("");
        setHeader(request);
        okHttpClient.newCall(request.build()).enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                buffer.append("请求出错：").append(e.getMessage());
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                assert response.body() != null;
                buffer.append(response.body().string());
                getSemaphoreInstance().release();
            }
        });
        try {
            getSemaphoreInstance().acquire();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return buffer.toString();
    }

    /**
     * 异步请求，带有接口回调
     *
     * @param callBack
     */
    public void async(ICallBack callBack) {
        setHeader(request);
        okHttpClient.newCall(request.build()).enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                callBack.onFailure(call, e.getMessage());
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                assert response.body() != null;
                callBack.onSuccessful(call, response.body().string());
            }
        });
    }

    /**
     * 为request添加请求头
     *
     * @param request
     */
    private void setHeader(Request.Builder request) {
        if (headerMap != null) {
            try {
                for (Map.Entry<String, String> entry : headerMap.entrySet()) {
                    request.addHeader(entry.getKey(), entry.getValue());
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }


    /**
     * 生成安全套接字工厂，用于https请求的证书跳过
     *
     * @return
     */
    private static SSLSocketFactory createSSLSocketFactory(TrustManager[] trustAllCerts) {
        SSLSocketFactory ssfFactory = null;
        try {
            SSLContext sc = SSLContext.getInstance("SSL");
            sc.init(null, trustAllCerts, new SecureRandom());
            ssfFactory = sc.getSocketFactory();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return ssfFactory;
    }

    private static TrustManager[] buildTrustManagers() {
        return new TrustManager[]{
                new X509TrustManager() {
                    @Override
                    public void checkClientTrusted(X509Certificate[] chain, String authType) {
                    }

                    @Override
                    public void checkServerTrusted(X509Certificate[] chain, String authType) {
                    }

                    @Override
                    public X509Certificate[] getAcceptedIssuers() {
                        return new X509Certificate[]{};
                    }
                }
        };
    }

    /**
     * 自定义一个接口回调
     */
    public interface ICallBack {

        void onSuccessful(Call call, String data);

        void onFailure(Call call, String errorMsg);

    }
}
```

## [**使用教程**](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247542384&idx=2&sn=4f1148b7fc0090760ba0e8ef4c299ac1&chksm=eb50a346dc272a50baca4f3f935323612c0ce4283599e87550276336dcc79b9bd6bde2f627df&scene=21#wechat_redirect)

```
public static void main(String[] args) {
    // get请求，方法顺序按照这种方式，切记选择post/get一定要放在倒数第二，同步或者异步倒数第一，才会正确执行
    OkHttpUtils.builder().url("请求地址，http/https都可以")
            // 有参数的话添加参数，可多个
            .addParam("参数名", "参数值")
            .addParam("参数名", "参数值")
            // 也可以添加多个
            .addHeader("Content-Type", "application/json; charset=utf-8")
            .get()
            // 可选择是同步请求还是异步请求
            //.async();
            .sync();

    // post请求，分为两种，一种是普通表单提交，一种是json提交
    OkHttpUtils.builder().url("请求地址，http/https都可以")
            // 有参数的话添加参数，可多个
            .addParam("参数名", "参数值")
            .addParam("参数名", "参数值")
            // 也可以添加多个
            .addHeader("Content-Type", "application/json; charset=utf-8")
            // 如果是true的话，会类似于postman中post提交方式的raw，用json的方式提交，不是表单
            // 如果是false的话传统的表单提交
            .post(true)
            .sync();
    
    // 选择异步有两个方法，一个是带回调接口，一个是直接返回结果
    OkHttpUtils.builder().url("")
            .post(false)
            .async();

    OkHttpUtils.builder().url("").post(false).async(new OkHttpUtils.ICallBack() {
        @Override
        public void onSuccessful(Call call, String data) {
            // 请求成功后的处理
        }

        @Override
        public void onFailure(Call call, String errorMsg) {
            // 请求失败后的处理
        }
    });
}
```

## [**结语**](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247542384&idx=2&sn=4f1148b7fc0090760ba0e8ef4c299ac1&chksm=eb50a346dc272a50baca4f3f935323612c0ce4283599e87550276336dcc79b9bd6bde2f627df&scene=21#wechat_redirect)

[封装的明明白白，使用的简简单单，简单的几下就能做请求，用建造者模式是真的舒服。](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247542384&idx=2&sn=4f1148b7fc0090760ba0e8ef4c299ac1&chksm=eb50a346dc272a50baca4f3f935323612c0ce4283599e87550276336dcc79b9bd6bde2f627df&scene=21#wechat_redirect)

原文链接：https://blog.csdn.net/m0\_37701381/article/details/103386345

版权声明：本文为CSDN博主「如漩涡」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。