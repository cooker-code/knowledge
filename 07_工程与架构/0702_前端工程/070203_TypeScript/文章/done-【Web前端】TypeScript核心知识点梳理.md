> 已吸收至：[[07_工程与架构/0702_前端工程/070203_TypeScript/070203_核心知识点/TypeScript类型边界准则|TypeScript类型边界准则]]、[[07_工程与架构/0702_前端工程/070203_TypeScript/070203_知识地图|070203_TypeScript知识地图]]

---
title: 【Web前端】TypeScript核心知识点梳理
author: 宝青书坊
date:
url: https://mp.weixin.qq.com/s?__biz=MzIzNDgyMzY2Mg==&mid=2247487972&idx=1&sn=4a6e7fe03a00be8082dc70fca7a1b806&chksm=e9c70784599eaba0c46fcaedea6a0eef155ff35c79bbe54d61e8dcd17774035d52d100aaa280&mpshare=1&scene=24&srcid=1204cWzwpL5ZBePztzDScPQA&sharer_shareinfo=e235d1d170d4540a487a7623b854213e&sharer_shareinfo_first=e235d1d170d4540a487a7623b854213e#rd
---

第一部分：TypeScript 基础与核心概念

1. TypeScript 是什么？

1）JavaScript 的超集：任何合法的 JavaScript 代码都是合法的 TypeScript 代码。

2）静态类型系统：在代码运行前进行类型检查，而 JavaScript 是动态类型。

3）编译时：TypeScript 代码需要被编译成 JavaScript 才能在浏览器或 Node.js 中运行。

4）目标：增强代码的可读性、可维护性，并在开发阶段暴露潜在错误。

2. 原始类型

对应 JavaScript 的原始数据类型。

① string：字符串

② number：数字（包括整数、浮点数、二进制、十六进制等）

③ boolean：布尔值

④ null 和 undefined：各自是自身的类型。

⑤ symbol 和 bigint：ES6 和 ES2020 新增的类型。

3. 数组类型

两种定义方式：

① 类型[]： number[], string[]

② Array<类型>： Array<number>, Array<string>

4. any、unknown 和 void

1）any： “逃脱”类型检查，相当于回到纯 JavaScript。应尽量避免使用。

2）unknown： “安全”的 any。不能直接赋值给其他变量或调用方法，必须先进行类型检查（类型收窄）。

3）void： 表示函数没有返回值。如果函数返回 undefined，类型也是 void。

5. never 类型

1）表示永远不可能存在的值的类型。

2）应用场景：

① 抛出错误的函数：function error(message: string): never { throw new Error(message); }

② 无限循环的函数。

③ 在联合类型中，never 会被自动移除。

6. 类型注解与类型推断

1）类型注解： 我们主动告诉 TypeScript 变量的类型。

```
let myName: string = "Alice";let age: number;
```

2）类型推断： TypeScript 根据上下文自动推断出变量的类型。

```
let myName = "Alice"; // TypeScript 推断为 string 类型let age = 25; // 推断为 number 类型
```

7. 函数类型

1）参数和返回值类型：

```
function add(x: number, y: number): number {    return x + y;}
```

2）可选参数： 使用 ?

```
function buildName(firstName: string, lastName?: string) { ... }
```

3）默认参数： 拥有默认值的参数也是可选的。

```
function buildName(firstName: string, lastName: string = "Smith") { ... }
```

4）剩余参数：

```
function buildName(firstName: string, ...restOfName: string[]) { ... }
```

5）函数表达式：

```
const myAdd: (x: number, y: number) => number = function(x: number, y: number): number { ... };
```

8. 对象类型与接口

1）接口：是定义对象结构的主要工具。

```
interface Person {    name: string;    age: number;    readonly id: number; // 只读属性    nickname?: string; // 可选属性}
```

2）类型别名：也可以定义对象类型。

```
type Person = {    name: string;    age: number;}
```

3）接口 vs 类型别名：

① 接口可以通过 extends 继承，类型别名使用 & 交叉类型。

② 接口可以声明合并（同名接口会自动合并），类型别名不能。

③ 在定义对象类型时，优先使用接口。

9. 元组

表示一个已知元素数量和类型的数组。

```
let x: [string, number];x = ["hello", 10]; // OKx = [10, "hello"]; // Error
```

10. 枚举

用于定义一组命名常量。

1）数字枚举（默认从0开始自增）

```
enum Direction { Up, Down, Left, Right } let dir: Direction = Direction.Up;
```

2）字符串枚举

```
enum Direction { Up = "UP", Down = "DOWN" }
```

3）常量枚举（编译后被内联，性能更好）

```
const enum Direction { ... }
```

第二部分：TypeScript 进阶特性

11. 联合类型与交叉类型

1）联合类型： |， 表示取值可以是多种类型中的一种。

```
let id: string | number;
```

2）交叉类型： &， 表示将多个类型合并为一个类型。

```
interface A { a: number }interface B { b: string }type C = A & B; // { a: number, b: string }
```

12. 类型断言

告诉编译器“你比它更了解这个类型”。

两种语法：

1）“尖括号”语法： <类型>值

2）as 语法： 值 as 类型（在 JSX 中必须使用这种）

```
let someValue: any = "this is a string";let strLength: number = (<string>someValue).length;let strLength2: number = (someValue as string).length;
```

13. 类型守卫与类型收窄

1）在条件分支中缩小类型的范围。

2）typeof 守卫： 处理原始类型。

```
if (typeof padding === "string") {    // 这里 padding 的类型被收窄为 string}
```

3）instanceof 守卫： 处理类。

4）in 守卫： 检查对象是否有某个属性。

5）字面量类型守卫： 检查 ===, !==, ==, !=。

6）自定义类型守卫： 定义一个返回 value is Type 的函数。

```
function isFish(pet: Fish | Bird): pet is Fish {    return (pet as Fish).swim !== undefined;}
```

14. 字面量类型

1）将值作为类型。

2）常与联合类型一起使用，实现枚举一样的效果。

```
type Easing = "ease-in" | "ease-out" | "ease-in-out";function animate(dx: number, dy: number, easing: Easing) { ... }animate(0, 0, "ease-in"); // OKanimate(0, 0, "uneasy"); // Error
```

15. 索引签名

用于描述那些“通过索引得到”的类型，比如对象 obj[prop]。

```
interface StringArray {    [index: number]: string; // 用数字索引时，返回字符串}
```

16. 泛型

1）核心思想： 在定义函数、接口或类的时候，不预先指定具体的类型，而在使用的时候再指定类型。

2）泛型函数：

```
function identity<T>(arg: T): T {    return arg;}let output = identity<string>("myString");let output2 = identity("myString"); // 类型推断
```

3）泛型接口：

```
interface GenericIdentityFn<T> {    (arg: T): T;}
```

4）泛型类：

```
class GenericNumber<T> {    zeroValue: T;    add: (x: T, y: T) => T;}
```

5）泛型约束： 使用 extends 关键字限制泛型的范围。

```
interface Lengthwise { length: number; }function loggingIdentity<T extends Lengthwise>(arg: T): T {    console.log(arg.length);    return arg;}
```

第三部分：TypeScript 高级主题与工程化

17. 关键字

1）keyof： 获取某种类型的所有键，其返回类型是联合类型。

```
interface Person { name: string; age: number; }type K1 = keyof Person; // "name" | "age"
```

2）typeof（在类型上下文中）： 用于获取变量或属性的类型。

```
let s = "hello";let n: typeof s; // n 的类型是 string
```

3）in（在映射类型中）： 用于遍历联合类型。

```
type Keys = "option1" | "option2";type Flags = { [K in Keys]: boolean }; // { option1: boolean, option2: boolean }
```

18. 实用工具类型

TypeScript 提供了一系列内置的实用工具类型，这些类型可以帮助开发者更方便地进行类型转换和操作。

1）Partial<T>

将类型 T 的所有属性设置为可选属性。这在需要创建部分更新对象或可选配置时非常有用。

```
interface User {  id: number;  name: string;  age: number;}
type PartialUser = Partial<User>;// 等价于 { id?: number; name?: string; age?: number; }
```

2）Required<T>

与 Partial 相反，将类型 T 的所有属性设置为必选属性。适用于强制要求完整对象的情况。

```
interface OptionalUser {  id?: number;  name?: string;}
type RequiredUser = Required<OptionalUser>;// 等价于 { id: number; name: string; }
```

3）Readonly<T>

将类型 T 的所有属性设置为只读属性，防止对象被修改。

```
interface Config {  apiUrl: string;  timeout: number;}
type ReadonlyConfig = Readonly<Config>;// 等价于 { readonly apiUrl: string; readonly timeout: number; }
```

4）Record<K, T>

构造一个类型，其属性名为 K 类型，属性值为 T 类型。常用于创建字典或映射类型。

```
type UserRoles = Record<string, boolean>;// 等价于 { [key: string]: boolean; }
const roles: UserRoles = {  admin: true,  editor: false};
```

5）Pick<T, K>

从类型 T 中挑选出指定的属性 K 来构造新类型。K 可以是字符串字面量或字符串字面量联合类型。

```
interface User {  id: number;  name: string;  age: number;  email: string;}
type UserBasicInfo = Pick<User, 'id' | 'name'>;// 等价于 { id: number; name: string; }
```

6）Omit<T, K>

从类型 T 中排除指定的属性 K 来构造新类型。与 Pick 相反。

```
type UserWithoutId = Omit<User, 'id'>;// 等价于 { name: string; age: number; email: string; }
```

7）Exclude<T, U>

从类型 T 中排除那些可以赋值给 U 的类型。通常用于联合类型。

```
type T = 'a' | 'b' | 'c' | 'd';type U = 'a' | 'c';
type Result = Exclude<T, U>; // 'b' | 'd'
```

8）Extract<T, U>

从类型 T 中提取那些可以赋值给 U 的类型。与 Exclude 相反。

```
type Extracted = Extract<T, U>; // 'a' | 'c'
```

9）NonNullable<T>

从类型 T 中排除 null 和 undefined 类型。

```
type MaybeString = string | null | undefined;type DefinitelyString = NonNullable<MaybeString>; // string
```

这些工具类型可以单独使用，也可以组合使用来创建更复杂的类型定义，极大地提高了 TypeScript 的类型操作灵活性。

19. 声明文件

用于为现有的 JavaScript 库提供类型定义，文件后缀为 .d.ts。

1）声明语法：

① 声明变量： declare let jQuery: (selector: string) => any;

② 声明函数： declare function greet(message: string): void;

③ 声明类： declare class Animal { ... }

④ 声明枚举： declare enum Direction { ... }

⑤ 声明命名空间/模块： declare namespace MyLib { ... }

2）安装社区类型定义：

通常使用 @types/\* 包，如 npm install --save-dev @types/lodash。

20. 模块与命名空间

1）模块： ES6 模块是标准，使用 import/export。

```
// math.tsexport function add(x: number, y: number): number { ... }// main.tsimport { add } from './math';
```

2）命名空间：旧的代码组织方式，使用 namespace 和 /// <reference>。在新代码中应优先使用模块。

21. 配置 tsconfig.json

TypeScript 项目的核心配置文件。

重要配置项：

1）compilerOptions： 编译选项。

① target： 编译目标（如 es5, es2015, esnext）。

② module： 模块系统（如 commonjs, es2015, esnext）。

③ strict： 是否开启所有严格类型检查（推荐开启）。

④ outDir： 输出目录。

⑤ rootDir： 源文件根目录。

2）include： 需要编译的文件。

3）exclude： 需要排除的文件。

第四部分：最佳实践与总结

22. 严格模式

1）在 tsconfig.json 中设置 "strict": true。

2）它包含了一系列严格的类型检查选项（如 noImplicitAny, strictNullChecks 等），能极大地提高代码质量。

23. 逐步迁移

1）对于现有 JavaScript 项目，可以将 .js 文件重命名为 .ts 文件，并逐步解决类型错误。

2）使用 // @ts-check 在 JavaScript 文件中开启初步的类型检查。