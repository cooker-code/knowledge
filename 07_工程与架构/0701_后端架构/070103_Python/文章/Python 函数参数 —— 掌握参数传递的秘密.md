---
title: Python 函数参数 —— 掌握参数传递的秘密
author: PYant
date: 
url: https://mp.weixin.qq.com/s?__biz=MzYyNTM2MzkyNA==&mid=2247483998&idx=1&sn=053040c09342015f48d01773ed56da58&chksm=f1d8f06000106f477d94ea520b415f5c679f07bbb19519d997c6b7454aa36417be18c387aa83&mpshare=1&scene=24&srcid=1030jt4OZJKCIJB91ro1LpAa&sharer_shareinfo=d4d7f10dc59d1f45e57b4a2303d6059c&sharer_shareinfo_first=d4d7f10dc59d1f45e57b4a2303d6059c#rd
---

## 一、参数基础类型

### 1. 位置参数（Positional Arguments）

* **定义**：函数定义时按顺序声明的参数，调用时必须按相同顺序传递值
* **特点**：

+ 参数顺序决定传值逻辑
+ 调用时缺少参数会触发`TypeError`

* **注意**：多参数场景下需严格保持顺序，否则可能引发逻辑错误
* **样例**：

  ```
  def positional(name, age):  
      print(f'我的名字是：{name}，年龄：{age}')  
        
  positional('yant', 18)
  ```

### 2. 默认参数（Default Arguments）

* **定义**：在参数声明时赋予默认值，调用时可省略传参；如果调用时赋值，则覆盖默认值；
* **特性**：

+ 必须声明在位置参数之后
+ 默认值在函数定义时初始化，应避免使用可变对象（如列表）

* **样例**：

  ```
  def func(a=1, b=5):  
      return a + b  
    
  func()  # 输出 6  
  # 赋值调用  
  func(2) # 输出 7
  ```

### 3. 可变参数

#### 3.1 可变位置参数（`*args`）

* **功能**：接收任意数量位置参数，打包为元组
* **调用方式**：

```
 def sum_all(*nums):   
     return sum(nums)  
       
 sum_all(1,2,3)  # 输出6
```

* 可通过`list`解包列表/元组传递参数

#### 3.2 可变关键字参数（\*\*kwargs）

* **功能**：接收任意数量键值对，打包为字典
* **调用方式**：

```
def print_info(**data):  
    for k,v in data.items(): print(f"{k}:{v}")  
print_info(name="Alice", age=25)
```

* 可通过\*\*dict解包字典传递参数

---

## 二、高级参数组合

### 1. 命名关键字参数（Keyword-only Arguments）

* **定义**：通过`*`分隔符强制要求以键值对形式传参
* **语法**：

```
def register(name, *, email, phone):  
    pass  
# 必须指定参数名  
register("Bob", email="bob@test.com", phone=123456)
```

### 2. 参数组合顺序规则（非常重要）

1. 位置参数 → 默认参数 → 可变位置参数（\*args） → 命名关键字参数 → 可变关键字参数（\*\*kwargs）
2. 错误示例：def func(a=1, b)会导致语法错误（默认参数在位置参数前）

---

## 三、参数传递机制

### 1. 参数解包技巧

* **列表/元组解包**：使用`*`传递到`*args`

```
def sum_all(*args):  
    print(sum(args))  
  
nums = [2,3,4]  
sum_all(*nums)  # 等价于sum_all(2,3,4)
```

* **字典解包**：使用`**`传递到`**kwargs`

```
class People:  
    def __init__(self, name, age, **kwargs):  
        self.name = name  
        self.age = age  
        self.info = kwargs  
  
  
    def __str__(self):  
        return f'Name: {self.name}, Age: {self.age}, Other Message: {self.info}'  
  
  
  
data = {'city':'GuangZhou', 'job':'Engineer'}  
yant = People('Yant', 24, **data)  
print(yant) ## Name: Yant, Age: 24, Other Message: {'city': 'GuangZhou', 'job': 'Engineer'}
```

---

## 四、常见问题与陷阱

### 1. 默认参数陷阱

* **问题**：默认值在函数定义时创建，多次调用共享同一对象，这个问题需要留意，新手开发者出现的概率非常高

```
def append_to(element, target=[]):  
    target.append(element)  
    return target  
# 连续调用会在原列表累积元素，不会创建新的列表  
print(append_to(1))  # [1]  
print(append_to(2))  # [1, 2]
```

* **解决方案**：使用None占位并重新初始化

```
def append_to(element, target=None):  
    target = target or []  
    target.append(element)  
    return target
```

### 2. 参数顺序错误

* **典型错误**：`def func(a=1, b)`导致语法错误，这个错误pycharm会提示
* **修正**：必选参数需在默认参数前声明`def func(b, a=1)`

### 3. 命名冲突

* **现象**：`**kwargs`中的键名与函数参数名重复时引发逻辑错误
* **规避**：确保键名不与已有参数名冲突

```
def insert_dict(name, age, **kwargs):  
    person = {  
        'name': name,  
        'age': age,  
        **kwargs  
    }  
    return person  
  
result = insert_dict('Yant', 28, name='Yant', City='Guangzhou')  
print(result)
```

```
Traceback (most recent call last):  
  File "E:\PythonProject\pythonStudy\drfStudy\.debug\main.py", line 10, in <module>  
    result = insert_dict('Yant', 28, name='Yant', City='Guangzhou')  
TypeError: insert_dict() got multiple values for argument 'name'
```

---

## 五、最佳实践建议

1. **参数设计原则**：

* 优先使用位置参数保证基础功能
* 默认参数简化高频使用场景
* `*args/**kwargs`增强灵活性

2. **可读性优化**：

* 关键字参数明确传参意图（如`connect(timeout=10)`）
* 复杂参数组合添加类型注解

3. **调试技巧**：

* 使用`inspect` 模块分析参数签名
* 通过 `locals()` 打印函数内部参数状态

> 本文综合了Python参数系统的核心机制与实战经验，完整参数规范请参考Python官方标准库文档

---

✨**关注我**，获取更多Python学习资源、实战项目和行业动态！在公众号后台回复"python学习"，获取Python学习电子书籍！