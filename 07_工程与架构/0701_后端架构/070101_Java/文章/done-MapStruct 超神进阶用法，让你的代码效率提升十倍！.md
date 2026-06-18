> 已吸收至：[[07_工程与架构/0701_后端架构/070101_Java/070101_核心知识点/Java来源校准与降权准则|Java来源校准与降权准则]]、[[07_工程与架构/0701_后端架构/070101_Java/070101_知识地图|070101_Java知识地图]]

---
title: MapStruct 超神进阶用法，让你的代码效率提升十倍！
author: 架构文摘
date:
url: http://mp.weixin.qq.com/s?__biz=MzkyMzcxODg0MQ==&mid=2247531896&idx=1&sn=dfb3709c7b592c155621b6fb497d67b3&chksm=c0a4b7b9997123f8b98438a6a5944edc65289f51d0deadef114f2a97012c89edcc50377d7076&mpshare=1&scene=24&srcid=0530UU7jOzu259Swlz1AKFvq&sharer_shareinfo=562589857550ecab93c5e99f52bbda5f&sharer_shareinfo_first=562589857550ecab93c5e99f52bbda5f#rd
---

## 来源：网络 ***后台点击菜单“学习资料”—“书籍”**，**免费**领取**《程序员书籍资料一份》*** ***后台回复“******5000******”，免费**领取****面******试技术学习资料****一份* **前言**

MapStruct 是一个 Java 编译时注解处理框架，用来自动化将一种 Java Bean 对象映射成另一种类型的对象。

该框架的主要目标是使开发人员在尽可能少的代码和最低的运行时间成本下实现属性映射。MapStruct 通过在编译时生成代码来实现这点，这与大多数其他 Java Bean 映射框架在运行时通过反射进行映射形成了鲜明对比。

MapStruct 具有以下主要特性：

* **简洁：** 简化了 Java Beans 之间转换的代码，自动生成使用简单的赋值语句完成的映射实现。
* **性能优秀：** 由于 MapStruct 是在编译时生成代码，不涉及任何反射，因此执行映射的性能优越。
* **安全：** 通过在编译时生成映射代码，MapStruct 提供了类型安全的映射，并能在编译时就发现潜在的错误。
* **灵活：** 可通过自定义转换方法、类型转换和映射策略等来满足复杂的映射需求。
* **良好的 IDE 支持：** 由于 MapStruct 是编译时工具，所以拥有良好的 IDE 集成，如代码自动完成、错误高亮等。

总的来说， MapStruct 是一个强大且灵活的映射框架，很好的解决有关对象转换的问题，实现了代码的简洁和性能的兼顾。MapStruct的常规用法，网上有很多教程了，本文将列举一些进阶用法，方便日常开发使用。

## **expression**

在转化的时候，执行 java 表达式，直接看例子：

```
@Mapper(componentModel = "spring")public interface MyMapper {    @Mapping(target = "createTime", expression = "java(System.currentTimeMillis())")    Target toTarget(Source source);}
```

转化成 target 对象时，createTime字段的值，会设置为`System.currentTimeMillis()`，生成的代码如下：

```
@Componentpublic class MyMapperImpl implements MyMapper {    @Override    public Target toTarget(Source source) {        Target target = new Target();        target.setCreateTime( System.currentTimeMillis() );        return target;    }}
```

## **qualifiedByName**

做映射时，默认情况下，从source 字段到target 字段是直接使用 `get/set`，如下：

```
@Datapublicclass Source {    private String name;}@Datapublicclass Target {    private String name;}    @Mapper(componentModel = "spring")publicinterface MyMapper {    Target toTarget(Source source);}
```

生成的转化代码类如下：

```
@Componentpublicclass MyMapperImpl implements MyMapper {    @Override    public Target toTarget(Source source) {        if ( source == null ) {            returnnull;        }        Target target = new Target();        // 无脑 set/get        target.setName( source.getName() );        return target;    }}
```

如果这种直接的 `set/get` 无法满足需求，比如需要把 name 转化成大写格式，那么可以使用`qualifiedByName`:

```
@Mapper(componentModel = "spring")public interface MyMapper {    @Mapping(target = "name", source = "name", qualifiedByName = "toUpperCase")    Target toTarget(Source source);    @Named("toUpperCase")    default String toUpperCase(String value) {        // 这里写转换大写的逻辑        return value == null ? null : value.toUpperCase();    }}
```

生成的代码如下：

```
@Componentpublicclass MyMapperImpl implements MyMapper {    @Override    public Target toTarget(Source source) {        if ( source == null ) {            returnnull;        }        Target target = new Target();        target.setName( toUpperCase( source.getName() ) );        return target;    }}
```

## **nullValueMappingStrategy**

如果 source 为 null 时，对应的 target 的处理策略，默认是 `NullValueMappingStrategy.RETURN_NULL`，即 target 中对应的字段也设置为 null。

但有时候设置为 null 可能不符合我们的需求，关注工众号：码猿技术专栏，回复关键词：1111 获取阿里内部Java性能调优手册！比如 target 中有个 List ids，我们希望如果 source 中ids 为 null 时，target 的 ids 设置为空 list。这时候可以使用`nullValueMappingStrategy`策略中的`NullValueMappingStrategy.RETURN_DEFAULT`。

`nullValueMappingStrategy` 可以使用在某个方法上（只对该方法生效），也可以使用在类上（对类中的所有方法都生效），如下：

```
@Componentpublicclass MyMapperImpl implements MyMapper {    @Override    public Target toTarget(Source source) {        if ( source == null ) {            returnnull;        }        Target target = new Target();        target.setName( source.getName() );        List<Integer> list = source.getIds();        if ( list != null ) {            target.setIds( new ArrayList<Integer>( list ) );        }        else {            target.setIds( null );        }        return target;    }}
```

指定`NullValueMappingStrategy.RETURN_DEFAULT`策略后：

```
@Mapper(componentModel = "spring",        nullValueMappingStrategy = org.mapstruct.NullValueMappingStrategy.RETURN_DEFAULT)publicinterface MyMapper {    Target toTarget(Source source);}@Componentpublicclass MyMapperImpl implements MyMapper {    @Override    public Target toTarget(Source source) {        Target target = new Target();        if ( source != null ) {            target.setName( toUpperCase( source.getName() ) );            List<Integer> list = source.getIds();            if ( list != null ) {                target.setIds( new ArrayList<Integer>( list ) );            }            else {                target.setIds( new ArrayList<Integer>() );            }        }        return target;    }}
```

可以看到，当 source 或者 `source.ids` 为 null 时，返回的 target 和 `target.ids` 都是默认的空值（空对象+空 list）。

## **Decorator**

你可以通过创建一个 `Decorator` 类来对你的方法进行修饰并实现全局处理。

以下是一个例子：

```
public abstractclass YourMapperDecorator implements YourMapper {    privatefinal YourMapper delegate;    public YourMapperDecorator(YourMapper delegate) {        this.delegate = delegate;    }    @Override    public Target toTarget(Source source) {        Target result = delegate.toTarget(source);        if (result != null) {            if (result.getField() == null) {                result.setField("");            }            // 你可以在这里对其他字段进行同样的处理...        }        return result;    }}
```

然后你只需在你的 Mapper 接口上添加 `@DecoratedWith` 注解：

```
@Mapper@DecoratedWith(YourMapperDecorator.class)public interface YourMapper {    Target toTarget(Source source);}
```

这样，每次调用 `toTarget` 方法时，`YourMapperDecorator` 中的实现会被调用。在这里，你可以实现任何你想要的逻辑，例如对空字段赋予特定的默认值。

```
福利：

扫码回复【图书】可免费领取图书管理系统源码

往期内容：

MySQL 8.2 支持读写分离！

2025-05-29

升级了 ！Spring 6.0 + Boot 3.0，性能太强了！

2025-05-28

再见VMware，一款更轻量级的虚拟机！

2025-05-27

Java 中 JSON 字段不固定怎么搞序列化？用好这两个注解就够了！

2025-05-25

8个让你直呼卧槽的 Docker 神器，让你的服务器瞬间开挂！

2025-05-23

— EOF —
```