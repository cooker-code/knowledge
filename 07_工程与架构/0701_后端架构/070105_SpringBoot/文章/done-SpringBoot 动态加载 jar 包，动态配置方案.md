> 已吸收至：[[07_工程与架构/0701_后端架构/070105_SpringBoot/070105_核心知识点/SpringBoot来源校准与扩展边界|SpringBoot来源校准与扩展边界]]、[[07_工程与架构/0701_后端架构/070105_SpringBoot/070105_知识地图|070105_SpringBoot知识地图]]

---
title: SpringBoot 动态加载 jar 包，动态配置方案
author: 小哈学Java
date:
url: http://mp.weixin.qq.com/s?__biz=MzU4MDUyMDQyNQ==&mid=2247535110&idx=2&sn=b386277b7e9b2f0f610936ff27ae565c&chksm=fd579a80ca201396d97681c5f1443623be3b66880a0fd67ea27b29d0e23bdfd77c777788a702&mpshare=1&scene=24&srcid=0509sdq6IwYv5WrmprKiCs2C&sharer_shareinfo=6ac73be9685825ad3bbf813e1565ed42&sharer_shareinfo_first=6ac73be9685825ad3bbf813e1565ed42#rd
---

来源：网络

> 👉 欢迎[加入小哈的星球](https://mp.weixin.qq.com/s?__biz=MzU4MDUyMDQyNQ==&mid=2247520263&idx=1&sn=62c4b0a4e307f177ea76eb93f469c0b7&chksm=fd574081ca20c99768c6f513eb4d373281b03d0051b6cc439b079611e64a9b958a6d175f1a39&token=981697568&lang=zh_CN&scene=21#wechat_redirect) ，你将获得: **专属的项目实战 / Java 学习路线 / 一对一提问 / 学习打卡 /  赠书福利**
>
> **新项目:**仿小红书**(微服务架构)正在更新中... , 全栈前后端分离博客项目 2.0 版本完结啦， **演示链接**：**http://116.62.199.48/ 。全程手摸手，后端 + 前端全栈开发，从 0 到 1 讲解每个功能点开发步骤，1v1 答疑，直到项目上线。**目前已更新了255小节，累计40w+字，讲解图：1716张，还在持续爆肝中..** 后续还会上新更多项目，目标是将Java领域典型的项目都整一波，如秒杀系统, 在线商城, IM即时通讯，Spring Cloud Alibaba 等等，[戳我加入学习，已有1300+小伙伴加入(早鸟价超低)](https://mp.weixin.qq.com/s?__biz=MzU4MDUyMDQyNQ==&mid=2247522465&idx=1&sn=49da64cae0a62b0c2beac8e0ad2b420e&chksm=fd574827ca20c1314c036377d7698e1fe12a7e716a15a0cfb56d5870f209c3dded22dd848d26&token=621118242&lang=zh_CN&scene=21#wechat_redirect)

* 一、概述

+ 1、背景
+ 2、目标
+ 3、方案

* 二、动态加载

+ 1、自定义类加载器
+ 2、动态加载
+ 3、动态卸载
+ 4、动态配置

* 三、分离打包

---

## **一、概述**

### **1、背景**

目前数据治理服务中有众多治理任务，当其中任一治理任务有改动需要升级或新增一个治理任务时，都需要将数据治理服务重启，会影响其他治理任务的正常运行。

### **2、目标**

* 能够动态启动、停止任一治理任务
* 能够动态升级、添加治理任务
* 启动、停止治理任务或升级、添加治理任务不能影响其他任务

### **3、方案**

* 为了支持业务代码尽量的解耦，把部分业务功能通过动态加载的方式加载到主程序中，以满足可插拔式的加载、组合式的部署。
* 配合xxl-job任务调度框架，将数据治理任务做成xxl-job任务的方式注册到xxl-job中，方便统一管理。

## **二、动态加载**

### **1、自定义类加载器**

URLClassLoader 是一种特殊的类加载器，可以从指定的 URL 中加载类和资源。它的主要作用是动态加载外部的 JAR 包或者类文件，从而实现动态扩展应用程序的功。为了便于管理动态加载的jar包，自定义类加载器继承URLClassloader。

```
package cn.jy.sjzl.util;

import java.lang.reflect.Method;
import java.net.URL;
import java.net.URLClassLoader;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

/**
 * 自定义类加载器
 *
 * @author lijianyu
 * @date 2023/04/03 17:54
 **/
public class MyClassLoader extends URLClassLoader {

    private Map<String, Class<?>> loadedClasses = new ConcurrentHashMap<>();

    public Map<String, Class<?>> getLoadedClasses() {
        return loadedClasses;
    }

    public MyClassLoader(URL[] urls, ClassLoader parent) {
        super(urls, parent);
    }

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        // 从已加载的类集合中获取指定名称的类
        Class<?> clazz = loadedClasses.get(name);
        if (clazz != null) {
            return clazz;
        }
        try {
            // 调用父类的findClass方法加载指定名称的类
            clazz = super.findClass(name);
            // 将加载的类添加到已加载的类集合中
            loadedClasses.put(name, clazz);
            return clazz;
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
            return null;
        }
    }

    public void unload() {
        try {
            for (Map.Entry<String, Class<?>> entry : loadedClasses.entrySet()) {
                // 从已加载的类集合中移除该类
                String className = entry.getKey();
                loadedClasses.remove(className);
                try{
                    // 调用该类的destory方法，回收资源
                    Class<?> clazz = entry.getValue();
                    Method destory = clazz.getDeclaredMethod("destory");
                    destory.invoke(clazz);
                } catch (Exception e ) {
                    // 表明该类没有destory方法
                }
            }
            // 从其父类加载器的加载器层次结构中移除该类加载器
            close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

* 自定义类加载器中，为了方便类的卸载，定义一个map保存已加载的类信息。key为这个类的ClassName，value为这个类的类信息。
* 同时定义了类加载器的卸载方法，卸载方法中，将已加载的类的集合中移除该类。由于此类可能使用系统资源或调用线程，为了避免资源未回收引起的内存溢出，通过反射调用这个类中的destroy方法，回收资源。
* 最后调用close方法。

### **2、动态加载**

由于此项目使用spring框架，以及xxl-job任务的机制调用动态加载的代码，因此要完成以下内容

* 将动态加载的jar包读到内存中
* 将有spring注解的类，通过注解扫描的方式，扫描并手动添加到spring容器中。
* 将@XxlJob注解的方法，通过注解扫描的方式，手动添加到xxljob执行器中。

```
package com.jy.dynamicLoad;

import com.jy.annotation.XxlJobCron;
import com.jy.classLoader.MyClassLoader;
import com.xxl.job.core.executor.impl.XxlJobSpringExecutor;
import com.xxl.job.core.handler.annotation.XxlJob;
import com.xxl.job.core.handler.impl.MethodJobHandler;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.beans.factory.support.AbstractBeanDefinition;
import org.springframework.beans.factory.support.BeanDefinitionBuilder;
import org.springframework.beans.factory.support.DefaultListableBeanFactory;
import org.springframework.context.ApplicationContext;
import org.springframework.core.MethodIntrospector;
import org.springframework.core.annotation.AnnotatedElementUtils;
import org.springframework.stereotype.Component;

import java.io.File;
import java.io.IOException;
import java.lang.reflect.Method;
import java.net.JarURLConnection;
import java.net.URL;
import java.net.URLConnection;
import java.util.Enumeration;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.jar.JarEntry;
import java.util.jar.JarFile;

/**
 * @author lijianyu
 * @date 2023/04/29 13:18
 **/
@Component
public class DynamicLoad {

    private static Logger logger = LoggerFactory.getLogger(DynamicLoad.class);

    @Autowired
    private ApplicationContext applicationContext;

    private Map<String, MyClassLoader> myClassLoaderCenter = new ConcurrentHashMap<>();

    @Value("${dynamicLoad.path}")
    private String path;

    /**
     * 动态加载指定路径下指定jar包
     * @param path
     * @param fileName
     * @param isRegistXxlJob  是否需要注册xxljob执行器，项目首次启动不需要注册执行器
     * @return map<jobHander, Cron> 创建xxljob任务时需要的参数配置
     */
    public void loadJar(String path, String fileName, Boolean isRegistXxlJob) throws ClassNotFoundException, InstantiationException, IllegalAccessException {
        File file = new File(path +"/" + fileName);
        Map<String, String> jobPar = new HashMap<>();
        // 获取beanFactory
        DefaultListableBeanFactory beanFactory = (DefaultListableBeanFactory) applicationContext.getAutowireCapableBeanFactory();
        // 获取当前项目的执行器
        try {
            // URLClassloader加载jar包规范必须这么写
            URL url = new URL("jar:file:" + file.getAbsolutePath() + "!/");
            URLConnection urlConnection = url.openConnection();
            JarURLConnection jarURLConnection = (JarURLConnection)urlConnection;
            // 获取jar文件
            JarFile jarFile = jarURLConnection.getJarFile();
            Enumeration<JarEntry> entries = jarFile.entries();

            // 创建自定义类加载器，并加到map中方便管理
            MyClassLoader myClassloader = new MyClassLoader(new URL[] { url }, ClassLoader.getSystemClassLoader());
            myClassLoaderCenter.put(fileName, myClassloader);
            Set<Class> initBeanClass = new HashSet<>(jarFile.size());
            // 遍历文件
            while (entries.hasMoreElements()) {
                JarEntry jarEntry = entries.nextElement();
                if (jarEntry.getName().endsWith(".class")) {
                    // 1. 加载类到jvm中
                    // 获取类的全路径名
                    String className = jarEntry.getName().replace('/', '.').substring(0, jarEntry.getName().length() - 6);
                    // 1.1进行反射获取
                    myClassloader.loadClass(className);
                }
            }
            Map<String, Class<?>> loadedClasses = myClassloader.getLoadedClasses();
            XxlJobSpringExecutor xxlJobExecutor = new XxlJobSpringExecutor();
            for(Map.Entry<String, Class<?>> entry : loadedClasses.entrySet()){
                String className = entry.getKey();
                Class<?> clazz = entry.getValue();
                // 2. 将有@spring注解的类交给spring管理
                // 2.1 判断是否注入spring
                Boolean flag = SpringAnnotationUtils.hasSpringAnnotation(clazz);
                if(flag){
                    // 2.2交给spring管理
                    BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition(clazz);
                    AbstractBeanDefinition beanDefinition = builder.getBeanDefinition();
                    // 此处beanName使用全路径名是为了防止beanName重复
                    String packageName = className.substring(0, className.lastIndexOf(".") + 1);
                    String beanName = className.substring(className.lastIndexOf(".") + 1);
                    beanName = packageName + beanName.substring(0, 1).toLowerCase() + beanName.substring(1);
                    // 2.3注册到spring的beanFactory中
                    beanFactory.registerBeanDefinition(beanName, beanDefinition);
                    // 2.4允许注入和反向注入
                    beanFactory.autowireBean(clazz);
                    beanFactory.initializeBean(clazz, beanName);
                    /*if(Arrays.stream(clazz.getInterfaces()).collect(Collectors.toSet()).contains(InitializingBean.class)){
                        initBeanClass.add(clazz);
                    }*/
                    initBeanClass.add(clazz);
                }

                // 3. 带有XxlJob注解的方法注册任务
                // 3.1 过滤方法
                Map<Method, XxlJob> annotatedMethods = null;
                try {
                    annotatedMethods = MethodIntrospector.selectMethods(clazz,
                            new MethodIntrospector.MetadataLookup<XxlJob>() {
                                @Override
                                public XxlJob inspect(Method method) {
                                    return AnnotatedElementUtils.findMergedAnnotation(method, XxlJob.class);
                                }
                            });
                } catch (Throwable ex) {
                }
                // 3.2 生成并注册方法的JobHander
                for (Map.Entry<Method, XxlJob> methodXxlJobEntry : annotatedMethods.entrySet()) {
                    Method executeMethod = methodXxlJobEntry.getKey();
                    // 获取jobHander和Cron
                    XxlJobCron xxlJobCron = executeMethod.getAnnotation(XxlJobCron.class);
                    if(xxlJobCron == null){
                        throw new CustomException("500", executeMethod.getName() + "()，没有添加@XxlJobCron注解配置定时策略");
                    }
                    if (!CronExpression.isValidExpression(xxlJobCron.value())) {
                        throw new CustomException("500", executeMethod.getName() + "(),@XxlJobCron参数内容错误");
                    }
                    XxlJob xxlJob = methodXxlJobEntry.getValue();
                    jobPar.put(xxlJob.value(), xxlJobCron.value());
                    if (isRegistXxlJob) {
                        executeMethod.setAccessible(true);
                        // regist
                        Method initMethod = null;
                        Method destroyMethod = null;
                        xxlJobExecutor.registJobHandler(xxlJob.value(), new CustomerMethodJobHandler(clazz, executeMethod, initMethod, destroyMethod));
                    }
                }

            }
            // spring bean实际注册
            initBeanClass.forEach(beanFactory::getBean);
        } catch (IOException e) {
            logger.error("读取{} 文件异常", fileName);
            e.printStackTrace();
            throw new RuntimeException("读取jar文件异常: " + fileName);
        }
    }
}
```

以下是判断该类是否有spring注解的工具类

```
apublic class SpringAnnotationUtils {

    private static Logger logger = LoggerFactory.getLogger(SpringAnnotationUtils.class);
    /**
     * 判断一个类是否有 Spring 核心注解
     *
     * @param clazz 要检查的类
     * @return true 如果该类上添加了相应的 Spring 注解；否则返回 false
     */
    public static boolean hasSpringAnnotation(Class<?> clazz) {
        if (clazz == null) {
            return false;
        }
        //是否是接口
        if (clazz.isInterface()) {
            return false;
        }
        //是否是抽象类
        if (Modifier.isAbstract(clazz.getModifiers())) {
            return false;
        }

        try {
            if (clazz.getAnnotation(Component.class) != null ||
            clazz.getAnnotation(Repository.class) != null ||
            clazz.getAnnotation(Service.class) != null ||
            clazz.getAnnotation(Controller.class) != null ||
            clazz.getAnnotation(Configuration.class) != null) {
                return true;
            }
        }catch (Exception e){
            logger.error("出现异常：{}",e.getMessage());
        }
        return false;
    }
}
```

注册xxljob执行器的操作是仿照的xxljob中的XxlJobSpringExecutor的注册方法。

### **3、动态卸载**

动态卸载的过程，就是将动态加载的代码，从内存，spring以及xxljob中移除。

代码如下：

```
/**
 * 动态卸载指定路径下指定jar包
 * @param fileName
 * @return map<jobHander, Cron> 创建xxljob任务时需要的参数配置
 */
public void unloadJar(String fileName) throws IllegalAccessException, NoSuchFieldException {
    // 获取加载当前jar的类加载器
    MyClassLoader myClassLoader = myClassLoaderCenter.get(fileName);

    // 获取jobHandlerRepository私有属性,为了卸载xxljob任务
    Field privateField = XxlJobExecutor.class.getDeclaredField("jobHandlerRepository");
    // 设置私有属性可访问
    privateField.setAccessible(true);
    // 获取私有属性的值jobHandlerRepository
    XxlJobExecutor xxlJobSpringExecutor = new XxlJobSpringExecutor();
    Map<String, IJobHandler> jobHandlerRepository = (ConcurrentHashMap<String, IJobHandler>) privateField.get(xxlJobSpringExecutor);
    // 获取beanFactory，准备从spring中卸载
    DefaultListableBeanFactory beanFactory = (DefaultListableBeanFactory) applicationContext.getAutowireCapableBeanFactory();
    Map<String, Class<?>> loadedClasses = myClassLoader.getLoadedClasses();

    Set<String> beanNames = new HashSet<>();
    for (Map.Entry<String, Class<?>> entry: loadedClasses.entrySet()) {
        // 1. 将xxljob任务从xxljob执行器中移除
        // 1.1 截取beanName
        String key = entry.getKey();
        String packageName = key.substring(0, key.lastIndexOf(".") + 1);
        String beanName = key.substring(key.lastIndexOf(".") + 1);
        beanName = packageName + beanName.substring(0, 1).toLowerCase() + beanName.substring(1);

        // 获取bean，如果获取失败，表名这个类没有加到spring容器中，则跳出本次循环
        Object bean = null;
        try{
            bean = applicationContext.getBean(beanName);
        }catch (Exception e){
            // 异常说明spring中没有这个bean
            continue;
        }

        // 1.2 过滤方法
        Map<Method, XxlJob> annotatedMethods = null;
        try {
            annotatedMethods = MethodIntrospector.selectMethods(bean.getClass(),
                    new MethodIntrospector.MetadataLookup<XxlJob>() {
                        @Override
                        public XxlJob inspect(Method method) {
                            return AnnotatedElementUtils.findMergedAnnotation(method, XxlJob.class);
                        }
                    });
        } catch (Throwable ex) {
        }
        // 1.3 将job从执行器中移除
        for (Map.Entry<Method, XxlJob> methodXxlJobEntry : annotatedMethods.entrySet()) {
            XxlJob xxlJob = methodXxlJobEntry.getValue();
            jobHandlerRepository.remove(xxlJob.value());
        }
        // 2.0从spring中移除，这里的移除是仅仅移除的bean，并未移除bean定义
        beanNames.add(beanName);
        beanFactory.destroyBean(beanName, bean);
    }
    // 移除bean定义
    Field mergedBeanDefinitions = beanFactory.getClass()
            .getSuperclass()
            .getSuperclass().getDeclaredField("mergedBeanDefinitions");
    mergedBeanDefinitions.setAccessible(true);
    Map<String, RootBeanDefinition> rootBeanDefinitionMap = ((Map<String, RootBeanDefinition>) mergedBeanDefinitions.get(beanFactory));
    for (String beanName : beanNames) {
        beanFactory.removeBeanDefinition(beanName);
        // 父类bean定义去除
        rootBeanDefinitionMap.remove(beanName);
    }

    // 卸载父任务，子任务已经在循环中卸载
    jobHandlerRepository.remove(fileName);
    // 3.2 从类加载中移除
    try {
        // 从类加载器底层的classes中移除连接
        Field field = ClassLoader.class.getDeclaredField("classes");
        field.setAccessible(true);
        Vector<Class<?>> classes = (Vector<Class<?>>) field.get(myClassLoader);
        classes.removeAllElements();
        // 移除类加载器的引用
        myClassLoaderCenter.remove(fileName);
        // 卸载类加载器
        myClassLoader.unload();
    } catch (NoSuchFieldException e) {
        logger.error("动态卸载的类，从类加载器中卸载失败");
        e.printStackTrace();
    } catch (IllegalAccessException e) {
        logger.error("动态卸载的类，从类加载器中卸载失败");
        e.printStackTrace();
    }
    logger.error("{} 动态卸载成功", fileName);

}
```

### **4、动态配置**

使用动态加载时，为了避免服务重新启动后丢失已加载的任务包，使用动态配置的方式，加载后动态更新初始化加载配置。

以下提供了两种自己实际操作过的配置方式。

#### **4.1 动态修改本地yml**

动态修改本地yml配置文件，需要添加snakeyaml的依赖

**4.1.1 依赖引入**

```
<dependency>
 <groupId>org.yaml</groupId>
    <artifactId>snakeyaml</artifactId>
    <version>1.29</version>
</dependency>
```

**4.1.2 工具类**

读取指定路径下的配置文件，并进行修改。

```
package com.jy.util;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.env.Environment;
import org.springframework.stereotype.Component;
import org.yaml.snakeyaml.DumperOptions;
import org.yaml.snakeyaml.Yaml;

import java.io.*;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

/**
 * 用于动态修改bootstrap.yml配置文件
 * @author lijianyu
 * @date 2023/04/18 17:57
 **/
@Component
public class ConfigUpdater {

    public void updateLoadJars(List<String> jarNames) throws IOException {
        // 读取bootstrap.yml
        Yaml yaml = new Yaml();
        InputStream inputStream = new FileInputStream(new File("src/main/resources/bootstrap.yml"));
        Map<String, Object> obj = yaml.load(inputStream);
        inputStream.close();

        obj.put("loadjars", jarNames);

        // 修改
        FileWriter writer = new FileWriter(new File("src/main/resources/bootstrap.yml"));
        DumperOptions options = new DumperOptions();
        options.setDefaultFlowStyle(DumperOptions.FlowStyle.BLOCK);
        options.setPrettyFlow(true);
        Yaml yamlWriter = new Yaml(options);
        yamlWriter.dump(obj, writer);
    }
}
```

#### **4.2 动态修改nacos配置**

Spring Cloud Alibaba Nacos组件完全支持在运行时通过代码动态修改配置，还提供了一些API供开发者在代码里面实现动态修改配置。在每次动态加载或卸载数据治理任务jar包时，执行成功后都会进行动态更新nacos配置。

```
package cn.jy.sjzl.config;

import com.alibaba.nacos.api.NacosFactory;
import com.alibaba.nacos.api.config.ConfigService;
import com.alibaba.nacos.api.exception.NacosException;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Configuration;

import java.util.Properties;

@Configuration
public class NacosConfig {
    @Value("${spring.cloud.nacos.server-addr}")
    private String serverAddr;

    @Value("${spring.cloud.nacos.config.namespace}")
    private String namespace;

    public ConfigService configService() throws NacosException {
        Properties properties = new Properties();
        properties.put("serverAddr", serverAddr);
        properties.put("namespace", namespace);
        return NacosFactory.createConfigService(properties);
    }
}
package cn.jy.sjzl.util;

import cn.jy.sjzl.config.NacosConfig;
import com.alibaba.fastjson.JSONObject;
import com.alibaba.nacos.api.config.ConfigService;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.dataformat.yaml.YAMLMapper;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.List;
import java.util.stream.Collectors;

/**
 * nacos配置中，修改sjzl-loadjars.yml
 *
 * @author lijianyu
 * @date 2023/04/19 17:59
 **/
@Component
public class NacosConfigUtil {

    private static Logger logger = LoggerFactory.getLogger(NacosConfigUtil.class);

    @Autowired
    private NacosConfig nacosConfig;

    private String dataId = "sjzl-loadjars.yml";

    @Value("${spring.cloud.nacos.config.group}")
    private String group;

    /**
     * 从nacos配置文件中，添加初始化jar包配置
     * @param jarName 要移除的jar包名
     * @throws Exception
     */
    public void addJarName(String jarName) throws Exception {
        ConfigService configService = nacosConfig.configService();
        String content = configService.getConfig(dataId, group, 5000);
        // 修改配置文件内容
        YAMLMapper yamlMapper = new YAMLMapper();
        ObjectMapper jsonMapper = new ObjectMapper();
        Object yamlObject = yamlMapper.readValue(content, Object.class);

        String jsonString = jsonMapper.writeValueAsString(yamlObject);
        JSONObject jsonObject = JSONObject.parseObject(jsonString);
        List<String> loadjars;
        if (jsonObject.containsKey("loadjars")) {
            loadjars = (List<String>) jsonObject.get("loadjars");
        }else{
            loadjars = new ArrayList<>();
        }
        if (!loadjars.contains(jarName)) {
            loadjars.add(jarName);
        }
        jsonObject.put("loadjars" , loadjars);

        Object yaml = yamlMapper.readValue(jsonMapper.writeValueAsString(jsonObject), Object.class);
        String newYamlString = yamlMapper.writeValueAsString(yaml);
        boolean b = configService.publishConfig(dataId, group, newYamlString);

        if(b){
            logger.info("nacos配置更新成功");
        }else{
            logger.info("nacos配置更新失败");
        }
    }
}
```

## **三、分离打包**

分离打包时，根据实际情况在pom.xml中修改以下配置

```
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>3.2.4</version>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                    <configuration>
                        <filters>
                            <filter>
                                <artifact>*:*</artifact>
                                <includes>
                                    <include>com/jy/job/demo/**</include>
                                </includes>
                            </filter>
                        </filters>
                        <finalName>demoJob</finalName>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

> 👉 欢迎[加入小哈的星球](https://mp.weixin.qq.com/s?__biz=MzU4MDUyMDQyNQ==&mid=2247520263&idx=1&sn=62c4b0a4e307f177ea76eb93f469c0b7&chksm=fd574081ca20c99768c6f513eb4d373281b03d0051b6cc439b079611e64a9b958a6d175f1a39&token=981697568&lang=zh_CN&scene=21#wechat_redirect) ，你将获得: **专属的项目实战 / Java 学习路线 / 一对一提问 / 学习打卡 /  赠书福利**
>
> **新项目:**仿小红书**(微服务架构)正在更新中... , 全栈前后端分离博客项目 2.0 版本完结啦， **演示链接**：**http://116.62.199.48/ 。全程手摸手，后端 + 前端全栈开发，从 0 到 1 讲解每个功能点开发步骤，1v1 答疑，直到项目上线。**目前已更新了255小节，累计40w+字，讲解图：1716张，还在持续爆肝中..** 后续还会上新更多项目，目标是将Java领域典型的项目都整一波，如秒杀系统, 在线商城, IM即时通讯，Spring Cloud Alibaba 等等，[戳我加入学习，已有1300+小伙伴加入(早鸟价超低)](https://mp.weixin.qq.com/s?__biz=MzU4MDUyMDQyNQ==&mid=2247522465&idx=1&sn=49da64cae0a62b0c2beac8e0ad2b420e&chksm=fd574827ca20c1314c036377d7698e1fe12a7e716a15a0cfb56d5870f209c3dded22dd848d26&token=621118242&lang=zh_CN&scene=21#wechat_redirect)

```
```
```
```
1. 我的私密学习小圈子~

2. 9.5k star，一款高颜值、现代化的 Git 可视化管理工具

3. SpringBoot+Redis+定时任务模拟手机短信验证

4. 如何优雅的实现接口统一调用
```
```
```
```

```
```
最近面试BAT，整理一份面试资料《Java面试BATJ通关手册》，覆盖了Java核心技术、JVM、Java并发、SSM、微服务、数据库、数据结构等等。

获取方式：点“在看”，关注公众号并回复 Java 领取，更多内容陆续奉上。
```

```
PS：因公众号平台更改了推送规则，如果不想错过内容，记得读完点一下“在看”，加个“星标”，这样每次新文章推送才会第一时间出现在你的订阅列表里。

点“在看”支持小哈呀，谢谢啦
```
```