---
title: 分享 Function 封装的 12 个常用工具类！
author: Java程序员成神之路
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzNDgxMDI5NQ==&mid=2247486189&idx=1&sn=083f5a4f81a08f893baa0348c0eb58ac&chksm=c39e93e777ae1a6d1f09d836702dc8923c9a71e0f26c51a4b6340d44bd0a1654ce03aa075932&mpshare=1&scene=24&srcid=0910onxYmW511n6XZz8p6gDj&sharer_shareinfo=a464f33a388e96db6182aebe4f4e8fed&sharer_shareinfo_first=a464f33a388e96db6182aebe4f4e8fed#rd
---

> 字数 1568，阅读大约需 4 分钟

在日常开发中，很多所谓的“工具类”其实是机械性的堆砌，方法写了一大堆，但调用时依然冗长。久而久之，工具类就成了“万能垃圾桶”。我后来发现，把这些方法抽象成 `Function`，再结合 `Stream`、`Optional`、缓存、懒加载等机制，可以极大提升代码的可读性和复用性。  
这次我把最常用的 12 个 Function 封装沉淀了一下，感兴趣的可以阅读下。

---

## 1. 字符串转整型

```
public static final Function<String, Integer> toInt = s -> {  
    try {  
        return s == null ? null : Integer.parseInt(s.trim());  
    } catch (NumberFormatException e) {  
        return null;  
    }  
};
```

用途：避免到处写 `try/catch`，特别适合做 Map 数据解析。

---

## 2. BigDecimal 保留两位小数

```
public static final Function<BigDecimal, BigDecimal> scale2 =   
        b -> b == null ? null : b.setScale(2, RoundingMode.HALF_UP);
```

用途：金额处理场景，能直接作为流的 mapping。

---

## 3. String 转 BigDecimal

```
public static final Function<String, BigDecimal> toDecimal = s -> {  
    try {  
        return (s == null || s.isEmpty()) ? null : new BigDecimal(s.trim());  
    } catch (Exception e) {  
        return null;  
    }  
};
```

用途：解析 Excel 或外部接口的金额字段，保证健壮性。

---

## 4. Null 安全的 Trim

```
public static final Function<String, String> safeTrim =   
        s -> s == null ? null : s.trim();
```

用途：最简单，却是数据清洗里出现频率最高的。

---

## 5. 日期字符串转 LocalDate

```
public static final Function<String, LocalDate> toLocalDate = s -> {  
    try {  
        return (s == null || s.isEmpty()) ? null : LocalDate.parse(s, DateTimeFormatter.ISO_DATE);  
    } catch (Exception e) {  
        return null;  
    }  
};
```

用途：REST API 或 JSON 里传日期字符串时用。

---

## 6. 对象转 JSON

```
public static final Function<Object, String> toJson = o -> {  
    try {  
        return o == null ? null : new ObjectMapper().writeValueAsString(o);  
    } catch (JsonProcessingException e) {  
        return null;  
    }  
};
```

用途：调试、日志、缓存时直接拿来用。

---

## 7. JSON 转 Map

```
public static final Function<String, Map<String, Object>> jsonToMap = s -> {  
    try {  
        return s == null ? null : new ObjectMapper().readValue(s, new TypeReference<>() {});  
    } catch (Exception e) {  
        return Collections.emptyMap();  
    }  
};
```

用途：在动态表单、配置系统中解析 JSON 时特别常见。

---

## 8. String 转 Boolean（多格式支持）

```
public static final Function<String, Boolean> toBoolean = s -> {  
    if (s == null) return null;  
    String lower = s.trim().toLowerCase();  
    return Arrays.asList("1", "true", "yes", "y", "t").contains(lower);  
};
```

用途：兼容多种布尔表示，避免重复写判断。

---

## 9. 文件大小格式化

```
public static final Function<Long, String> fileSizeFormat = size -> {  
    if (size == null) return null;  
    if (size < 1024) return size + "B";  
    int z = (63 - Long.numberOfLeadingZeros(size)) / 10;  
    return String.format("%.1f %sB", (double) size / (1L << (z * 10)), " KMGTPE".charAt(z));  
};
```

用途：日志打印、UI 展示。

---

## 10. Optional 包装器

```
public static final Function<Object, Optional<Object>> toOptional =   
        Optional::ofNullable;
```

用途：在流里避免空指针，还能进一步链式操作。

---

## 11. 驼峰转下划线

```
public static final Function<String, String> camelToUnderscore = s -> {  
    if (s == null) return null;  
    return s.replaceAll("([a-z])([A-Z]+)", "$1_$2").toLowerCase();  
};
```

用途：数据库字段与 Java 属性的映射。

---

## 12. 下划线转驼峰

```
public static final Function<String, String> underscoreToCamel = s -> {  
    if (s == null) return null;  
    StringBuilder result = new StringBuilder();  
    boolean toUpper = false;  
    for (char c : s.toCharArray()) {  
        if (c == '_') {  
            toUpper = true;  
        } else {  
            result.append(toUpper ? Character.toUpperCase(c) : c);  
            toUpper = false;  
        }  
    }  
    return result.toString();  
};
```

用途：数据库 -> DTO 映射反向场景。

---

## 为什么用 Function，而不是写死在工具类？

传统工具类的问题是：

1. 1. **调用方式死板**：你只能 `Util.method(x)`。
2. 2. **组合性差**：两个方法组合必须自己写新方法。
3. 3. **与 Stream 隔离**：无法自然嵌入数据流处理。

而 `Function` 的优势在于：

* • 可以直接 `.map(toInt.andThen(scale2))`，组合性极佳。
* • 可以做缓存，例如 `Function<String, Integer> cachedToInt = toInt.andThen(i -> i == null ? null : cache(i));`。
* • 可以传参、做策略模式的实现，非常优雅。

---

## 总结

很多人写工具类只是出于“能复用”，却忽略了“能组合”。当你把常见逻辑提炼成 `Function`，不仅调用简洁，还能和 Java Stream、Optional、CompletableFuture 等 API 无缝对接。

真正的复用不是工具方法越多越好，而是这些方法能够在不同的场景里自由拼装。  
这是我过去几年在项目里反复打磨出的心得，也希望能帮你彻底摆脱“工具类垃圾桶”的困扰。

最后奉上源码如下：

---

```
package com.example.util;  
  
import com.fasterxml.jackson.core.JsonProcessingException;  
import com.fasterxml.jackson.core.type.TypeReference;  
import com.fasterxml.jackson.databind.ObjectMapper;  
  
import java.math.BigDecimal;  
import java.math.RoundingMode;  
import java.time.LocalDate;  
import java.time.format.DateTimeFormatter;  
import java.util.*;  
import java.util.function.Function;  
  
public final class Functions {  
  
    private static final ObjectMapper MAPPER = new ObjectMapper();  
  
    private Functions() {  
        // 工具类禁止实例化  
    }  
  
    /** 1. 字符串转整型 */  
    public static final Function<String, Integer> TO_INT = s -> {  
        try {  
            return s == null ? null : Integer.parseInt(s.trim());  
        } catch (NumberFormatException e) {  
            return null;  
        }  
    };  
  
    /** 2. BigDecimal 保留两位小数 */  
    public static final Function<BigDecimal, BigDecimal> SCALE2 =  
            b -> b == null ? null : b.setScale(2, RoundingMode.HALF_UP);  
  
    /** 3. String 转 BigDecimal */  
    public static final Function<String, BigDecimal> TO_DECIMAL = s -> {  
        try {  
            return (s == null || s.isEmpty()) ? null : new BigDecimal(s.trim());  
        } catch (Exception e) {  
            return null;  
        }  
    };  
  
    /** 4. Null 安全的 Trim */  
    public static final Function<String, String> SAFE_TRIM =  
            s -> s == null ? null : s.trim();  
  
    /** 5. 日期字符串转 LocalDate */  
    public static final Function<String, LocalDate> TO_LOCAL_DATE = s -> {  
        try {  
            return (s == null || s.isEmpty()) ? null : LocalDate.parse(s, DateTimeFormatter.ISO_DATE);  
        } catch (Exception e) {  
            return null;  
        }  
    };  
  
    /** 6. 对象转 JSON */  
    public static final Function<Object, String> TO_JSON = o -> {  
        try {  
            return o == null ? null : MAPPER.writeValueAsString(o);  
        } catch (JsonProcessingException e) {  
            return null;  
        }  
    };  
  
    /** 7. JSON 转 Map */  
    public static final Function<String, Map<String, Object>> JSON_TO_MAP = s -> {  
        try {  
            return s == null ? null : MAPPER.readValue(s, new TypeReference<>() {});  
        } catch (Exception e) {  
            return Collections.emptyMap();  
        }  
    };  
  
    /** 8. String 转 Boolean（多格式支持） */  
    public static final Function<String, Boolean> TO_BOOLEAN = s -> {  
        if (s == null) return null;  
        String lower = s.trim().toLowerCase();  
        return Arrays.asList("1", "true", "yes", "y", "t").contains(lower);  
    };  
  
    /** 9. 文件大小格式化 */  
    public static final Function<Long, String> FILE_SIZE_FORMAT = size -> {  
        if (size == null) return null;  
        if (size < 1024) return size + "B";  
        int z = (63 - Long.numberOfLeadingZeros(size)) / 10;  
        return String.format("%.1f %sB", (double) size / (1L << (z * 10)), " KMGTPE".charAt(z));  
    };  
  
    /** 10. Optional 包装器 */  
    public static final Function<Object, Optional<Object>> TO_OPTIONAL =  
            Optional::ofNullable;  
  
    /** 11. 驼峰转下划线 */  
    public static final Function<String, String> CAMEL_TO_UNDERSCORE = s -> {  
        if (s == null) return null;  
        return s.replaceAll("([a-z])([A-Z]+)", "$1_$2").toLowerCase();  
    };  
  
    /** 12. 下划线转驼峰 */  
    public static final Function<String, String> UNDERSCORE_TO_CAMEL = s -> {  
        if (s == null) return null;  
        StringBuilder result = new StringBuilder();  
        boolean toUpper = false;  
        for (char c : s.toCharArray()) {  
            if (c == '_') {  
                toUpper = true;  
            } else {  
                result.append(toUpper ? Character.toUpperCase(c) : c);  
                toUpper = false;  
            }  
        }  
        return result.toString();  
    };  
}
```

---

### 使用方式

在类中引入：

```
import static com.example.util.Functions.*;
```

然后直接使用，例如：

```
List<String> numbers = List.of("1", " 2", "abc", "  3");  
List<Integer> parsed = numbers.stream()  
        .map(TO_INT)  
        .filter(Objects::nonNull)  
        .toList();  
  
String size = FILE_SIZE_FORMAT.apply(10_485_760L); // => "10.0 MB"
```

 

识别二维码关注我们

**点击分享此文**