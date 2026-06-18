> 已吸收至：[[07_工程与架构/0701_后端架构/070103_Python/070103_核心知识点/Python来源校准与降权准则|Python来源校准与降权准则]]、[[07_工程与架构/0701_后端架构/070103_Python/070103_知识地图|070103_Python知识地图]]

---
title: 掌握函数，你的 Python 水平将直接跨一个台阶
author: 虎妞编程乐园
date:
url: https://mp.weixin.qq.com/s?__biz=Mzk4ODM1NjczMQ==&mid=2247485156&idx=1&sn=a2a70fc20b7195236b7e46808f7d005a&chksm=c4e0818be158d1aebb07d8b869dba4d663d60bfb17785ba09c9d25625823e21fbe27d76697fd&mpshare=1&scene=24&srcid=1012gzPdujnulKZHaRTaH9Al&sharer_shareinfo=2492bb0e9ee2f4a61931e6f289a33dd9&sharer_shareinfo_first=2492bb0e9ee2f4a61931e6f289a33dd9#rd
---

在学 Python 的过程中，你会发现一个有趣的现象：不管是写小脚本，还是做大项目，**函数**几乎无处不在。 如果说变量是程序的“砖瓦”，那么函数就是“积木模块”。学会函数，就像学会了把零散的砖瓦搭建成房子。

很多新手学到函数时，脑袋里可能会冒出这样的疑问：

* 函数到底是干啥的？
* 我写 print() 打印也能跑，为什么要费劲定义函数？
* 函数里面那些参数、返回值、作用域看着有点绕，到底该怎么理解？

别急，这篇文章会带你**从基础到进阶**，一口气把 Python 函数的精髓讲清楚。文章比较长，但读完你会对函数有个全面认知，足够应付大多数开发场景。

---

## 1. 函数是啥？

一句话解释：**函数就是帮你把代码“打包”起来，想用的时候直接拿出来调用。**

想象一下：如果你每天都要写一百遍 `print("hello world")`，是不是很麻烦？这时候你就可以定义一个函数，把“打印 hello world”的逻辑放进去，后面只要写函数名，代码就能跑，既省事又干净。

```
def say_hello():
    """打印问候语"""
    print('hello world')

# 调用函数
say_hello()
say_hello()
```

输出：

```
hello world
hello world
```

这就是函数最朴素的意义：**把重复的逻辑抽出来，提高复用性。**

---

## 2. 参数的魔力：让函数更灵活

光打印 hello world 显然没啥意思。函数真正的威力在于它可以接收“参数”，让逻辑变得灵活。

```
def greet(name):
    """向指定的人问好"""
    print(f'Hello, {name}!')

greet('Alice')
greet('Bob')
```

输出：

```
Hello, Alice!
Hello, Bob!
```

这就像你点外卖时给备注：

* 说“加辣”就是一个参数；
* 说“不要葱”也是一个参数。

函数的参数，正是让逻辑根据输入而变化的关键。

---

## 3. 深入理解函数的“参数系统”

Python 的函数参数有点小门道，新手经常被坑。咱们一条一条拆开讲：

### 3.1 位置参数

按照顺序把值传给函数，叫位置参数。

```
def describe_pet(animal_type, pet_name):
    print(f"I have a {animal_type}.")
    print(f"My {animal_type}'s name is {pet_name}.")

describe_pet('hamster', 'Harry')
```

输出：

```
I have a hamster.
My hamster's name is Harry.
```

---

### 3.2 关键字参数

懒得记顺序？直接写上参数名就行：

```
describe_pet(pet_name='Harry', animal_type='hamster')
```

效果完全一样。

---

### 3.3 默认参数

有些参数常用值差不多，干脆给它一个默认值。

```
def describe_pet(pet_name, animal_type='dog'):
    print(f"I have a {animal_type}.")
    print(f"My {animal_type}'s name is {pet_name}.")

describe_pet('Willie')  # 不写 animal_type，就默认是 dog
```

结果：

```
I have a dog.
My dog's name is Willie.
```

就像点奶茶默认是“半糖”，除非你主动说“我要全糖”。

---

## 4. 返回值：函数的“输出口”

如果函数只能打印结果，那未免太局限了。更常见的需求是：**函数做了计算，把结果返回出来，供后续代码使用。**

### 4.1 返回单个值

```
def add_numbers(a, b):
    return a + b

result = add_numbers(3, 5)
print(result)  # 8
```

---

### 4.2 返回多个值

Python 里返回多个值很常见，其实是打包成了一个元组。

```
def get_name():
    first = 'John'
    last = 'Doe'
    return first, last

first_name, last_name = get_name()
print(first_name, last_name)
```

结果：

```
John Doe
```

---

## 5. 高级玩法：\*args 和 \*\*kwargs

有时候你不知道要传几个参数，这时候就用可变参数。

### 5.1 `*args` —— 不定长位置参数

```
def make_pizza(*toppings):
    print("Making a pizza with the following toppings:")
    for topping in toppings:
        print(f"- {topping}")

make_pizza('pepperoni')
make_pizza('mushrooms', 'green peppers', 'extra cheese')
```

---

### 5.2 `**kwargs` —— 不定长关键字参数

```
def build_profile(first, last, **user_info):
    user_info['first_name'] = first
    user_info['last_name'] = last
    return user_info

user_profile = build_profile('albert', 'einstein',
                            location='princeton',
                            field='physics')
print(user_profile)
```

输出：

```
{'location': 'princeton', 'field': 'physics', 'first_name': 'albert', 'last_name': 'einstein'}
```

这就像填简历：前两项是必须的，其它信息随便加。

---

## 6. 函数也能“像变量一样玩”

函数在 Python 里是一等公民，能干的事可多了。

### 6.1 赋值给变量

```
def shout(text):
    return text.upper()

yell = shout
print(yell('hello'))  # HELLO
```

---

### 6.2 当作参数传递

```
def greet(func):
    greeting = func('Hi, I am created by a function')
    print(greeting)

greet(shout)
```

这就是函数式编程的雏形。

---

## 7. Lambda 函数：匿名的小工具

有时候函数太短，不想大费周章写 `def`，就用 `lambda`。

```
square = lambda x: x ** 2
print(square(5))  # 25

numbers = [1, 2, 3, 4]
squared_numbers = list(map(lambda x: x**2, numbers))
print(squared_numbers)  # [1, 4, 9, 16]
```

---

## 8. 作用域：变量能活多久？

### 8.1 局部 vs 全局

```
x = 10

def my_func():
    x = 20
    print("局部x:", x)

my_func()
print("全局x:", x)
```

结果：

```
局部x: 20
全局x: 10
```

---

### 8.2 修改全局变量

```
x = 10

def modify_global():
    global x
    x = 20

modify_global()
print(x)  # 20
```

---

## 9. 装饰器：优雅的“外挂”

装饰器本质上就是函数套函数，可以在不修改原函数的前提下，给它加点“前戏”和“后戏”。

```
def my_decorator(func):
    def wrapper():
        print("装饰器: 在函数调用前执行")
        func()
        print("装饰器: 在函数调用后执行")
    return wrapper

@my_decorator
def say_hello():
    print("Hello!")

say_hello()
```

---

## 10. 实战案例

### 10.1 小型计算器

```
def calculator(operation, a, b):
    if operation == 'add':
        return a + b
    elif operation == 'subtract':
        return a - b
    elif operation == 'multiply':
        return a * b
    elif operation == 'divide':
        if b == 0:
            return "Error: Division by zero"
        return a / b
    else:
        return "Error: Invalid operation"

print(calculator('add', 5, 3))       # 8
print(calculator('divide', 10, 2))   # 5.0
print(calculator('power', 2, 3))     # Error: Invalid operation
```

---

### 10.2 用户验证

```
def validate_user(username, password):
    if len(username) < 4:
        return False, "用户名至少需要4个字符"
    if len(password) < 8:
        return False, "密码至少需要8个字符"
    if not any(c.isdigit() for c in password):
        return False, "密码必须包含数字"
    return True, "验证成功"

result, message = validate_user("admin", "password123")
print(result, message)
```

---

## 结语

函数在 Python 里的地位，就像发动机之于汽车。**没有函数的代码，迟早会乱成一团；善用函数的代码，却能清晰、可维护、还容易扩展。**

从最简单的封装，到高阶的装饰器、lambda，函数不仅仅是语法工具，更是一种思维方式。掌握函数，你才算真正跨入了 Python 的大门。