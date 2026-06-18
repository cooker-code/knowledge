---
title: 推荐 2 个 Spring Boot 的工作流项目
author: AI工具迷
date: 
url: http://mp.weixin.qq.com/s?__biz=MzI4OTI2Nzg4Mg==&mid=2247498781&idx=1&sn=1e420a86ca2588425715f4395d641107&chksm=ec3375d8db44fcce0f5972db995d9fc760667ab2bd9dd7ed1e6a5066436fc3455d64d85c2701&mpshare=1&scene=24&srcid=0720cFtOXRlOc84bakJjSqBa&sharer_sharetime=1658272520494&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

今天主要推荐两个工作流的springboot项目，开源项目中有具体的部署操作文档，核心表结构说明，都可以帮助理解工作流原理，其实大厂华为阿里里面的工作流虽然号称自研（很多都是参考开源），跟开源工作流的原理差不多的。

工作流出问题比较高频的是配置出现低级问题，比如少一个符号或大小写不规范，别问我怎么知道的（玩工作流好几年了），工作流玩熟悉了，开发效率是极快的。

* Spring-boot-activiti
* RuoYi-vue 4.x + flowable

## **1、Spring-boot-activiti**

在常用的ERP系统、OA系统的开发中，工作流引擎是一个必不可少的工具。本项目旨在基于Spring boot这一平台，整合业界流行的工作流引擎Activiti，并建立了两个完整的工作流进行演示：请假OA和采购流程。

其中包含的内容如下：

1. 不采用activiti自带的用户、角色功能，因为过于简单，转而自行实现一个用户、角色、权限的三级结构，用户到角色，角色到权限均为多对多映射，持久层框架使用mybatis的collection和association标签嵌套实现；

2. 使用默认的用户登录后（用户名xiaomi，密码1234），可看到已部署好的两个流程，请假OA和采购流程，其中，请假OA包含了用户任务、排他网关、起始结束事件，较为简单；采购流程除此之外，还使用了异常结束事件、子流程和边界事件的使用；

3. 两个流程均包含了待办任务签收、运行流程进度追踪、已运行完流程历史记录查看的功能，运行流程进度在流程图中以红色标注；

4. 使用时，将流程数据和业务数据相分离，使用业务号（businessKey）建立关联流程数据和业务数据的桥梁，使其相互可以访问,业务数据的主键即为业务号；

5. 本系统所有表单均使用普通表单，而不是activiti的动态表单和外置表单，这样做是为了分表存放业务数据和流程数据;

6. 系统前端采用基于Bootstrap的模板devoops建立。

7. 起始页面入口：http://localhost:8888/login 使用前，编译(可直接在myeclipse完成)：

```
mvn clean install
```

8. swagger入口：http://localhost:8888/swagger-ui.html

9.新增流程图设计器，将activiti-explorer.war放入Tomcat8.5的webapps目录下，启动Tomcat，访问http://localhost:8080/activiti-explorer ，即可进入登录页面，用户名和密码都是kermit：

点击流程->流程设计工作区->新建模型，填入模型名称，点击创建按钮即可开始流程图设计：

设计完后，可以导出源文件：

10.新增动态菜单权限，通过给不同用户分配不同的角色，使他们看到不同的菜单。

11.请假流程直接分配到具体审批人，只有指定的审批人才能审批。采购流程按照候选者组进行任务签收，只要能拥有相关菜单的权限即可审批。

12.效果图：

**2、RuoYi-vue 4.x + flowable**

基 RuoYi-vue 4.x + flowable 6.5 的工作流管理 ~

一直想学习和入门flowable, 期间大量面向百度编程解决了很多问题, 感谢 flowable初级使用手册 ,零基础学习很值得一看。最后感谢 若依框架 ,快速集成开发爽到飞起 ~~~

**演示地址**

访问链接：http://139.155.16.243/

使用文档：https://www.yuque.com/u1024153/icipor

> 为了方便体验，请勿删除和改动初始化的几个流程和表单，感谢！有其它流程实现，请自行定义新流程、表单。

**项目**

* 前端采用Vue、Element UI。
* 后端采用Spring Boot、Spring Security、Redis & Jwt。
* 权限认证使用Jwt，支持多终端认证系统。
* 支持加载动态权限菜单，多方式轻松权限控制。
* 高效率开发，使用代码生成器可以一键生成前后端代码。

**内置功能**

* 流程设计
* 表单配置
* 流程发起
* 流转处理
* 参考文档

文档地址：http://doc.ruoyi.vip

**演示图**

**项目地址**

* https://gitee.com/shenzhanwang/Spring-activiti
* https://gitee.com/tony2y/RuoYi-flowable

```
```
```
```
```
```
```
```
```
```
```
其他优质好项目

推荐一款牛逼的接私活项目，微服务也能搞定！

推荐一套开源通用后台管理系统

基于 SpringBoot + MyBatis 前后端分离实现的在线办公系统

分享70套 Java 项目+实战课

Java项目精选读者群正式开发，先到先得 ！

```
欢迎添加编程君个人微信 cxycode666  进粉丝群或围观朋友
```
```
```

喜欢文章，点个在看
```
```
```
```
```
```
```
```
```