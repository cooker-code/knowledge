> 已吸收至：[[04_OLAP与数据库/0403_嵌入式分析/040301_DuckDB/040301_核心知识点/DuckDB向量化执行与Pipeline|DuckDB向量化执行与Pipeline]]
---
title: 深入分析DuckDB的向量化执行
author: Hello大数据
date:
url: https://mp.weixin.qq.com/s?__biz=MzA5MDE0MDE1OQ==&mid=2247484255&idx=1&sn=4c110195b4b2353b923d2c6f9b5f59e7&chksm=91d60ae05c960abe7bddad124baa9d18d4900cbc15ab0a3a0b0fa290b0228bfc1a9af077efd5&mpshare=1&scene=24&srcid=032659Krjq2XQeBLtwOJ7dMU&sharer_shareinfo=bafedcee48db1c40b5eafee6a907cc56&sharer_shareinfo_first=bafedcee48db1c40b5eafee6a907cc56#rd
---

DuckDB的向量化执行是其高性能查询处理的核心机制，通过批量处理数据来最大化CPU缓存利用率和并行效率。

## 核心数据结构

### Vector - 向量化执行的基础

`Vector`类是向量化执行的基石，代表一列数据的批量存储 。

```
class Vector {  
    VectorType vector_type;        // 向量类型：指定数据的物理存储方式（如FLAT_VECTOR、CONSTANT_VECTOR、DICTIONARY_VECTOR等）  
    LogicalType type;              // 逻辑类型：向量中存储元素的数据类型（如INTEGER、VARCHAR、STRUCT等）  
    data_ptr_t data;               // 数据指针：指向实际数据存储位置的指针  
    ValidityMask validity;         // 有效性掩码：用于跟踪NULL值的位掩码  
    unique_ptr<VectorBuffer> buffer;      // 主缓冲区：持有向量主要数据的VectorBuffer  
    unique_ptr<VectorBuffer> auxiliary;   // 辅助缓冲区：持有向量辅助数据的VectorBuffer（如字符串堆、结构体子向量等）  
};
```

**关键特性：**

* • **批量处理**：默认`STANDARD_VECTOR_SIZE`为2048，这是一个编译时常量，如果需要修改，需要在编译时设置，运行时无法修改。且必须是2的幂
* • **多种向量类型**：FLAT\_VECTOR、CONSTANT\_VECTOR、DICTIONARY\_VECTOR等

### ValidityMask

ValidityMask是DuckDB向量化执行系统中用于跟踪NULL值的核心组件，设计非常巧妙，它是一个高效的位掩码结构，每个bit对应向量中的一个元素，标记该位置是否为有效值（非NULL） 。ValidityMask使用uint64\_t作为基础类型，每个64位整数可以跟踪64个元素的有效性

```
//! Type used for validity masks
template <typename V>
struct TemplatedValidityMask {
    using ValidityBuffer = TemplatedValidityData<V>;

public:
    static constexpr const idx_t BITS_PER_VALUE = ValidityBuffer::BITS_PER_VALUE;
    static constexpr const idx_t STANDARD_ENTRY_COUNT = (STANDARD_VECTOR_SIZE + (BITS_PER_VALUE - 1)) / BITS_PER_VALUE;
    static constexpr const idx_t STANDARD_MASK_SIZE = STANDARD_ENTRY_COUNT * sizeof(V);
    
..............
    
    
protected:
    V *validity_mask;
    buffer_ptr<ValidityBuffer> validity_data;
    idx_t capacity;
```

每个元素的有效性通过位运算确定：

```
// 计算元素在位掩码中的位置  
idx_t entry_idx = row_idx / BITS_PER_VALUE;     // 哪个64位整数  
idx_t idx_in_entry = row_idx % BITS_PER_VALUE;  // 该整数的哪一位  
  
// 检查有效性  
bool is_valid = validity_mask[entry_idx] & (1 << idx_in_entry);
```

### DataChunk - 数据块容器

DataChunk是DuckDB向量化执行系统中的核心数据结构，代表一个关系子集，包含多个具有相同行数的向量

```
class DataChunk {  
public:  
    vector<Vector> data;              // 向量集合，每个向量代表一列  
      
private:  
    idx_t count;                      // 当前存储的元组数量  
    idx_t capacity;                   // 可存储的元组容量  
    idx_t initial_capacity;           // 初始化时设置的初始容量  
    vector<VectorCache> vector_caches; // 向量缓存，用于存储数据  
};
```

### SelectionVector

SelectionVector是DuckDB向量化执行系统中的核心组件，用于高效的数据过滤和切片操作，避免不必要的数据复制。SelectionVector是一个索引数组，通常指向Vector中的特定位置  。它允许在不复制实际数据的情况下重新排序、过滤或切片向量数据。

## 向量化操作体系

### VectorOperations - 核心操作库

`VectorOperations`提供了完整的向量化操作集合：

```
struct VectorOperations {
    // 算术操作
    static void AddInPlace(Vector &left, int64_t delta, idx_t count);
    
    // 比较操作  
    static void Equals(Vector &left, Vector &right, Vector &result, idx_t count);
    static void GreaterThan(Vector &left, Vector &right, Vector &result, idx_t count);
    
    // 逻辑操作
    static void And(Vector &left, Vector &right, Vector &result, idx_t count);
    static void Or(Vector &left, Vector &right, Vector &result, idx_t count);
    
    // NULL处理
    static void IsNotNull(Vector &arg, Vector &result, idx_t count);
    static bool HasNull(Vector &input, idx_t count);
};
```

## 统一向量格式系统

### UnifiedVectorFormat - 高效数据访问

`UnifiedVectorFormat`提供了统一的数据访问接口，屏蔽不同向量类型的差异 。

```
struct UnifiedVectorFormat {
    const SelectionVector *sel;
    data_ptr_t data;
    ValidityMask validity;
    PhysicalType physical_type;
    SelectionVector owned_sel;
};
```

**转换机制：**

```
void Vector::ToUnifiedFormat(idx_t count, UnifiedVectorFormat &data) {
    // 将不同类型的向量转换为统一格式
    // 支持FLAT、CONSTANT、DICTIONARY等类型的无缝转换
        format.physical_type = GetType().InternalType();
    switch (GetVectorType()) {
    case VectorType::DICTIONARY_VECTOR: {
        ..............
        break;
    }
    case VectorType::CONSTANT_VECTOR:
 ..............
        break;
    default:
 ..............
        break;
    }
    
}
```

## 向量缓冲区管理

### VectorBufferType - 多样化缓冲区

系统支持多种缓冲区类型以优化不同场景的内存使用 ：

```
enum class VectorBufferType : uint8_t {
    STANDARD_BUFFER,     // 标准数据缓冲区
    DICTIONARY_BUFFER,   // 字典压缩缓冲区
    VECTOR_CHILD_BUFFER, // 嵌套类型子向量
    STRING_BUFFER,       // 字符串堆缓冲区
    FSST_BUFFER,         // FSST压缩字符串
    STRUCT_BUFFER,       // 结构体缓冲区
    LIST_BUFFER,         // 列表缓冲区
    MANAGED_BUFFER,      // 管理器控制的缓冲区
    ARRAY_BUFFER         // 数组缓冲区
};
```

## 特殊向量类型处理

### 字典向量 - 高效去重

`DictionaryVector`通过字典编码实现数据去重和压缩  。

```
struct DictionaryVector {
    static inline const SelectionVector &SelVector(const Vector &vector);
    static inline const Vector &Child(const Vector &vector);
    static inline optional_idx DictionarySize(const Vector &vector);
    static inline bool CanCacheHashes(const Vector &vector);
};
```

### 列表向量 - 嵌套数据支持

`ListVector`处理变长数组数据 。

```
struct ListVector {
    static inline const list_entry_t *GetData(const Vector &v);
    static const Vector &GetEntry(const Vector &vector);
    static idx_t GetListSize(const Vector &vector);
    static void Reserve(Vector &vec, idx_t required_capacity);
};
```

## 性能优化特性

### 1. 缓存友好的数据布局

* • 列式存储提高缓存命中率
* • 批量处理减少函数调用开销
* • SIMD友好的数据对齐

### 2. 零拷贝操作

* • 向量引用避免数据复制
* • 切片操作实现高效子集提取
* • 统一格式转换的优化路径

### 3. 类型特化优化

* • 针对不同数据类型的专门实现
* • 编译时类型推导和优化
* • 运行时类型检查的最小化

## 向量化执行流程

查询解析

物理计划生成

向量化操作符创建

DataChunk分配

Vector初始化

批量数据读取

向量化操作执行

结果输出

算术操作

比较操作

聚合操作

连接操作

## 向量化执行为什么会快

向量化执行主要在于它充分利用了现代CPU的架构特性，通过批量处理数据来最大化硬件效率。

### CPU缓存利用率提升

向量化执行按列存储数据，相同类型的数据连续存放，大幅提高CPU缓存命中率。

### 减少函数调用开销

传统逐行处理每行都需要函数调用，而向量化操作一次处理整个数据块。VectorOperations提供批量操作接口

### SIMD优化潜力

连续内存布局为SIMD指令优化创造条件。在TightLoopHash函数中，紧凑的循环结构便于编译器自动向量化

### 分支预测优化

向量化操作减少了条件分支，提高CPU分支预测准确率。在处理NULL值时，使用ValidityMask位掩码避免分支

### 批量哈希计算示例

TightLoopHash是DuckDB向量化哈希计算的核心函数，它通过模板特化实现高性能的批量哈希计算

restrict关键字帮助编译器优化,尽可能的转成SIMD指令来执行。

```
template <bool HAS_RSEL, bool HAS_SEL_VECTOR, class T, bool INPUT_IS_ALREADY_HASH>  
void TightLoopHash(const T *__restrict ldata, hash_t *__restrict result_data,   
                   const SelectionVector *rsel, idx_t count,  
                   const SelectionVector *__restrict sel_vector, const ValidityMask &mask) {  
    if (!mask.AllValid()) {  
        for (idx_t i = 0; i < count; i++) {  
            auto ridx = HAS_RSEL ? rsel->get_index_unsafe(i) : i;  
            auto idx = HAS_SEL_VECTOR ? sel_vector->get_index_unsafe(ridx) : ridx;  
            result_data[ridx] = INPUT_IS_ALREADY_HASH ? CachedHashOp::Operation(ldata[idx])  
                : HashOp::Operation(ldata[idx], !mask.RowIsValidUnsafe(idx));  
        }  
    } else {  
        // 无NULL值的优化路径  
        for (idx_t i = 0; i < count; i++) {  
            auto ridx = HAS_RSEL ? rsel->get_index_unsafe(i) : i;  
            auto idx = HAS_SEL_VECTOR ? sel_vector->get_index_unsafe(ridx) : ridx;  
            result_data[ridx] = INPUT_IS_ALREADY_HASH ? CachedHashOp::Operation(ldata[idx])   
                : duckdb::Hash<T>(ldata[idx]);  
        }  
    }  
}
```

## 总结

向量化处理特别适合分析型工作负载，通过充分利用现代CPU架构特性，在复杂查询处理中实现数倍于传统逐行处理的性能提升。它是DuckDB作为高性能分析数据库的核心技术基础。