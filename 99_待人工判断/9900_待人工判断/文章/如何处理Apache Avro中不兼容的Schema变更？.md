---
title: 如何处理Apache Avro中不兼容的Schema变更？
author: Apache Hudi
date: 
url: http://mp.weixin.qq.com/s?__biz=MzIyMzQ0NjA0MQ==&mid=2247488559&idx=2&sn=ec9e684d8700e0dee5486156ec77be3f&chksm=e81f4159df68c84f4366d1c028d9f8d7d7947a2bd8413b19de2adc06d8dd3da435cc5206e0e4&mpshare=1&scene=24&srcid=0315eud7MW75VKytNTjk80jq&sharer_sharetime=1647304068857&sharer_shareid=4145ee29355a80b1b2b01921ded70e3a#rd
---

Apache Avro[1] 有模式兼容性的概念，它允许我们判定一个模式是否与一个或多个早期或更新的模式是否兼容，有兼容的变更必然意味着也可以有不兼容的变更，在这种情况下应该做什么来实现这些重大变化（breaking changes），同时最大限度地减少对下游无论是流式还是批处理消费者的影响。重大变更意味着精心策划的迁移和相关的中断，因此建议尽可能避免重大变更，即使这意味着只有通过妥协才能实现所需的最终状态模式，事实证明根据兼容模式，至少可以通过一系列兼容变更来实现功能等价的模式。本文演示了如何将一个示例不兼容的变更实现为一系列兼容的变更。

# 重大变更

假设我们有一个业务需求，我们需要将包含复合全名的字符串字段更改为具有封装单独名称元素的记录的字段。从字符串到记录的转变显然是一个重大变更

## 当前状态

```
record Person {  
   string name; // example: "Joan Smith"  
}
```

## 预期状态

```
record Person {  
   Name name;  
}  
record Name {  
   string first_name; // example: "Joan"  
   string last_name;  // example: "Smith"  
}
```

# 可兼容变更顺序

这显然是一个重大变更，我们描述达到在功能上等同于所需最终状态的模式所需的步骤，通过避免重大更改，我们可以最大程度地减少因数据集不兼容版本之间的迁移而对消费者造成的中断。

## 第一步-添加默认值

可以删除具有默认值的字段，因此我们现在添加一个默认值，以便我们以后可以删除该字段，选择一个默认值，该值在消费者系统中没有实际意义，以后可用于将该字段标识为已弃用，继续使用消费者的数据填充该字段。注意：对于 BACKWARDS 或 BACKWARDS\_TRANSITIVE 不需要此步骤，其中字段可以在没有默认值的情况下被删除。

```
record Person {  
   string name = "<DEPRECATED>"; // example: "Joan Smith"  
}
```

## 第二步-引入新字段（尽可能有默认值）

我们正在引入我们想要的最终状态的字段，但是我们还不能使用所需的字段名称，因为它将被重载。此外对于 FORWARDS 或 FORWARDS\_TRANSITIVE 以外的兼容模式，我们必须提供默认值。生产者应该使用有效数据填充两个字段 - 现有的和新的。现在与所有消费者沟通，他们应该开始使用新字段，当他们都这样做时便可以继续下一步。注意：如果使用 FORWARDS\_TRANSITIVE 或 FULL\_TRANSITIVE，这是可以预期的最佳结果。

```
record Person {  
   string name        = "<DEPRECATED>"; // example: "Joan Smith"  
   Name   person_name = {"first_name":"<NOT_IN_USE>","last_name":"<NOT_IN_USE>"};  
}  
record Name {  
   string first_name; // example: "Joan"  
   string last_name;  // example: "Smith"  
}
```

## 第三步-移除旧字段

因为旧字段有一个默认值，并且现在没有消费者使用它，因此现在可以安全地删除它。注意：如果使用 BACKWARDS\_TRANSITIVE，这是可以预期的最佳结果。

```
record Person {  
   Name person_name = {"first_name":"<NOT_IN_USE>","last_name":"<NOT_IN_USE>"};  
}  
record Name {  
   string first_name; // example: "Joan"  
   string last_name;  // example: "Smith"  
}
```

## 第四步-移除默认值

现在可以从新字段中删除默认值，请注意，这不适用于 FORWARDS，因为该字段可以在步骤 2 中声明而没有默认值。

```
record Person {  
   Name person_name;  
}  
record Name {  
   string first_name; // example: "Joan"  
   string last_name;  // example: "Smith"  
}
```

## 第五步-重命名字段

最后我们可以通过提供具有所需名称的别名来有效地重命名它：

```
record Person {  
   @aliases(["name"])  
   Name person_name;  
}  
record Name {  
   string first_name; // example: "Joan"  
   string last_name;  // example: "Smith"  
}
```

# 兼容模式

本节总结了可在每种兼容模式中应用的步骤顺序，以及每种情况下可实现的最佳结果模式。请注意，虽然最终模式可能不像所需的最终状态模式那样简洁，但已经避免了大量的中断，否则这些中断可能会因不兼容的更改而导致。

# 最终结果

这些是可用于每种兼容模式的最佳可实现结果。

## Backwards（向后兼容）

```
record Person {  
   @aliases(["name"])  
   Name person_name;  
}  
record Name {  
   string first_name; // example: "Joan"  
   string last_name;  // example: "Smith"  
}
```

## Backwards Transitive（向后传递）

```
record Person {  
   string name; // example: "Joan Smith"  
   Name   person_name = {"first_name":"<NOT_IN_USE>","last_name":"<NOT_IN_USE>"};  
}  
record Name {  
   string first_name; // example: "Joan"  
   string last_name;  // example: "Smith"  
}
```

## Forwards（向前兼容）

```
record Person {  
   @aliases(["name"])  
   Name person_name;  
}  
record Name {  
   string first_name; // example: "Joan"  
   string last_name;  // example: "Smith"  
}
```

## Forwards Transitive（向前传递）

```
record Person {  
   string name        = "<DEPRECATED>"; // example: "Joan Smith"  
   Name   person_name = {"first_name":"<NOT_IN_USE>","last_name":"<NOT_IN_USE>"};  
}  
record Name {  
   string first_name; // example: "Joan"  
   string last_name;  // example: "Smith"  
}
```

## Full（全兼容）

```
record Person {  
   @aliases(["name"])  
   Name person_name;  
}  
record Name {  
   string first_name; // example: "Joan"  
   string last_name;  // example: "Smith"  
}
```

## Full Transitive（全传递）

```
record Person {  
   string name        = "<DEPRECATED>"; // example: "Joan Smith"  
   Name   person_name = {"first_name":"<NOT_IN_USE>","last_name":"<NOT_IN_USE>"};  
}  
record Name {  
   string first_name; // example: "Joan"  
   string last_name;  // example: "Smith"  
}
```

如果发现本文有用或在项目中使用 Apache Avro，请点赞收藏！

#### 引用链接

`[1]` Apache Avro: *[https://avro.apache.org/](https://avro.apache.org/)*