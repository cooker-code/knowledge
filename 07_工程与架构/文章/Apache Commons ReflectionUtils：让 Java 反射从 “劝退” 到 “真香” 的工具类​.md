---
title: Apache Commons ReflectionUtils：让 Java 反射从 “劝退” 到 “真香” 的工具类​
author: Java程序员-晴天Y
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzk1NzU1NDIzMA==&mid=2247484357&idx=1&sn=0544ec0c028442d948b1547a1bc6cd9d&chksm=c25c4784b45395a4f85a50737846527938f200b542701303cbfbc7cd781ea272e866a5be05f8&mpshare=1&scene=24&srcid=0909VUNZFiDHThyiP2NOpL82&sharer_shareinfo=76ee1027c6ba3541665edf8cef1db8b2&sharer_shareinfo_first=76ee1027c6ba3541665edf8cef1db8b2#rd
---

在 Java 开发中，原生反射 API 的繁琐程度却让不少开发者望而却步。获取 Class 对象、处理访问权限、捕获一堆异常…… 仅仅是调用一个简单的方法，就要写十几行模板代码。我猜你肯定不想这么操作！

你肯定在想要是有一个工具类能帮我做这些事情就好了。看我表情就知道有没有。

Apache Commons 提供的 ReflectionUtils 工具类，正是为了拯救被反射折磨的开发者。它将反射操作的复杂逻辑封装成直观的静态方法，用一行代码替代多行冗余代码，让反射从 “劝退” 级难度变成 “真香” 级体验。下面咱们就来一起解锁这个神器！

一、原生反射的 “劝退” 瞬间

先看一个典型场景：通过反射调用 User 类的 setName 方法。用 JDK 原生反射实现会是这样：

```
public class User {    private String name;    private int age;    public User() {}    private User(String name) {        this.name = name;    }    public String getName() { return name; }    private void setName(String name) { this.name = name; }}// 原生反射调用setName方法public static void main(String[] args) {    User user = new User();    String methodName = "setName";    Class<?>[] paramTypes = {String.class};    Object[] params = {"Tom"};    try {        // 1. 获取Class对象        Class<?> userClass = User.class;        // 2. 获取方法（需处理私有方法）        Method method = userClass.getDeclaredMethod(methodName, paramTypes);        // 3. 设置访问权限（私有方法需要打开访问权限）        method.setAccessible(true);        // 4. 调用方法        method.invoke(user, params);        // 5. 验证结果        System.out.println(user.getName()); // 输出"Tom"    } catch (NoSuchMethodException e) {        // 处理方法不存在异常        e.printStackTrace();    } catch (IllegalAccessException e) {        // 处理访问权限异常        e.printStackTrace();    } catch (InvocationTargetException e) {        // 处理方法执行异常        e.getTargetException().printStackTrace();    }}
```

这段代码用了 5 步操作和 3 个 catch 块才完成一个简单的方法调用，其中：

1、getDeclaredMethod和setAccessible(true)是访问私有方法的必备步骤

2、必须捕获NoSuchMethodException、IllegalAccessException等多种异常

3、每次反射操作都要重复这套模板代码，繁琐且易出错

而用 ReflectionUtils 实现同样的功能，代码会精简到：

```
// 用ReflectionUtils调用setName方法public static void main(String[] args) {    User user = new User();    // 一行代码完成反射调用    ReflectionUtils.invokeMethod(user, "setName", "Tom");    System.out.println(user.getName()); // 输出"Tom"}
```

无需手动获取 Method 对象，无需处理访问权限，无需捕获一堆异常，现在发现是不是清清爽爽！

二、ReflectionUtils 核心功能：反射操作的 “全能工具箱”

ReflectionUtils 提供了 20 多个静态方法，覆盖了类、方法、字段的常见反射操作，按功能可分为四大类：

1. 方法反射：轻松调用任意方法

调用方法是反射最常用的场景，ReflectionUtils 提供了多种便捷方法：

（1）调用指定方法

```
// 调用无参方法String name = (String) ReflectionUtils.invokeMethod(user, "getName");// 调用带参数方法（自动匹配参数类型）ReflectionUtils.invokeMethod(user, "setName", "Tom");// 调用静态方法（第一个参数传Class对象）Object result = ReflectionUtils.invokeMethod(User.class, "staticMethod", 123);
```

invokeMethod方法会自动：

* 1、根据方法名和参数查找对应的 Method 对象（支持私有方法）

* 2、自动设置setAccessible(true)，无需手动处理访问权限

* 3、将方法执行时的异常包装为ReflectionException（RuntimeException 的子类），无需强制捕获

（2）查找方法

当需要复用 Method 对象时，可以先查找再调用：

```
// 查找指定方法（参数类型精确匹配）Method setNameMethod = ReflectionUtils.findMethod(User.class, "setName", String.class);// 调用查找后的方法ReflectionUtils.invokeMethod(setNameMethod, user, "Jerry");// 查找所有方法（可配合Predicate过滤）List<Method> allMethods = ReflectionUtils.getAllMethods(    User.class,    method -> method.getName().startsWith("get") // 只保留getter方法);
```

findMethod解决了原生getDeclaredMethod需要精确参数类型的痛点，getAllMethods则能快速获取类的所有方法（包括父类继承的方法）。

2. 字段反射：灵活操作类的字段

与方法反射类似，ReflectionUtils 简化了字段的获取和修改操作：

（1）获取 / 设置字段值

```
// 获取字段值（支持私有字段）String name = (String) ReflectionUtils.getFieldValue(user, "name");// 设置字段值（自动处理私有字段）ReflectionUtils.setFieldValue(user, "name", "Tom");// 操作静态字段（第一个参数传Class对象）ReflectionUtils.setFieldValue(User.class, "staticField", "value");
```

无需手动调用field.setAccessible(true)，无需处理NoSuchFieldException，一行代码即可完成私有字段的读写。

（2）查找字段

```
// 查找指定字段Field nameField = ReflectionUtils.findField(User.class, "name");// 查找所有字段（支持过滤）List<Field> allFields = ReflectionUtils.getAllFields(    User.class,    field -> !Modifier.isStatic(field.getModifiers()) // 排除静态字段);
```

findField会在类及其父类中查找指定名称的字段，解决了原生getDeclaredField只能查找当前类字段的局限。

3. 类反射：便捷获取类信息

ReflectionUtils 还提供了类级别的反射工具方法：

```
// 实例化类（支持无参构造和有参构造）User user = (User) ReflectionUtils.newInstance(User.class); // 无参构造User userWithName = (User) ReflectionUtils.newInstance(User.class, "Tom"); // 有参构造// 获取类的所有接口Class<?>[] interfaces = ReflectionUtils.getAllInterfaces(User.class);// 判断类是否为特定类型（如是否为集合类）boolean isCollection = ReflectionUtils.isAssignable(Collection.class, ArrayList.class); // true
```

newInstance方法简化了通过反射创建对象的过程，支持任意参数的构造方法，比原生Class.newInstance()更灵活（后者仅支持无参构造）。

4. 反射异常处理：告别繁琐的 try-catch

原生反射需要处理NoSuchMethodException、IllegalAccessException等多种受检异常，导致代码臃肿。ReflectionUtils 将所有反射异常统一包装为ReflectionException（运行时异常），无需强制捕获：

```
// 原生反射（必须处理异常）try {    Method method = User.class.getDeclaredMethod("setName", String.class);    method.invoke(user, "Tom");} catch (NoSuchMethodException | IllegalAccessException | InvocationTargetException e) {    // 繁琐的异常处理    throw new RuntimeException(e);}// ReflectionUtils（无需try-catch）ReflectionUtils.invokeMethod(user, "setName", "Tom"); // 异常自动包装为ReflectionException
```

这种设计既避免了代码中充斥 try-catch 块，又保留了异常信息，需要时可通过try-catch ReflectionException处理特定场景。

三、实战案例：用 ReflectionUtils 简化框架开发

反射在框架开发中应用广泛，以一个简单的 ORM 框架字段映射功能为例，看看 ReflectionUtils 如何简化代码。

业务需求：实现一个通用方法，将 ResultSet 中的字段值自动映射到 Java 对象的属性（类似 MyBatis 的结果集映射）。

传统实现（原生反射）：

```
public class ResultSetMapper {    // 将ResultSet映射到对象    public static <T> T map(ResultSet rs, Class<T> clazz) throws SQLException {        try {            // 1. 创建对象实例            T obj = clazz.newInstance();  
            // 2. 获取ResultSet元数据            ResultSetMetaData metaData = rs.getMetaData();            int columnCount = metaData.getColumnCount();  
            // 3. 遍历字段并赋值            for (int i = 1; i <= columnCount; i++) {                String columnName = metaData.getColumnName(i);                Object value = rs.getObject(i);  
                // 4. 查找对应的字段（下划线转驼峰）                String fieldName = columnNameToFieldName(columnName); // 如user_name→userName                Field field = clazz.getDeclaredField(fieldName);  
                // 5. 设置字段可访问                field.setAccessible(true);  
                // 6. 转换类型并设置值                Object convertedValue = convertValue(value, field.getType());                field.set(obj, convertedValue);            }            return obj;        } catch (InstantiationException | IllegalAccessException | NoSuchFieldException e) {            throw new RuntimeException("映射失败", e);        }    }  
    // 下划线转驼峰（省略实现）    private static String columnNameToFieldName(String columnName) { ... }  
    // 类型转换（省略实现）    private static Object convertValue(Object value, Class<?> targetType) { ... }}
```

用 ReflectionUtils 重构后：

```
public class ResultSetMapper {    // 将ResultSet映射到对象（用ReflectionUtils简化）    public static <T> T map(ResultSet rs, Class<T> clazz) throws SQLException {        // 1. 创建对象实例（支持有参构造）        T obj = (T) ReflectionUtils.newInstance(clazz);  
        // 2. 获取ResultSet元数据        ResultSetMetaData metaData = rs.getMetaData();        int columnCount = metaData.getColumnCount();  
        // 3. 遍历字段并赋值        for (int i = 1; i <= columnCount; i++) {            String columnName = metaData.getColumnName(i);            Object value = rs.getObject(i);  
            // 4. 查找对应的字段（下划线转驼峰）            String fieldName = columnNameToFieldName(columnName);  
            // 5. 设置字段值（自动处理访问权限和异常）            Object convertedValue = convertValue(value, ReflectionUtils.findField(clazz, fieldName).getType());            ReflectionUtils.setFieldValue(obj, fieldName, convertedValue);        }        return obj;    }  
    // 下划线转驼峰（省略实现）    private static String columnNameToFieldName(String columnName) { ... }  
    // 类型转换（省略实现）    private static Object convertValue(Object value, Class<?> targetType) { ... }}
```

重构后的代码变化：

1. 1、对象实例化用ReflectionUtils.newInstance替代clazz.newInstance()，支持有参构造且无需处理InstantiationException

2. 2、字段查找用ReflectionUtils.findField替代clazz.getDeclaredField()，自动处理父类字段且无需捕获NoSuchFieldException

3. 3、字段赋值用ReflectionUtils.setFieldValue替代field.set()，自动设置访问权限且无需捕获IllegalAccessException

4. 4、整体代码量减少 30%，异常处理逻辑更清晰，可读性显著提升

四、最佳实践与避坑指南

1. 依赖引入

ReflectionUtils 位于commons-beanutils包中，需在项目中引入依赖：

```
<dependency>    <groupId>commons-beanutils</groupId>    <artifactId>commons-beanutils</artifactId>    <version>1.9.4</version> <!-- 推荐最新稳定版 --></dependency>
```

注意：commons-beanutils依赖commons-logging和commons-collections，Maven 会自动引入这些依赖，但需注意版本兼容性。

2. 性能优化

反射操作本身比直接调用慢，使用时需注意性能优化：

* 缓存反射结果：对于频繁调用的反射操作（如框架中的对象映射），缓存查找后的 Method 或 Field 对象：

```
// 缓存字段对象private static final Map<Class<?>, Map<String, Field>> FIELD_CACHE = new ConcurrentHashMap<>();// 获取字段（优先从缓存获取）private Field getField(Class<?> clazz, String fieldName) {    return FIELD_CACHE.computeIfAbsent(clazz, k -> new ConcurrentHashMap<>())        .computeIfAbsent(fieldName, k -> ReflectionUtils.findField(clazz, fieldName));}
```

避免过度使用反射：简单场景优先使用直接调用，反射仅用于动态性要求高的场景（如框架、动态代理）。

3. 安全与权限

* 1、访问私有成员的风险：ReflectionUtils 通过setAccessible(true)访问私有方法 / 字段，这会绕过 Java 的访问权限检查，可能导致安全管理器（SecurityManager）抛出异常。在有安全管理器的环境中使用时需特别注意。

* 2、不可变对象的修改：对于String、Integer等不可变对象，通过反射修改其内部字段可能导致不可预期的结果，应避免这种操作。

4. 与 Java 8 + 特性的配合

Java 8 引入的 MethodHandle API 提供了另一种反射方式，性能比传统反射更好。可以结合 ReflectionUtils 和 MethodHandle 使用：

```
// 用ReflectionUtils查找方法，再创建MethodHandleMethod method = ReflectionUtils.findMethod(User.class, "setName", String.class);MethodHandle handle = MethodHandles.lookup().unreflect(method);// 调用MethodHandle（性能优于传统反射）handle.bindTo(user).invoke("Tom");
```

这种方式兼顾了 ReflectionUtils 的便捷性和 MethodHandle 的高性能，适合对性能要求高的场景。

5. 常见误区

* 混淆方法名和参数：invokeMethod通过方法名和参数类型查找方法，若存在重载方法，需确保参数类型匹配：

```
// User类有两个setAge方法：setAge(int)和setAge(String)ReflectionUtils.invokeMethod(user, "setAge", "20"); // 调用setAge(String)ReflectionUtils.invokeMethod(user, "setAge", 20); // 调用setAge(int)
```

* 忽略继承关系：findMethod和findField会查找父类中的成员，但不包括私有成员（父类的私有成员对子类不可见）。如需访问父类私有成员，需手动遍历父类 Class 对象。

* 异常处理不当：ReflectionException是运行时异常，无需强制捕获，但重要场景应捕获并处理：

```
try {    ReflectionUtils.invokeMethod(user, "setName", "Tom");} catch (ReflectionException e) {    log.error("调用setName方法失败", e);    // 处理异常（如返回默认值、重试）}
```

五、ReflectionUtils 的适用场景

ReflectionUtils 在以下场景中能发挥最大价值：

1. 1、框架开发：ORM 框架（如 MyBatis）的结果集映射、Spring 的依赖注入等，需要动态访问对象的属性和方法。

2. 2、动态代理：AOP 框架中通过反射调用被代理对象的方法，如事务管理、日志记录。

3. 3、数据转换：JSON 框架（如 Jackson）的序列化 / 反序列化，需要动态读写对象字段。

4. 4、测试工具：单元测试中访问私有方法 / 字段进行测试，如验证私有方法的逻辑正确性。

结语：让反射回归其本质价值

无论是开发框架还是业务系统中的动态功能，ReflectionUtils 都能大幅减少模板代码，降低反射的使用门槛。但同时也要记住：反射是一把双刃剑，过度使用会导致代码可读性下降、性能受损。

优秀的开发者应当知其然，也知其所以然 —— 既会用 ReflectionUtils 简化反射操作，也明白其背后的原理和潜在风险。

看完之后发现是不是非常赞！

如果有什么不同的见解，那就在评论区发表一下吧！

还有更多好看的内容点击这里：