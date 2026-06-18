---
title: DuckDB 插件开发实战：Mac 版 Everything
author: MaxAiDB
date: maxaidbmaxaidb
url: https://mp.weixin.qq.com/s?__biz=MzUwOTU4OTU2NQ==&mid=2247483735&idx=1&sn=fddf7d7df8422cf5c6e347b6c00d5b1e&chksm=f872f988f7c862f0a364d7f98518ca66826129cb2cb51c71f3187fa740213d64fe6069615624&mpshare=1&scene=24&srcid=0428mzXl5SxXs3W8hgVJgyyw&sharer_shareinfo=1f80b235b666de6e058fac087d30bf62&sharer_shareinfo_first=1f80b235b666de6e058fac087d30bf62#rd
---

# 用 SQL 查文件——Mac 版 Everything 完整体验

**—— 从产品效果到代码，一篇看全**

> 本文是《DuckDB 插件开发实战：Mac 版 Everything》系列的第 1 篇。整个系列以一个真实的 DuckDB 扩展——apfs 插件（简单版本everything）为案例，带你从零走完插件开发的完整链路。这篇先体验效果，再看项目长什么样。

---

## Mac 版 Everything 是什么？

Windows 上有个神器叫 Everything——敲个文件名，毫秒级返回全盘搜索结果。Mac 上呢？Spotlight 算半个，但它没法做这种事：

* • "我磁盘上最大的 10 个文件是什么？"
* • "哪种文件类型占了最多空间？"
* • "有没有同名同大小的重复文件？"

Spotlight 能搜文件名，但**做不了分析**。你需要的不是搜索框，是一个数据库。

这就是 apfs 扩展做的事——**把你的整个 macOS 文件系统变成一张 SQL 表**。一句 `SELECT`，文件名、路径、大小、时间戳、权限，全在手边。

它还有一个配套的 Web UI——Mac Everything：

APFS Scanner Web UI

上图是实际运行截图：**658 万文件已索引**，搜索响应 396ms，支持文件类型过滤（All / Files / Folders / Images / Documents / Code）、多列排序、右键操作。这个 Web UI 的实现会在第 5 篇详细拆解，这篇先聚焦 DuckDB 扩展本身。

整个项目的技术栈：

```
Mac Everything 技术栈

HTTP JSON

CLI 子进程

浏览器前端index.html ~1000 行

Go 服务端~1700 行

DuckDB + apfs 扩展C++ ~1200 行

macOS 系统 API

fts(3) POSIX 遍历

MDQuery Spotlight 索引

CoreFoundation
```

---

## 三步跑通

### 第 1 步：编译

```
cd /path/to/duckdb-apfs  
  
# Debug 版（开发用，约 5-10 分钟首次编译）  
make debug  
  
# 或 Release 版（推荐实际使用，查询快 22-77 倍）  
make
```

编译产物是一个带 apfs 扩展的 DuckDB CLI，直接启动就能用，不需要额外 `LOAD`。

### 第 2 步：启动

```
./build/debug/duckdb
```

### 第 3 步：查

```
-- 搜索文件名包含 "duckdb" 的文件  
SELECT * FROM apfs_search('duckdb') LIMIT 5;
```

就这样。没有配置文件，没有服务器，没有 pip install——一个二进制文件搞定。

---

## apfs 扩展提供了什么？

三个 SQL 函数，覆盖"扫描"、"搜索"、"变更检测"三个场景：

| 函数 | 类型 | 用途 |
| --- | --- | --- |
| `apfs_scan(path, mode)` | 表函数 | 递归扫描目录，返回所有文件的元数据 |
| `apfs_search(keyword)` | 表函数 | 基于 Spotlight 索引的全盘文件名搜索 |
| `apfs_has_changes(path, since)` | 标量函数 | 检测目录内是否有文件变更 |

三个函数返回相同的 **11 列 schema**：

```
path        │ 完整路径  
name        │ 文件名  
extension   │ 扩展名（不含 .）  
size        │ 文件大小（字节）  
file_type   │ file / directory / symlink / other  
modified_at │ 最后修改时间  
created_at  │ 创建时间  
accessed_at │ 最后访问时间  
permissions │ 权限字符串（如 rwxr-xr-x）  
owner       │ 所有者用户名  
depth       │ 相对扫描根目录的深度
```

---

## 两种扫描模式：快 vs 准

apfs\_scan 有两种工作模式，这是设计上的一个关键取舍：

```
apfs_scan(path, mode)

mode = 'fts'

POSIX fts(3) 遍历

物理遍历每个文件

100% 文件覆盖

30-120秒 全盘

mode = 'spotlight'

macOS MDQuery API

查 macOS 索引数据库

只含已索引文件

1-3秒 全盘
```

|  | fts 模式 | spotlight 模式 |
| --- | --- | --- |
| 速度 | 全盘 30-120 秒 | 全盘 1-3 秒 |
| 覆盖 | 100% 可访问文件 | 仅 Spotlight 已索引的文件 |
| 权限 | 受保护目录需要"完全磁盘访问权限" | 无需额外权限 |
| 适合 | 需要绝对精确、扫描 `/tmp` 等未索引目录 | 日常使用、快速分析 |

**建议**：日常用 spotlight，需要 100% 覆盖时切 fts。

### Debug vs Release：性能差距有多大？

实测数据（M2 MacBook Pro 16GB，macOS，duckdb-apfs）：

| 操作 | Debug (`-O0`, 573MB) | Release (`-O2`, 43MB) | 加速比 |
| --- | --- | --- | --- |
| 进程启动 `SELECT 1` | 0.099s | 0.002s | **50x** |
| fts 扫描 `/tmp`（214 文件） | 0.160s | 0.016s | **10x** |
| fts 扫描 `~/Downloads`（10.5 万文件） | 9.39s | 1.64s | **5.7x** |
| 建表（10.5 万文件） | 7.79s | 1.84s | **4.2x** |
| 查询：扩展名聚合 TOP 5 | 0.774s | 0.010s | **77x** |
| 查询：`ILIKE '%config%'` | 0.427s | 0.019s | **22x** |
| `apfs_search('duckdb')` 全盘 | 21.06s | 19.60s | 1.1x |

几个值得注意的点：

1. 1. **纯查询差距最大**——聚合查询 77 倍、ILIKE 搜索 22 倍。查询是纯 CPU 密集的，Release 的 `-O2` 优化 + SIMD + 内联全面生效。
2. 2. **扫描差距较小**——fts 扫描是 I/O bound（`stat()` 系统调用），编译器优化帮不了太多，但仍有 4-6 倍差距（来自 DataChunk 填充和 UTF-8 处理的优化）。
3. 3. **apfs\_search 几乎没差距**——Spotlight MDQuery 是 macOS 系统调用，瓶颈在操作系统索引层，不在 DuckDB 代码里。

**结论：日常使用务必用 release 版。Debug 只在开发调试 C++ 扩展代码时使用。**

---

## 三个实战场景

### 场景 1：找出磁盘上最大的 10 个文件

```
-- 先扫描 Downloads 目录建表  
CREATE TABLE files AS  
SELECT*FROM apfs_scan('/Users/max/Downloads', 'fts');  
  
-- 找最大的 10 个文件  
SELECT  
  name,  
  ROUND(size /1024.0/1024.0, 2) AS mb,  
  extension,  
  modified_at  
FROM files  
WHERE file_type ='file'  
ORDERBY size DESC  
LIMIT 10;
```

### 场景 2：按扩展名统计磁盘占用

```
SELECT  
  extension,  
  COUNT(*) AS cnt,  
  ROUND(SUM(size) / 1024.0 / 1024.0, 2) AS mb  
FROM files  
WHERE file_type = 'file'  
GROUP BY extension  
ORDER BY mb DESC  
LIMIT 10;
```

### 场景 3：找出可能的重复文件

```
SELECT  
  name,  
  ROUND(size /1024.0/1024.0, 2) AS mb,  
COUNT(*) AS copies  
FROM files  
WHERE file_type ='file'AND size >1024*1024-- 只看大于 1MB 的  
GROUPBY name, size  
HAVING copies >1  
ORDERBY size DESC  
LIMIT 10;
```

同名同大小——大概率是重复下载或者复制遗留。

---

## apfs\_search：全盘即时搜索

如果你只是想快速找个文件，不需要先建表：

```
-- 搜索所有文件名包含 "config" 的文件  
SELECT name, path, ROUND(size/1024.0, 1) AS kb  
FROM apfs_search('config')  
LIMIT 20;  
  
-- 搜索所有 PDF 文件  
SELECT name, path, ROUND(size/1024.0/1024.0, 2) AS mb  
FROM apfs_search('*.pdf')  
ORDER BY size DESC  
LIMIT 10;
```

`apfs_search` 走的是 Spotlight 索引，全盘搜索通常在几秒内返回。

---

## apfs\_has\_changes：轻量变更检测

```
SELECT apfs_has_changes(  
  '/Users/max/Documents',  
  TIMESTAMP '2026-04-17 00:00:00'  
);  
-- 返回 true 或 false
```

这个函数不遍历文件系统，而是用一个轻量的 Spotlight 查询检查有没有任何文件的修改时间在给定时间之后。耗时大约 50-200ms。第 4 篇会详细拆解它的实现原理。

---

## DuckDB 扩展项目长什么样？

看完效果，来看看 duckdb-apfs 的项目结构：

```
duckdb-apfs/  
├── src/                          # C++ 扩展源码（5 个 .cpp + 5 个 .hpp，共 ~1200 行）  
│   ├── apfs_extension.cpp        # 扩展入口，函数注册（180 行）  
│   ├── apfs_fts_scanner.cpp      # fts 模式实现  
│   ├── apfs_spotlight_scanner.cpp # Spotlight 模式实现  
│   ├── apfs_has_changes.cpp      # 变更检测  
│   ├── apfs_utils.cpp            # 工具函数  
│   └── include/                  # 头文件  
├── test/sql/apfs/                # 8 个 sqllogictest 测试文件（574 行）  
├── duckdb/                       # DuckDB 源码（git submodule）  
├── extension-ci-tools/           # CI 工具链（git submodule）  
├── CMakeLists.txt                # 37 行，构建配置  
├── Makefile                      # 8 行，快捷命令  
├── extension_config.cmake        # 6 行，扩展注册  
└── vcpkg.json                    # 依赖管理
```

这不是随便画的——就是 DuckDB 官方 extension-template 的标准结构。所有第三方扩展（httpfs、spatial、delta 等）都长这样。

---

## 四个关键文件

一个 DuckDB 扩展项目，核心就四个配置文件 + 你的 C++ 代码。

### 1. `extension_config.cmake`——告诉 DuckDB "我有个扩展"

```
# extension_config.cmake（全部就 6 行）  
duckdb_extension_load(apfs  
    SOURCE_DIR ${CMAKE_CURRENT_LIST_DIR}  
    LOAD_TESTS  
)
```

`duckdb_extension_load` 宏做三件事：把 `apfs` 注册为扩展名、指定源码目录、`LOAD_TESTS` 让 `make test` 能发现测试文件。

### 2. `CMakeLists.txt`——怎么编译

```
cmake_minimum_required(VERSION 2.8.12...3.29)  
  
set(TARGET_NAME apfs)  
set(EXTENSION_NAME ${TARGET_NAME}_extension)  
set(LOADABLE_EXTENSION_NAME ${TARGET_NAME}_loadable_extension)  
  
project(${TARGET_NAME})  
include_directories(src/include)  
  
set(EXTENSION_SOURCES  
    src/apfs_extension.cpp  
    src/apfs_fts_scanner.cpp  
    src/apfs_spotlight_scanner.cpp  
    src/apfs_has_changes.cpp  
    src/apfs_utils.cpp)  
  
build_static_extension(${TARGET_NAME}${EXTENSION_SOURCES})  
set(PARAMETERS "-warnings")  
build_loadable_extension(${TARGET_NAME}${PARAMETERS}${EXTENSION_SOURCES})  
  
# macOS 框架链接  
if(APPLE)  
    find_library(CORE_FOUNDATION_LIB CoreFoundation)  
    find_library(CORE_SERVICES_LIB CoreServices)  
    if(CORE_FOUNDATION_LIB AND CORE_SERVICES_LIB)  
        target_link_libraries(${EXTENSION_NAME}  
            ${CORE_FOUNDATION_LIB}${CORE_SERVICES_LIB})  
        target_link_libraries(${LOADABLE_EXTENSION_NAME}  
            ${CORE_FOUNDATION_LIB}${CORE_SERVICES_LIB})  
    endif()  
endif()
```

两个关键宏：

* • **`build_static_extension`**：生成静态库 `.a`，链接进 `duckdb` CLI 二进制
* • **`build_loadable_extension`**：生成共享库 `.duckdb_extension`，可通过 `LOAD` 动态加载

apfs 扩展额外链接了 macOS 的 `CoreFoundation` 和 `CoreServices` 框架——因为 Spotlight API 在这两个框架里。

### 3. `Makefile`——快捷入口

```
PROJ_DIR := $(dir $(abspath $(lastword $(MAKEFILE_LIST))))  
EXT_NAME=apfs  
EXT_CONFIG=${PROJ_DIR}extension_config.cmake  
  
include extension-ci-tools/makefiles/duckdb_extension.Makefile
```

就 8 行。所有编译逻辑都在 `extension-ci-tools` 里，你不需要自己写。

### 4. `src/include/apfs_extension.hpp`——扩展类定义

```
#pragma once  
#include "duckdb.hpp"  
  
namespace duckdb {  
  
class ApfsExtension : public Extension {  
public:  
    void Load(ExtensionLoader &loader) override;  
    std::string Name() override;  
    std::string Version() const override;  
};  
  
} // namespace duckdb
```

每个 DuckDB 扩展都继承 `Extension` 基类，实现三个方法：

* • `Load()`：注册你的函数——这是扩展的"main"
* • `Name()`：返回扩展名（"apfs"）
* • `Version()`：返回版本号

---

## 编译流程

```
make debug

Makefile → extension-ci-tools

CMake 配置读取 extension_config.cmake

编译 DuckDB 源码duckdb/ 子目录，首次约 5-10 分钟

编译 apfs 扩展源码src/*.cpp

build_static_extension→ apfs_extension.a

build_loadable_extension→ apfs.duckdb_extension

链接进 build/debug/duckdbCLI 二进制

独立文件可用

LOAD 加载
```

编译产物：

| 产物 | 路径 | 说明 |
| --- | --- | --- |
| DuckDB CLI（内含扩展） | `build/debug/duckdb` （573MB） | 直接用，不需要 LOAD |
|  | `build/release/duckdb` （43MB） | 推荐实际使用 |
| 动态扩展 | `build/debug/extension/apfs/apfs.duckdb_extension` | 给已有 DuckDB 加载 |

**首次编译会很慢**（要编译整个 DuckDB 源码），后续只改 `src/` 下的文件，增量编译只需几秒。

加速技巧：

```
# 用 Ninja 替代 Make（并行度更好）  
GEN=ninja make debug  
  
# 只编译 apfs 扩展（跳过其他扩展）  
BUILD_EXTENSIONS="apfs" make debug
```

---

## 两个 git submodule

项目依赖两个子模块：

```
# .gitmodules  
[submodule "duckdb"]  
    path = duckdb  
    url = https://github.com/duckdb/duckdb.git  
[submodule "extension-ci-tools"]  
    path = extension-ci-tools  
    url = https://github.com/duckdb/extension-ci-tools.git
```

`duckdb/` 子模块指向一个特定的 DuckDB 版本。**扩展版本必须与 DuckDB 版本严格匹配**——这是 DuckDB 扩展生态的一个基本约束。

---

## 扩展入口：LoadInternal

扩展的"main 函数"在 `src/apfs_extension.cpp`：

```
static void LoadInternal(ExtensionLoader &loader) {  
    // 1. 注册 apfs_scan（两个重载：1 参数和 2 参数）  
    TableFunctionSet scan_set("apfs_scan");  
    TableFunction scan_one_arg("apfs_scan", {LogicalType::VARCHAR},  
        ApfsScanFunction, ApfsScanBindOneArg, ApfsScanInit);  
    scan_one_arg.named_parameters["since"] = LogicalType::TIMESTAMP;  
    scan_set.AddFunction(scan_one_arg);  
  
    TableFunction scan_two_args("apfs_scan",  
        {LogicalType::VARCHAR, LogicalType::VARCHAR},  
        ApfsScanFunction, ApfsScanBindTwoArgs, ApfsScanInit);  
    scan_two_args.named_parameters["since"] = LogicalType::TIMESTAMP;  
    scan_set.AddFunction(scan_two_args);  
    loader.RegisterFunction(scan_set);  
  
    // 2. 注册 apfs_search  
    TableFunction search_func("apfs_search", {LogicalType::VARCHAR},  
        ApfsSearchFunction, ApfsSearchBindWrapper, ApfsSearchInit);  
    loader.RegisterFunction(search_func);  
  
    // 3. 注册 apfs_has_changes（标量函数）  
    loader.RegisterFunction(CreateApfsHasChangesFunction());  
}
```

三个函数就这样注册完了。`loader.RegisterFunction()` 告诉 DuckDB："以后有人调 `apfs_scan`，用我这几个回调函数处理。"

文件末尾还有一个 C 入口点——动态加载时 DuckDB 通过 `dlsym` 找到它：

```
extern "C" {  
DUCKDB_CPP_EXTENSION_ENTRY(apfs, loader) {  
    duckdb::LoadInternal(loader);  
}  
}
```

---

## 验证扩展是否加载

```
SELECT * FROM duckdb_extensions() WHERE extension_name = 'apfs';
```

```
┌────────────────┬─────────┬───────────┬──────────────┐  
│ extension_name │ loaded  │ installed │ install_path │  
├────────────────┼─────────┼───────────┼──────────────┤  
│ apfs           │ true    │ true      │ (BUILT-IN)   │  
└────────────────┴─────────┴───────────┴──────────────┘
```

`(BUILT-IN)` 表示静态链接——编译时就内嵌进了 CLI 二进制。

---

## 如果你想从零开始

DuckDB 官方提供了扩展模板仓库，一键生成脚手架：

```
# 在 https://github.com/duckdb/extension-template 点 "Use this template"  
# 或手动克隆  
git clone --recurse-submodules https://github.com/duckdb/extension-template.git my_extension  
cd my_extension  
  
# 修改扩展名（模板默认叫 "quack"）  
# 1. extension_config.cmake 里改名  
# 2. CMakeLists.txt 里改 TARGET_NAME  
# 3. src/ 下的文件名和类名  
  
make debug  
./build/debug/duckdb -c "SELECT quack('world');"  
# Hello, world! 🐥
```

模板自带一个 `quack()` 标量函数作为示例——改改就是你的扩展。

## 总结

三个要点：

1. 1. **apfs 扩展把 macOS 文件系统变成了 SQL 表**——11 列元数据，支持全部 SQL 操作
2. 2. **两种扫描模式**：fts（慢但全覆盖）和 spotlight（秒级但依赖索引），按需选择
3. 3. **这是一个完整的 DuckDB 扩展项目**——C++ 插件 ~1200 行 + Go Web UI ~1700 行 + 完整测试，接下来我们逐层拆解

下一篇，我们从零开始构建 apfs 插件，完整走一遍决策过程。

---

## 参考资料

* • DuckDB Extension Template
* • DuckDB 官方文档 - Extensions Overview
* • Windows Everything — apfs 扩展的灵感来源
* • duckdb-apfs 源码：`extension_config.cmake`（6 行，扩展注册）
* • duckdb-apfs 源码：`CMakeLists.txt`（38 行，完整构建配置）
* • duckdb-apfs 源码：`src/apfs_extension.cpp`（180 行，扩展入口 + 3 个函数注册）