> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/FlinkUDF自定义函数|FlinkUDF自定义函数]]
---
title: 史上最全Flink1.20.0 UDF 使用教程，1.6w字从0到1教你学会各种自定义 UDF 的使用
author: 3分钟秒懂大数据
date:
url: http://mp.weixin.qq.com/s?__biz=Mzg5NDY3NzIwMA==&mid=2247514085&idx=1&sn=cf131978a8f8eb6db2fae4a4adff0d91&chksm=c18a79712a1666cb9e5a34c858e8a1de6eea418a8b8a37ea99176a30ce46d58b0d90a0338319&mpshare=1&scene=24&srcid=0122i1CleMc0aEKoDLtiZo4K&sharer_shareinfo=b498f939d8307649c9050fd472068142&sharer_shareinfo_first=b498f939d8307649c9050fd472068142#rd
---

```
点击上方公众号进入 3分钟秒懂大数据 主页

然后点击右上角 “设为标星” 

比别人更快接收硬核文章
```

## 1 背景

大家好，我是土哥。

目前已经辅导上百位小伙伴求职找工作，发现了一个问题，在这上百位小伙伴中，80% 同学不能熟练的使用 Flink 写自定义的 UDF，40% 的同学甚至没有听过，这当时让我就有些尴尬。

基本辅导每个同学，都会给他们发相应的文章或者案例，所以今天写一篇完整的教程及各种自定义案例，帮助各位同学快速使用。

**下面的文章已经生成完整的 PDF，包含文字 + github 代码链接，**需要的，加土哥微信领取。

## 2 自定义函数简介

在 Flink 中，如果系统自带的函数无法满足业务场景，允许用户自定义 UDF 函数来实现业务逻辑，即 Flink UDF。

Flink UDF（User-Defined Function，用户自定义函数）是 Flink 中一种自定义函数的实现方式，用于在 Flink 程序中对**输入数据**进行处理和转换。UDF 可以用于 Flink SQL 和 Table API 中，也可以在 Flink DataStream 中使用。

Flink UDF 函数，分为三类，分别为 Flink UDF, Flink UDTF, Flink UDAF：

1. Flink UDF

> Flink UDF（User-Defined Function） 为标量函数。
>
> 特点为：**单/多字段输入，单字段输出**，编写函数时，继承 Scalar Function.
>
> 使用场景：适合数据转换和简单的计算。如：字符串的格式转换，类型转换，根据某些条件计算新的字段值。

2. Flink UDTF

> Flink UDTF（User-Defined Table Function） 为表函数。
>
> 特点为：**单输入/多输入，多输出**。编写函数时，继承 Table Function
>
> 使用场景：数据拆分和数据扩展。例如：输入一个 json，返回 json 中的多个字段。或者根据某些规则生成额外的行数据。

3. Flink UDAF

> Flink UDAF（User-Defined Aggregate Function）为聚合函数。
>
> 特点为：**对一组数据进行聚合计算。可以维护中间状态，逐步累积计算结果**。编写函数时，继承 Table Function。
>
> 使用场景：
>
> 常见的聚合操作：如求平均值、总和、最大值、最小值等。
>
> 自定义的复杂聚合逻辑，比如计算移动平均值等。

## 3 标量函数的使用方式

### 3.1 根据 json 中的 key, 返回 value （多字段输入，单输出）

> 描述：根据 json 的 key，获取对应的 value，这在数仓的业务场景中，是非常基础的一个 udf 函数，同时也是使用最广泛、且重要的一个函数。如下是使用案例：

1. 定义 GetJsonObject 类，继承 **ScalarFunction。**
2. **重写 eval 方法**，该 eval 方法名不能修改，输入输出内容可以改变。
3. 根据 json 和 json 中的 key，获取对应的 value 值。

```
import com.alibaba.fastjson.JSONObject;
import org.apache.commons.lang3.StringUtils;
import org.apache.flink.table.functions.ScalarFunction;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class GetJsonObject extends ScalarFunction {

    private static final Logger LOGGER = LoggerFactory.getLogger(GetJsonObject.class);

    public String eval(String json, String key) {
        if (StringUtils.isNotBlank(json)) {
            try {
                JSONObject jsonObject = JSONObject.parseObject(json);
                Object value = jsonObject.get(key);
                if (value != null) {
                    //适配如果是json嵌套json,可以直接打成string
                    return value.toString();
                } else {
                    return null;
                }
            } catch (Exception e) {
                LOGGER.warn(String.format("get json object failed, json str:%s, key:%s", json, key), e);
            }
        }
        return null;

    }
}
```

### 3.2 根据日期进行类型转换（单字段输入，单字段输出）

1. 定义 DateFormatConverter 类，继承 ScalarFunction。
2. 重写 eval 方法。
3. 定义 SimpleDateFormat 类进行日期格式转换。

```
import org.apache.flink.table.functions.ScalarFunction;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

public class DateFormatConverter extends ScalarFunction {

    private static final SimpleDateFormat inputFormat = new SimpleDateFormat("yyyy-MM-dd");
    private static final SimpleDateFormat outputFormat = new SimpleDateFormat("dd/MM/yyyy");

    public String eval(String dateStr) {
        try {
            Date date = inputFormat.parse(dateStr);
            return outputFormat.format(date);
        } catch (ParseException e) {
            return null;
        }
    }
}
```

## 4 表函数的使用方式

### 4.1 Flink UDTF 使用介绍

Flink UDTF 在使用的过程中，需要通过 FunctionHint 注解，然后定义输入、输出类型，其中输入类型可以省略。

**FunctionHint 的作用：** 使用 FunctionHint 可以让 Flink 系统更准确地理解函数所期望的输入参数类型，避免类型不匹配的问题。

### 4.2 案例：输入一个 string 字符串，输出多个字段

1. 定义 RowConvertTableFunction 类，继承 TableFunction;
2. 重写 eval 方法;
3. 重写 getResultType() 方法，返回类型为：TypeInformation(Row)；由于 UDTF 输出一般为多个字段，所以需要用 Row 类型返回。
4. 通过JSON.parseObject 将string json 字符串转为 json 类型，然后获取其中的字段内容。
5. 定义Row row = Row.of(),将获取的多个字段，存入 of 方法中，进行返回。

```
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONArray;
import com.alibaba.fastjson.JSONObject;
import org.apache.flink.api.common.typeinfo.TypeInformation;
import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.table.annotation.DataTypeHint;
import org.apache.flink.table.annotation.FunctionHint;
import org.apache.flink.table.functions.TableFunction;
import org.apache.flink.types.Row;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.LinkedHashMap;
import java.util.Map;


@FunctionHint(
        input = @DataTypeHint("STRING"),
        output = @DataTypeHint(
                "ROW<project STRING," +
                        "module STRING," +
                        "userAgeRange STRING," +
                        "userCity STRING," +
                        "itemGender INT," +
                        "itemCnt INT>"
        )
)
public class RowConvertTableFunction extends TableFunction<Row> {

    private static final Logger LOG = LoggerFactory.getLogger(RowConvertTableFunction.class);

    private String project;

    private String module;

    private String userAgeRange;

    private String userCity;

    private int itemGender;

    private Map<JSONObject, Integer> itemObject;

    private int parserExceptionCount;

    private int errorCnt;

    private int itemCnt = 1;

    public RowConvertTableFunction() {
        itemObject = new LinkedHashMap<>();
        parserExceptionCount = 0;
        errorCnt = 1;
    }

    /**
     * @param str 输入是一行字符串，对应的是 json 格式的数据
     */
    public void eval(String str) {
        try {
            // 将数据的 str 字符串转为 json 对象
            JSONObject jsonObject = JSON.parseObject(str);
            // 1 解析最外层 json 数据，获取结果
            JsonObjectParser(jsonObject);
            // 获取 jsonArray 数组
            JSONArray item = jsonObject.getJSONArray("item");
            // 2 对 item 中相同的对象做聚合操作
            itemObject = itemAggregation(item);
            // 3 对解析后的结果进行收集
            collectResult(itemObject);
            // 清除 HashMap 存储记录
            itemObject.clear();
        } catch (Exception e) {
            ++parserExceptionCount;
            if (parserExceptionCount % errorCnt == 0) {
                LOG.error("json parse failed, the number of parse errors is {}", parserExceptionCount, e);
                if (errorCnt != 100000) {
                    errorCnt *= 10;
                }
            }
        }
    }

    /**
     * @param jsonObject 对获取的 json 数据进行解析
     */
    public void JsonObjectParser(JSONObject jsonObject) {
        project = jsonObject.getString("project");
        module = jsonObject.getString("module");
        userAgeRange = jsonObject.getJSONObject("user").getString("ageRange");
        userCity = jsonObject.getJSONObject("user").getString("city");
    }

    /**
     * @param item : 对输入的 item  Array格式进行解析，并聚合 item 中重复的数据
     * @return itemObject: 返回 Map 集合，key 为 包含的json 对象，value 为聚合后的统计值
     */
    public Map<JSONObject, Integer> itemAggregation(JSONArray item) {
        if (item != null && item.size() > 0) {
            // 遍历 jsonArray 数组，获取每个对象
            for (int i = 0; i < item.size(); i++) {
                JSONObject itemObj = item.getJSONObject(i);
                // 删除每个对象中的 id 属性
                itemObj.remove("id");
                // 通过 HashMap 对每个对象进行判断，如果集合中包含这个对象，value 值 +1
                if (itemObject.containsKey(itemObj)) {
                    Integer count = itemObject.get(itemObj);
                    itemObject.put(itemObj, ++count);
                } else {
                    itemObject.put(itemObj, itemCnt);
                }
            }
        }
        return itemObject;
    }

    /**
     * @param itemObject
     */
    public void collectResult(Map<JSONObject, Integer> itemObject) {
        Row row = null;
        if (itemObject.size() != 0) {
            // 遍历 HashMap 集合，
            for (Map.Entry<JSONObject, Integer> entry : itemObject.entrySet()) {
                itemGender = Integer.parseInt(entry.getKey().getString("gender"));
                itemCnt = entry.getValue();
                // 将输出类型封装成 Row 格式返回。
                row = Row.of(project, module, userAgeRange,
                        userCity, itemGender, itemCnt);
                collect(row);
            }
        } else {
            itemGender = 0;
            itemCnt = 1;
            row = Row.of(project, module, userAgeRange,
                    userCity, itemGender, itemCnt);
            collect(row);
        }
    }

    @Override
    public TypeInformation<Row> getResultType() {
        return Types.ROW(
                Types.STRING,
                Types.STRING,
                Types.STRING,
                Types.STRING,
                Types.INT,
                Types.INT);
    }
}
```

## 5 聚合函数的使用方式

### 5.1 Flink UDAF 使用介绍

介绍一下 Flink UDAF 的使用方式。

Flink UDAF，需要继承 AggregateFunction<T,ACC> 抽象类，实现一系列方法。AggregateFunction 抽象类如下：

```
abstract class AggregateFunction<T, ACC>
           extends UserDefinedAggregateFunction<T, ACC>
T: 表示 UDAF 最终输出的结果类型   

ACC: 表示 UDAF 存放中间结果的类型
```

最基本的 Flink UDAF 至少需要实现如下三个方法:

1. 需要先定义一个 Accumulator 类，类里面定义变量，存放聚合的中间结果;
2. **创建 createAccumulator**: createAccumulator 方法是用来初始化你定义的 Accumulator 类，将内部定义的变量赋值为空或者 0。
3. **创建 accumulate**: 定义如何根据输入更新 Accumulator，主要是编写中间的逻辑代码，根据输入变量来更新你的输出中间变量。
4. **创建 getValue**: 定义如何返回 Accumulator 中存储的中间结果作为 UDAF 的最终结果。

### 5.2 案例：计算输入元素的平均值

1. 定义 AggregateFunction 类，继承 AggregateFunction 函数，返回值为 Double 类型，中间结果类为 AverageAccumulator。
2. 定义 AverageAccumulator 代表中间结果类型和输出 double 类型。
3. 实现 createAccumulator 方法，用于创建聚合的初始状态。
4. 实现 accumulate 方法，将输入元素添加到聚合状态中。
5. 实现 getValue 方法，根据聚合状态计算最终结果。
6. 实现 merge 方法，用于合并不同分区的聚合状态,该方法为**（可选）**。
7. 实现 resetAccumulator 方法，用于重置聚合状态，该方法为**（可选）**。

```
import org.apache.flink.api.common.typeinfo.TypeHint;
import org.apache.flink.api.common.typeinfo.TypeInformation;
import org.apache.flink.table.functions.AggregateFunction;

// 定义一个简单的 UDAF，用于计算输入元素的平均值
public class AverageAggregateFunction extends AggregateFunction<Double, AverageAggreateFunction.AverageAccumulator> {

    // 1 定义一个 中间结果类型的类 Accumulator，存放聚合的中间结果
    public static class AverageAccumulator {
        public long sum = 0;
        public int count = 0;
    }

    // 2 创建初始的聚合状态,重写 createAccumulator 方法
    @Override
    public AverageAccumulator createAccumulator() {
        return new AverageAccumulator();
    }

    // 3 创建 accumulate 方法，将输入元素添加到聚合状态中，
    public void accumulate(AverageAccumulator accumulator, Long value) {
        if (value!= null) {
            accumulator.sum += value;
            accumulator.count++;
        }
    }

    // 4 重写getValue, 计算最终结果
    @Override
    public Double getValue(AverageAccumulator accumulator) {
        if (accumulator.count == 0) {
            return null;
        }
        return ((double) accumulator.sum) / accumulator.count;
    }

    // 合并不同分区的聚合状态
    public void merge(AverageAccumulator accumulator, Iterable<AverageAccumulator> its) {
        for (AverageAccumulator otherAcc : its) {
            accumulator.sum += otherAcc.sum;
            accumulator.count += otherAcc.count;
        }
    }

    // 重置聚合状态
    public void resetAccumulator(AverageAccumulator accumulator) {
        accumulator.sum = 0;
        accumulator.count = 0;
    }

    // 定义输出结果的类型信息
    @Override
    public TypeInformation<Double> getResultType() {
        return TypeInformation.of(new TypeHint<Double>() {});
    }

    // 定义中间结果的类型信息
    @Override
    public TypeInformation<AverageAccumulator> getAccumulatorType() {
        return TypeInformation.of(new TypeHint<AverageAccumulator>() {});
    }
}
```

## 6 UDF 进阶版

上述介绍了 Flink UDF、UDAF、UDTF的使用方法，但是在真实的业务中，如果遇到输入数据量 100w/s 的业务场景，同时每条数据调用调用 UDF, 那这个时候，UDF 的解析性能就非常重要了。

所以在生产环境中，为了对 udf 进行性能监测，我们多数情况下会对 UDF 添加监控，以及增加缓存，来提升 UDF 的解析性能。

### 6.1 UDF 中添加 Flink Metrics 监控指标

在 Flink 自定义函数中，我们经常重写的是 eval 方法，但是在 taskManager 的整个运行生命周期中，是从 open 方法开始初始化，到 close 方法结束。所以一个自定义函数
正常都是会包含 open 方法和 close 方法。

而 Flink Metrics 监控指标是在 open 方法初始化，但好多同学一般不太会使用，这里给大家写一个完整的案例，我们直接使用 **4.2 章节的案例添加 Flink Metrics 监控指标**。

```
package org.threeknowbigdata.udtf;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONArray;
import com.alibaba.fastjson.JSONObject;
import org.apache.flink.api.common.typeinfo.TypeInformation;
import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.metrics.Counter;
import org.apache.flink.metrics.Gauge;
import org.apache.flink.metrics.MetricGroup;
import org.apache.flink.table.annotation.DataTypeHint;
import org.apache.flink.table.annotation.FunctionHint;
import org.apache.flink.table.functions.FunctionContext;
import org.apache.flink.table.functions.TableFunction;
import org.apache.flink.types.Row;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.LinkedHashMap;
import java.util.Map;


/**
 * @ClassName: RowConvertTableFunction
 * @Description: TODO
 * @Author: 土哥
 * @Date: 2025/01/21 22:43:06
 * @Version: V1.0
 **/

@FunctionHint(
        input = @DataTypeHint("STRING"),
        output = @DataTypeHint(
                "ROW<project STRING," +
                        "module STRING," +
                        "userAgeRange STRING," +
                        "userCity STRING," +
                        "itemGender INT," +
                        "itemCnt INT>"
        )
)
public class RowConvertTableFunction extends TableFunction<Row> {

    /**
     * 一行转多行的转换时间
     */
    public static final String ROW_CONVERT_TIME = "rowConvertTime";
    /**
     * item 解析时间
     */
    public static final String ITEM_PARSER_TIME = "itemParserTime";
    /**
     * item 初始条数
     */
    public static final String ITEM_INITIAL_COUNT = "itemInitialCount";
    /**
     * item 聚合后条数
     */
    public static final String ITEM_AGGREGATE_COUNT = "itemAggregateCount";
    /**
     * json 解析异常次数
     */
    public static final String PARSER_EXCEPTION_CNT = "parserExceptionCnt";
    private static final Logger LOG = LoggerFactory.getLogger(RowConvertTableFunction.class);
    private transient Gauge<Long> rowConvertGuage;
    private transient Gauge<Long> itemParserGuage;
    private transient Gauge<Integer> itemInitialCounter;
    private transient Gauge<Integer> itemAggregateCounter;
    private transient Counter parserExceptionCnt;

    private String project;

    private String module;

    private String userAgeRange;

    private String userCity;

    private int itemGender;

    private Map<JSONObject, Integer> itemObject;

    private int parserExceptionCount;

    private int errorCnt;

    private int itemCnt = 1;
    // 用于存储行转换时间
    private long rowConvertTime;
    // 用于存储 item 解析时间
    private long itemParserTime;
    // 用于存储 item 初始条数
    private int itemInitialCount;
    // 用于存储 item 聚合后条数
    private int itemAggregateCount;

    public RowConvertTableFunction() {
        itemObject = new LinkedHashMap<>();
        parserExceptionCount = 0;
        errorCnt = 1;
    }

    /**
     * @param context 创建 Metrics 监控指标
     */
    @Override
    public void open(FunctionContext context) {
        MetricGroup metricGroup = context.getMetricGroup();
        // 创建和注册 Gauge 指标
        rowConvertGuage = metricGroup.gauge(ROW_CONVERT_TIME, () -> rowConvertTime);
        itemParserGuage = metricGroup.gauge(ITEM_PARSER_TIME, () -> itemParserTime);
        itemInitialCounter = metricGroup.gauge(ITEM_INITIAL_COUNT, () -> itemInitialCount);
        itemAggregateCounter = metricGroup.gauge(ITEM_AGGREGATE_COUNT, () -> itemAggregateCount);
        // 创建和注册 Counter 指标
        parserExceptionCnt = metricGroup.counter(PARSER_EXCEPTION_CNT);
    }

    /**
     * @param str 输入是一行字符串，对应的是 json 格式的数据
     */
    public void eval(String str) {
        try {
            long startTime = System.currentTimeMillis();
            long parserJsonStartTime = System.nanoTime();
            // 将数据的 str 字符串转为 json 对象
            JSONObject jsonObject = JSON.parseObject(str);
            // 1 解析最外层 json 数据，获取结果
            JsonObjectParser(jsonObject);
            // 获取 jsonArray 数组
            JSONArray item = jsonObject.getJSONArray("item");
            long parserItemStartTime = System.nanoTime();
            // 监控指标： 获取单个 item 聚合前的数量
            itemInitialCount = item.size();
            // 2 对 item 中相同的对象做聚合操作
            itemObject = itemAggregation(item);
            // 监控指标： 获取单个 item 聚合前的数量
            itemParserTime = System.nanoTime() - parserItemStartTime;
            // 监控指标：获取单个 item 聚合后的数量
            itemAggregateCount = itemObject.size();
            // 3 对解析后的结果进行收集
            collectResult(itemObject);
            // 监控指标：获取整个字符串的转换的时间
            rowConvertTime = System.nanoTime() - parserJsonStartTime;
            // 清除 HashMap 存储记录
            itemObject.clear();
            long executionTime = System.currentTimeMillis() - startTime;
            LOG.info("data conversion takes {} ms", executionTime);
        } catch (Exception e) {
            ++parserExceptionCount;
            parserExceptionCnt.inc();
            if (parserExceptionCount % errorCnt == 0) {
                LOG.error("json parse failed, the number of parse errors is {}", parserExceptionCount, e);
                if (errorCnt!= 100000) {
                    errorCnt *= 10;
                }
            }
        }
    }

    /**
     * @param jsonObject 对获取的 json 数据进行解析
     */
    public void JsonObjectParser(JSONObject jsonObject) {
        project = jsonObject.getString("project");
        module = jsonObject.getString("module");
        userAgeRange = jsonObject.getJSONObject("user").getString("ageRange");
        userCity = jsonObject.getJSONObject("user").getString("city");
    }

    /**
     * @param item : 对输入的 item  Array格式进行解析，并聚合 item 中重复的数据
     * @return itemObject: 返回 Map 集合，key 为 包含的json 对象，value 为聚合后的统计值
     */
    public Map<JSONObject, Integer> itemAggregation(JSONArray item) {
        if (item!= null && item.size() > 0) {
            // 遍历 jsonArray 数组，获取每个对象
            for (int i = 0; i < item.size(); i++) {
                JSONObject itemObj = item.getJSONObject(i);
                // 删除每个对象中的 id 属性
                itemObj.remove("id");
                // 通过 HashMap 对每个对象进行判断，如果集合中包含这个对象，value 值 +1
                if (itemObject.containsKey(itemObj)) {
                    Integer count = itemObject.get(itemObj);
                    itemObject.put(itemObj, ++count);
                } else {
                    itemObject.put(itemObj, itemCnt);
                }
            }
        }
        return itemObject;
    }

    /**
     * @param itemObject
     */
    public void collectResult(Map<JSONObject, Integer> itemObject) {
        Row row = null;
        if (itemObject.size()!= 0) {
            // 遍历 HashMap 集合，
            for (Map.Entry<JSONObject, Integer> entry : itemObject.entrySet()) {
                itemGender = Integer.parseInt(entry.getKey().getString("gender"));
                itemCnt = entry.getValue();
                // 将输出类型封装成 Row 格式返回。
                row = Row.of(project, module, userAgeRange,
                        userCity, itemGender, itemCnt);
                collect(row);
            }
        } else {
            itemGender = 0;
            itemCnt = 1;
            row = Row.of(project, module, userAgeRange,
                    userCity, itemGender, itemCnt);
            collect(row);
        }
    }

    @Override
    public TypeInformation<Row> getResultType() {
        return Types.ROW(
                Types.STRING,
                Types.STRING,
                Types.STRING,
                Types.STRING,
                Types.INT,
                Types.INT);
    }
}
```

代码解释：在 open 方法中：

1. MetricGroup metricGroup = context.getMetricGroup();：从 FunctionContext 中获取 MetricGroup，用于后续指标的创建和注册。
2. rowConvertGuage = metricGroup.gauge(ROW\_CONVERT\_TIME, () -> 0L);：创建一个 Gauge 指标用于监控行转换时间，初始值为 0L。
3. itemParserGuage = metricGroup.gauge(ITEM\_PARSER\_TIME, () -> 0L);：创建一个 Gauge 指标用于监控 item 解析时间，初始值为 0L。
4. itemInitialCounter = metricGroup.gauge(ITEM\_INITIAL\_COUNT, () -> 0);：创建一个 Gauge 指标用于监控 item 初始数量，初始值为 0。
5. itemAggregateCounter = metricGroup.gauge(ITEM\_AGGREGATE\_COUNT, () -> 0);：创建一个 Gauge 指标用于监控 item 聚合后数量，初始值为 0。
6. parserExceptionCnt = metricGroup.counter(PARSER\_EXCEPTION\_CNT);：创建一个 Counter 指标用于监控解析异常次数。

### 6.2 UDF 中添加缓存

在 ODS 层，get\_json\_object 解析 json 函数使用非常频繁，所以我们针对 3.1 章节的案例进行优化，对该函数添加缓存。优化后的案例如下：

```
import com.alibaba.fastjson.JSONObject;
import com.github.benmanes.caffeine.cache.Cache;
import com.github.benmanes.caffeine.cache.Caffeine;
import org.apache.commons.lang3.StringUtils;
import org.apache.flink.table.functions.ScalarFunction;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.util.concurrent.TimeUnit;
import java.util.function.Function;

public class GetJsonObject extends ScalarFunction {

    private static final Logger LOGGER = LoggerFactory.getLogger(GetJsonObject.class);

    private static final Cache<String, JSONObject> JsonCache;

    static {
        JsonCache = Caffeine.newBuilder().expireAfterAccess(30, TimeUnit.MINUTES).maximumSize(1000).build();
        LOGGER.info("using json cache for get json object");
    }

    public String eval(String json, String key) {
        if (StringUtils.isNotBlank(json)) {
            try {
                JSONObject jsonObject = JsonCache.get(json, new Function<String, JSONObject>() {
                    @Override
                    public JSONObject apply(String s) {
                        return JSONObject.parseObject(json);
                    }
                });
                Object obj = jsonObject.get(key);
                if (obj != null) {
                    return obj.toString();
                } else {
                    return null;
                }
            } catch (Exception e) {
                LOGGER.warn(String.format("get json object failed, json str:%s, key:%s", json, key), e);
            }
        }
        return null;
    }
}
```

以上 Flink UDF 的作用是将 JSON 字符串解析成 Map<String, String> 类型的对象，并且使用 Caffeine 缓存库对解析出的对象进行缓存，**缓存时间为 30 分钟，最大缓存数量为 1000**。

如果缓存中已经存在该 JSON 字符串的解析结果，则直接从缓存中获取结果，否则重新解析 JSON 字符串并将解析结果存入缓存。

## 7 总结

本次给大家全面进行Flink UDF 的讲解及案例分享，希望可以对你有所帮助，祝你学业有成，找到一份好的工作，同时在工作中步步高升，一路前进~

**上面的文章已经生成完整的PDF，包含文字 + github代码链接，**需要的，加土哥微信领取。

上一篇热门技术文章：[我鸽掉腾讯音乐邀约面试，只因为觉得...](https://mp.weixin.qq.com/s?__biz=Mzg5NDY3NzIwMA==&mid=2247513978&idx=1&sn=71def9da020b5a9a68d2dcb4793c8555&scene=21#wechat_redirect)

end

**增值服务：简历修改|面试辅导|Flink资料|**模拟面试****

你好，我是土哥，计算机硕士毕业，现某大厂资深大数据开发工程师。出生在一个 18 线开外的小村庄，通过自己努力毕业一年在新一线城市买房，在社招、校招斩获 25 家中大厂 offer。

2025已经打响了第一枪，很多公司已经开启了**年前面试-年后入职**的流程。如果你想跳槽，但苦于一个人孤军奋战、无人指导、复习无从下手，或者不擅长写简历，手上只有拿不出手的毫无难点亮点的项目经历...

那么我的建议是多和身边的大佬沟通，哪怕是付费咨询，只要你能从他身上学到经验，那就是值得的。如果身边没有这样的人，那么我就毛遂自荐一下吧，毕竟，茫茫网络你能看到这篇文章何尝不是一种命运安排。

[土哥社招参加 28 场面试，100% 通过率，拿到全部 offer！](http://mp.weixin.qq.com/s?__biz=Mzg5NDY3NzIwMA==&mid=2247511408&idx=1&sn=beb292ab97ada3ee486511bfe503117d&chksm=c01914cff76e9dd90fd81857805a57aadcf4fa0a3ce731e5939d8651ed9bac561dba6bb7e03a&scene=21#wechat_redirect)

[土哥这半年的悲惨人生，经历过被鸽 offer，最终触底反弹~](http://mp.weixin.qq.com/s?__biz=Mzg5NDY3NzIwMA==&mid=2247510455&idx=1&sn=9cccfbebca3d2ee9d72538d73dd6fe74&chksm=c0191008f76e991e1760857f7c8a75deb1e0231ce8ba189eaf280c84e8a5e65f7f0197988ff1&scene=21#wechat_redirect)

需要任一增值服务或进大厂交流求职群

都欢迎加土哥微信（备注来意）

点赞和在看就是最大的支持❤️