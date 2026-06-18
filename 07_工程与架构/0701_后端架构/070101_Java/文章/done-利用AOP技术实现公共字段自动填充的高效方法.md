> 已吸收至：[[07_工程与架构/0701_后端架构/070101_Java/070101_核心知识点/Java来源校准与降权准则|Java来源校准与降权准则]]、[[07_工程与架构/0701_后端架构/070101_Java/070101_知识地图|070101_Java知识地图]]

---
title: 利用AOP技术实现公共字段自动填充的高效方法
author: 业余草
date:
url: http://mp.weixin.qq.com/s?__biz=MzIyODE5NjUwNQ==&mid=2653370961&idx=1&sn=c687a6df25dc5fbe79694f4048e366a2&chksm=f38633a7c4f1bab122cf7658bdf24d48c4573dbe434e9516f6486e113e80d49bb693685fd2dc&mpshare=1&scene=24&srcid=0402dkUgjkCw13eYf9ZbNMe5&sharer_shareinfo=d46c4be08afaa68f92bf7171b3fee55a&sharer_shareinfo_first=d46c4be08afaa68f92bf7171b3fee55a#rd
---

## 公共字段自动填充

一般在表设计的时候，都会在表中添加一些系统字段，比如 create\_time、update\_time等。

这样以来，每个表中都有 create\_time、create\_user、update\_time、update\_user 4 个字段。我们经常调侃写 CRUD，你看，现在机会来了。这 4 个公共字段能否抽象出来进行统一处理？答案肯定是可以的。

今天我们花几分钟时间来简单讲一下非 MyBatis 的方式来实现公共字段自动填充。

### 问题分析

在进行新增员工、菜品功能时需要设置创建时间、更新时间、创建人、修改人字段，在编辑员工和菜品分类时需要设置修改时间、修改人的字段。这些字段属于公共字段，也就是很多表都会有的字段。如下：

| **序号** | **字段名** | **含义** | **数据类型** |
| --- | --- | --- | --- |
| 1 | create\_time | 创建时间 | datetime |
| 2 | create\_user | 创建人id | bigint |
| 3 | update\_time | 修改时间 | datetime |
| 4 | update\_user | 修改人id | bigint |

目前，在项目中这些字段都是在每一个业务方法中进行赋值，编码相对冗余、繁琐，那能不能对于这些公共字段在某个地方统一处理，来简化开发呢？

### 实现思路

在不改变原有业务代码，实现功能增强，就要使用AOP来实现功能增强，来完成公共字段填充功能。

### 分析

> ❝
>
> **先确定切入点：要增强谁**
>
> 使用excution，根据方法名选择
>
> 使用@annotation，使用注解选择
>
> 连接使用：连接符 && ||
>
> **确定什么时候增强：通知类型**
>
> @Before：前置通知，在目标方法执行之前先执行
>
> @AfterReturing：后置通知，在目标方法正常执行完成之后在执行
>
> @AfterThrowing：异常通知，在目标方法抛出异常之后在执行
>
> @After：最终通知，在目标方法执行之后，无论有没有异常都会执行
>
> @Around：环绕通知
>
> **确定要怎么增强，通知方法里要做的事情**

### 实现

> ❝
>
> **先确定切入点：要增强谁（切入点表达式）**
>
> Mapper里的`insert`和`update`方法，方法名可能并不规范，根据方法名选择切入点可能不合适。
>
> 为了更灵活的选中在增强的方法，使用注解的方式：哪个方法要增强，就在哪个方法上加注解 @注解(操作类型)
>
> **需要确定如何增强：通知**
>
> **通知类型：** 什么时候对Mapper的方法增强？在目标方法执行之前增强，使用@Before
>
> **如何增强：**
>
> 获取目标方法上的注解，得到其中的操作类型
>
> ***判断如果目标方法操作类型是insert新增：***
>
> 获取方法的实参entity对象
>
> 给entity对象设置createTime、updateTime、createUser、updateUser属性值
>
> ***如果目标方法操作类型是update修改：***
>
> 获取方法的实参entity对象
>
> 给entity对象设置updateTime、updateUser属性值

### 编码步骤

1. 先定义注解 @AutoFill，给注解添加一个参数是OperationType
2. 在需要增强的方法上添加注解 @AutoFill(OperationType.INSERT或UPDATE)
3. 编写切面类：

类上加注解@Component @Aspect

类里通过方法：

```
@Before("execution(选中mapper的方法) && @annotation(autoFill)")
public void autoFill(JoinPoint jp, AutoFill autoFill){  
        根据autoFill获取操作类型
        如果操作类型是INSERT：
                获取entity对象的createTime、updateTime、createUser、updateUser的set方法
                使用反射分别调用每个set方法，设置值
        如果操作类型是UPDATE：
                获取entity对象的updateTime、updateUser的set方法
                使用反射分别调用每个set方法，设置值
}
```

### 代码实现

1. **自定义注解@AutoFill**

```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface AutoFill {
    //数据库操作类型：UPDATE INSERT
    OperationType value();
}
```

**数据库操作类型**

```
public enum OperationType {

    /**
     * 更新操作
     */
    UPDATE,

    /**
     * 插入操作
     */
    INSERT
}
```

2. **在Mapper层 插入、修改方法添加注解@AutoFill**
3. **自定义切面AutoFillAspect**

```
@Aspect
@Component
public class AutoFillAspect {
    /*
    * 1. 在切入点表达式里：如果想要缩小spring的扫描范围，可以使用多切入点表达式连接 连接符 &&
    * 选择要增强的方法：
    *        只扫描mapper层里的方法 并且 有AutoFill注解的方法
    * 2. 在注解方式切入点的通知方法，如果想在通知方法里获取到注解对象可以写成：
    *   2.1 在方法上直接加注解类型的形参
    *   2.2 修改切入点表达式
    *       原来：@annotation(注解的全限定类名)
    *       改成：@annotation(方法的注解类型形参名)
    * 注意：如果通知方法有多个形参的话，那么JoinPoint类型的参数要放到第一个
    *
    *
    * Java里如果要调用一个方法，有两种方式
    *   方式1：对象名.方法名(实参)
    *   方式2：反射调用方法
    *       先获取目标类的字节码
    *       在获取想要调用的方法
    *       最后反射执行这个方法
    * */
    @Before("execution(* com.sky.mapper.*.*(..)) && @annotation(autoFill)")
    public void autoFill(JoinPoint joinPoint,AutoFill autoFill){//JoinPoint 得到要被增强的目标方法
        //获取目标方法的实参
        Object[] args = joinPoint.getArgs();
        // 为了提高程序的健壮性，加一个判断：如果方法没有实参，就什么都不做
        if(args == null || args.length == 0){
            return;
        }
        Object entity = args[0];
        Class clazz = entity.getClass();
        
        //如果目标方法操作类型是INSERT：设置四个值，否则是操作Update，设置两个值
        if(autoFill.value() == OperationType.INSERT){
            try {
                Method setCreateTime = clazz.getMethod("setCreateTime", LocalDateTime.class);
                Method setUpdateTime = clazz.getMethod("setUpdateTime", LocalDateTime.class);
                Method setCreateUser = clazz.getMethod("setCreateUser", Long.class);
                Method setUpdateUser = clazz.getMethod("setUpdateUser", Long.class);
                setUpdateUser.invoke(entity, BaseContext.getCurrentId());
                setUpdateTime.invoke(entity,LocalDateTime.now());
                setCreateUser.invoke(entity, BaseContext.getCurrentId());
                setCreateTime.invoke(entity,LocalDateTime.now());
            } catch (Exception e) {
                e.printStackTrace();
            }
        } else {
            try {
                Method setUpdateTime = clazz.getMethod("setUpdateTime", LocalDateTime.class);
                Method setUpdateUser = clazz.getMethod("setUpdateUser", Long.class);
                setUpdateUser.invoke(entity, BaseContext.getCurrentId());
                setUpdateTime.invoke(entity,LocalDateTime.now());
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```

当然如果是 Mybatis 或者是 MyBatis-Puls，则可以通过实现 MetaObjectHandler 接口等形式来实现公共字段的自动填充，大致代码如下。

```
@Component
public class CommonMetaObjectHandler implements MetaObjectHandler {
    /**
     * 创建时间
     */
    private static final String FIELD_SYS_CREATE_TIME = "createTime";
    /**
     * 修改时间
     */
    private static final String FIELD_SYS_MODIFIED_TIME = "updateTime";

    /**
     * 插入元对象字段填充（用于插入时对公共字段的填充）
     *
     * @param metaObject 元对象
     */
    @Override
    public void insertFill(MetaObject metaObject) {
        Date currentDate = new Date();
        // 插入创建时间
        if (metaObject.hasSetter(FIELD_SYS_CREATE_TIME)) {
            this.strictInsertFill(metaObject, FIELD_SYS_CREATE_TIME, Date.class, currentDate);
        }
        // 同时设置修改时间为当前插入时间
        if (metaObject.hasSetter(FIELD_SYS_MODIFIED_TIME)) {
            this.strictUpdateFill(metaObject, FIELD_SYS_MODIFIED_TIME, Date.class, currentDate);
        }
    }

    /**
     * 更新元对象字段填充（用于更新时对公共字段的填充）
     *
     * @param metaObject 元对象
     */
    @Override
    public void updateFill(MetaObject metaObject) {
        this.setFieldValByName(FIELD_SYS_MODIFIED_TIME, new Date(), metaObject);
    }
}
```

以上，期待大家活学活用！