---
title: 每天学习一点TS-Day12-命名空间
author: Tech Information快递猿
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkxMjcwOTAxMA==&mid=2247486313&idx=3&sn=bdc1ba7107d3506f934f2093ba85dac8&chksm=c08078d964ade817a488f8ef7dba00e8425f53096ce3353e1642b30b03179a065202e4be9dc5&mpshare=1&scene=24&srcid=0510rWYoCePXQ4tALi3fZcgZ&sharer_shareinfo=320c36114d23f8579714afa7764c109d&sharer_shareinfo_first=320c36114d23f8579714afa7764c109d#rd
---

> 内部模块、多文件组织、编译、别名、与模块的区别

# Day 12: 命名空间 (Namespaces)

命名空间（以前称为"内部模块"）是 TypeScript 中组织代码的一种方式，用于将相关的代码封装在一起。

## 命名空间基础

```
namespace Validation {  
    export interface StringValidator {  
        isAcceptable(s: string): boolean;  
    }  
      
    const lettersRegexp = /^[A-Za-z]+$/;  
    const numberRegexp = /^[0-9]+$/;  
      
    export class LettersOnlyValidator implements StringValidator {  
        isAcceptable(s: string) {  
            return lettersRegexp.test(s);  
        }  
    }  
      
    export class ZipCodeValidator implements StringValidator {  
        isAcceptable(s: string) {  
            return s.length === 5 && numberRegexp.test(s);  
        }  
    }  
}  
  
// 使用命名空间  
let strings = ["Hello", "98052", "101"];  
let validators: { [s: string]: Validation.StringValidator } = {};  
  
validators["ZIP code"] = new Validation.ZipCodeValidator();  
validators["Letters only"] = new Validation.LettersOnlyValidator();
```

**关键点**：

* 使用 `namespace` 关键字定义
* 需要导出的成员使用 `export`
* 未导出的成员在命名空间外不可见

## 分离到多文件

命名空间可以跨多个文件：

```
// Validation.ts  
namespace Validation {  
    export interface StringValidator {  
        isAcceptable(s: string): boolean;  
    }  
}  
  
// LettersOnlyValidator.ts  
/// <reference path="Validation.ts" />  
namespace Validation {  
    const lettersRegexp = /^[A-Za-z]+$/;  
      
    export class LettersOnlyValidator implements StringValidator {  
        isAcceptable(s: string) {  
            return lettersRegexp.test(s);  
        }  
    }  
}  
  
// ZipCodeValidator.ts  
/// <reference path="Validation.ts" />  
namespace Validation {  
    const numberRegexp = /^[0-9]+$/;  
      
    export class ZipCodeValidator implements StringValidator {  
        isAcceptable(s: string) {  
            return s.length === 5 && numberRegexp.test(s);  
        }  
    }  
}  
  
// Test.ts  
/// <reference path="Validation.ts" />  
/// <reference path="LettersOnlyValidator.ts" />  
/// <reference path="ZipCodeValidator.ts" />  
  
let validators: { [s: string]: Validation.StringValidator } = {};  
validators["ZIP code"] = new Validation.ZipCodeValidator();
```

**注意**：使用 `/// <reference path="..." />` 声明文件依赖。

## 编译命名空间

### 方式一：编译为单个文件

```
tsc --outFile sample.js Validation.ts LettersOnlyValidator.ts ZipCodeValidator.ts Test.ts
```

或使用 `tsconfig.json`：

```
{  
    "compilerOptions": {  
        "outFile": "sample.js"  
    },  
    "files": [  
        "Validation.ts",  
        "LettersOnlyValidator.ts",  
        "ZipCodeValidator.ts",  
        "Test.ts"  
    ]  
}
```

### 方式二：编译为多个文件

```
tsc Validation.ts LettersOnlyValidator.ts ZipCodeValidator.ts Test.ts
```

然后在 HTML 中按正确顺序引入：

```
<script src="Validation.js"></script>  
<script src="LettersOnlyValidator.js"></script>  
<script src="ZipCodeValidator.js"></script>  
<script src="Test.js"></script>
```

## 别名

为命名空间成员创建短名：

```
namespace Shapes {  
    export namespace Polygons {  
        export class Triangle { }  
        export class Square { }  
    }  
}  
  
// 创建别名  
import polygons = Shapes.Polygons;  
  
let sq = new polygons.Square();  // 等同于 new Shapes.Polygons.Square()
```

**注意**：这里的 `import` 不是模块导入，只是别名语法。

## 命名空间 vs 模块

### 命名空间

```
// 全局作用域  
namespace MyNamespace {  
    export class MyClass { }  
}  
  
// 使用  
let obj = new MyNamespace.MyClass();
```

### 模块

```
// 文件作用域  
export class MyClass { }  
  
// 另一个文件  
import { MyClass } from "./MyModule";  
let obj = new MyClass();
```

### 选择建议

| 场景 | 推荐方式 |
| --- | --- |
| 浏览器应用（无模块加载器） | 命名空间 |
| Node.js 应用 | 模块 |
| 现代前端项目 | 模块 |
| 需要在全局暴露的库 | 命名空间 |
| 已有代码是命名空间风格 | 命名空间 |

## 命名空间与外部库

为外部库编写命名空间声明：

```
// D3.d.ts  
declare namespace D3 {  
    export interface Selectors {  
        select: {  
            (selector: string): Selection;  
            (element: EventTarget): Selection;  
        };  
    }  
      
    export interface Event {  
        x: number;  
        y: number;  
    }  
      
    export interface Base extends Selectors {  
        event: Event;  
    }  
}  
  
declare var d3: D3.Base;
```

使用：

```
/// <reference path="D3.d.ts" />  
  
d3.select("body").style("background-color", "white");
```

## 模块内使用命名空间

```
// MyModule.ts  
export namespace MyNamespace {  
    export class MyClass {  
        sayHello() {  
            console.log("Hello!");  
        }  
    }  
}  
  
// Consumer.ts  
import { MyNamespace } from "./MyModule";  
let obj = new MyNamespace.MyClass();
```

**注意**：现代 TypeScript 项目中，这种方式很少使用。

## 命名空间的最佳实践

### 何时使用命名空间

1. **遗留项目维护**：已有代码使用命名空间
2. **全局库开发**：需要在全局作用域暴露 API
3. **类型声明文件**：为 JavaScript 库编写 `.d.ts`

### 避免使用命名空间的情况

1. ✅ **新项目**：优先使用 ES 模块
2. ✅ **Node.js 应用**：使用模块系统
3. ✅ **现代前端框架**：Vue/React/Angular 都使用模块

### 常见误区

```
// ❌ 错误：在模块中使用命名空间  
export namespace Foo {  
    export function bar() {}  
}  
  
// ✅ 正确：直接使用导出  
export function bar() {}
```

## 总结

| 特性 | 命名空间 | 模块 |
| --- | --- | --- |
| 关键字 | `namespace` | `import`/`export` |
| 作用域 | 全局/局部 | 文件级 |
| 依赖声明 | `/// <reference>` | `import` |
| 代码生成 | 单一文件/多个文件 | 每个文件独立 |
| 推荐使用 | ⚠️ 特定场景 | ✅ 首选 |

**现代 TypeScript 建议**：

1. 新项目优先使用 ES 模块
2. 命名空间主要用于类型声明文件
3. 避免在模块中使用命名空间
4. 使用模块路径管理依赖关系

明天我们将学习 TypeScript 中的装饰器！

---

*每天学习一点 TS，积少成多，你会成为 TypeScript 高手！*