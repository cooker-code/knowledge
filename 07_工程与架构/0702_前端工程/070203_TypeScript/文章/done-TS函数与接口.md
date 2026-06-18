> 已吸收至：[[07_工程与架构/0702_前端工程/070203_TypeScript/070203_核心知识点/TypeScript类型边界准则|TypeScript类型边界准则]]、[[07_工程与架构/0702_前端工程/070203_TypeScript/070203_知识地图|070203_TypeScript知识地图]]

---
title: TS函数与接口
author: 亚奇智汇
date:
url: https://mp.weixin.qq.com/s?__biz=MzU3MDYxNzI0NA==&mid=2247484358&idx=1&sn=5604115758ec5c12bfce6aef16034eaa&chksm=fd1a5e4b6dd0860290ee26baf8eccedad0be96a58b5b2f554ce65bd66ba15be074159e3ddf5f&mpshare=1&scene=24&srcid=1205MNRIQMQ0YmWKUpPWnfgK&sharer_shareinfo=66ffb625ee66ce03198c3e1c81dbd718&sharer_shareinfo_first=66ffb625ee66ce03198c3e1c81dbd718#rd
---

**TS函数与接口**

# TypeScript 函数与类教程

## 三、函数

### 1. 函数声明

```
function add(x: number, y: number): number {     return x + y; }// 如果传入了错误的类型，比如字符串，TypeScript 编译器就会报错：add("Hello", 1); // ❌ 错误：不能将类型"string"分配给类型"number"
```

### 2. 函数表达式

```
let myAdd: (x: number, y: number) => number = function(x: number, y: number): number { return x + y; };
```

### 3. 可选参数和默认参数

```
function buildName(firstName: string, lastName?: string) {   if (lastName) {      return firstName + " " + lastName;    }   else {      return firstName;    }  }  function buildNameWithDefault(firstName: string, lastName = "Smith") {      return firstName + " " + lastName;  }
```

### 4. 剩余参数（Rest Parameters）

```
function buildName(firstName: string, ...restOfName: string[]) { return firstName + " " + restOfName.join(" "); }
```

### 5. this 和箭头函数

```
let deck = {    suits: ["hearts", "spades", "clubs", "diamonds"],    createCardPicker: function() {       return () => {         let pickedCard = Math.floor(Math.random() * 52);         return { suit: this.suits[0], card: pickedCard };       };    }  };
```

### 6. 函数重载（Function Overloads）

```
functionreverse(x: number): number; functionreverse(x: string): string; functionreverse(x: number | string): number | string {   if (typeof x === 'number') {     returnNumber(x.toString().split('').reverse().join(''));   } else if (typeof x === 'string') {     return x.split('').reverse().join('');   } }
```

## 四、接口和类

### 1. 接口（Interfaces）

① 接口描述对象

```
interface LabelledValue { label: string; } function printLabel(labelledObj: LabelledValue) {    console.log(labelledObj.label); }
```

② 接口描述函数类型

```
interface SearchFunc {    (source: string, subString: string): boolean; }
```

③ 接口描述数组和索引类型

```
interface StringArray { [index: number]: string; } interface Dictionary { [index: string]: string; }
```

### 2. 类（Classes）

```
class Greeter {   greeting: string;  constructor(message: string) {   this.greeting = message;   }   greet() {     return"Hello, " + this.greeting;   }}
```

### 3. 类的继承

```
class Animal {   name: string;   constructor(theName: string) { this.name = theName; }   move(distanceInMeters: number = 0) {     console.log(`${this.name} moved ${distanceInMeters}m.`);   } } class DogextendsAnimal {    constructor(name: string) { super(name); }    bark() { console.log('Woof!'); } }
```

### 4. 抽象类（Abstract Classes）

```
abstract class Animal{    abstract makeSound(): void;    move(): void { console.log('roaming the earth...'); } } class DogextendsAnimal{    makeSound() { console.log('Woof!'); } }
```

### 5. 方法调用实例

```
const myDog = newDog('Tom'); myDog.makeSound(); // Woof! myDog.move(); // roaming...
```

## 总结

| 模块 | 核心内容 |
| --- | --- |
| 三、函数 | 函数声明、函数表达式、可选/默认参数、剩余参数、this与箭头函数、函数重载 |
| 四、接口和类 | 接口（对象、函数、数组）、类（定义、构造器、继承）、抽象类、方法调用实例 |