---
title: Maven官宣：干掉Maven和Gradle！推出更强更快更牛逼的新一代构建工具，炸裂！
author: Java后端技术
date: 
url: http://mp.weixin.qq.com/s?__biz=MzI1NDQ3MjQxNA==&mid=2247506367&idx=1&sn=3d1e01ad7ba4032758dce0ae9546177c&chksm=e9c6200edeb1a9184437644e376d952aa499a377507354d7b708e9815128eb505174e7e93bd3&mpshare=1&scene=24&srcid=0612f6HTk3TehQ7LPIvvOBKX&sharer_sharetime=1686533061763&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

```
#### 往期热门文章：

#### 1、大公司为什么禁止SpringBoot项目使用Tomcat？ 2、快速交付神器：阿里巴巴官方低代码引擎开源了！ 3、为什么 Spring和IDEA 都不推荐使用 @Autowired 注解 4、程序员的悲哀是什么？ 5、被问懵了：MySQL 自增主键一定是连续的吗？
```

文章来源：

https://blog.csdn.net/weixin\_48321993/article/details/125979820

前言

相信作为Java开发者的你早已经受够了maven的编译缓慢，但是又由于历史包袱、使用习惯等问题暂时切换不了其他更快的构建工具，这里笔者将给你介绍一款更快的maven——maven-mvnd。

## 

> > **一.介绍**

maven-mvnd是Apache Maven团队借鉴了Gradle和Takari后衍生出的更快的构建工具。mvnd内嵌了Maven，也正是因为这个原因我们可以无缝地将Maven切换为mvnd（也不需要单独安装Maven）。

在设计上，在mvnd中会生成一个或多个的守护进程来服务构建请求以此来达到并行构建的效果。另外在VM的选择上，mvnd使用了GraalVM来代替传统的JVM，与之相比GraalVM启动速度更快，占用的内存更少。

根据文档描述，与传统的Maven相比mvnd具有以下优势：

* 运行构建的JVM不需要为每个构建重新启动。
* Maven插件类的类加载器缓存在多个构建中，插件jars只会被读取和解析一次。
* JVM中JIT生成的本机代码会被保留。与Maven相比，JIT编译花费的时间更少。在重复构建期间，JIT优化的代码立即可用。这不仅适用于来自Maven插件和Maven内核的代码，也适用于来自JDK本身的所有代码。

默认情况下，mvnd使用多个CPU内核并行构建模块。使用的内核数由公式Math.max(Runtime.getRuntime().availableProcessors() - 1, 1)给出。如果您的源代码树不支持并行构建，请在命令行上传递-T1以使您的构建串行。

同时官方给出了24核机器上运行的动态图：

> > **二.安装**

对于mvnd的安装，官方文档给了十分详细的教程，建议先行阅读：https://github.com/apache/maven-mvnd 。

笔者是通过Homebrew进行安装的，实践证明macOS m1安装使用是没有问题的。不过需要注意的是通过此种方式安装的mvnd版本为0.7.1，而经过在ubuntu和macOS m1上进行测试发现此版本并不支持JDK8(可能仅是笔者电脑问题)，而通过官方例子所示的JDK11确是没问题。在JDK8运行mvnd命令会产生以下错误：

```
~ % mvnd -v  
Error: A JNI error has occurred, please check your installation and try again  
Exception in thread "main" java.lang.UnsupportedClassVersionError: org/mvndaemon/mvnd/client/DefaultClient has been compiled by a more recent version of the Java Runtime (class file version 55.0), this version of the Java Runtime only recognizes class file versions up to 52.0  
at java.lang.ClassLoader.defineClass1(Native Method)  
at java.lang.ClassLoader.defineClass(ClassLoader.java:757)  
at java.security.SecureClassLoader.defineClass(SecureClassLoader.java:142)  
at java.net.URLClassLoader.defineClass(URLClassLoader.java:473)  
at java.net.URLClassLoader.access$100(URLClassLoader.java:74)  
at java.net.URLClassLoader$1.run(URLClassLoader.java:369)  
at java.net.URLClassLoader$1.run(URLClassLoader.java:363)  
at java.security.AccessController.doPrivileged(Native Method)  
at java.net.URLClassLoader.findClass(URLClassLoader.java:362)  
at java.lang.ClassLoader.loadClass(ClassLoader.java:419)  
at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:352)  
at java.lang.ClassLoader.loadClass(ClassLoader.java:352)
```

预估应该是这种方式下载的执行文件是通过高版本的JDK编译的，在低版本上运行因为缺少某些方法或特性所以运行不了。在一筹莫展之际，笔者从maven-mvnd的最新版本的更新说明上发现一个Closed issues:Different java versions for mvnd and maven #512，在该问题上作者提供了一种解决方案就是将JAVA\_HOME所指定的版本设置为JDK11，并且在运行mvnd命令时加上参数-Dmaven.compiler.release=8，即

```
mvnd -Dmaven.compiler.release=8 compile
```

通过这种方式即能生成出JDK8所对应的编译代码。

对于issue #512中作者回应mvnd的最低支持版本是JDK8，但是笔者从0.5.2开始尝试还是报同样的错...或许是笔者电脑存在一定的问题，因为我看到其他人在贴出的结果图显示JDK8下最新版本也是能够安装并使用的。另外如果还是不行的话也许我们能够通过手动编译源码来生成可执行文件，具体步骤在官方readme上已经给出相应的步骤。

> > **三.使用**

在使用上与Maven的用法是完全相同的，只需将命令mvn改为mvnd即可。而在笔者本机的实测中，与传统的Maven相比，通过mvnd的构建所耗费的时间是原来的1/2。

> > **四.总结**

本文笔者分别从maven-mvnd的介绍、安装、使用及其出现的一些异常情况展开陈述，如果读者想知道更多细节可阅读官方文档。而也许强化后的Maven依然比不过Gradle，但是在历史包袱、使用习惯等背景下Maven的这次强化还是很香的。

```
往期热门文章：

#### 1、点一下详情系统挂了，CPU100% 2、我说用count(*)统计行数，面试官让我回去等消息... 3、世界第三大浏览器正在消亡 4、被问懵了：MySQL 自增主键一定是连续的吗？ 5、FastJson 很好，但不适合我！ 6、告警：线上慎用 BigDecimal ，坑的差点被开了 7、哪有这么多从零项目给你开发 8、从微服务转为单体架构、成本降低 90%！是的，你没看反！ 9、Lombok 造成的翻车事故，太坑了！ 10、通用的支付系统该如何设计？
```