---
title: Pydantic field_validator助你更优雅处理数据验证
author: PYant
date: 
url: https://mp.weixin.qq.com/s?__biz=MzYyNTM2MzkyNA==&mid=2247484779&idx=1&sn=953462a37e0666b56d6519b346de6f49&chksm=f1f22adb19a49844adb04a8a5fe1cc86af30da501df82724bb22a1c617713ead899e1139b1e2&mpshare=1&scene=24&srcid=1118wEkWb9BRQsek82LtxHjQ&sharer_shareinfo=34cc0995f1e93c97b34656ad04b7d8a3&sharer_shareinfo_first=34cc0995f1e93c97b34656ad04b7d8a3#rd
---

书接上回，上文讲了Pydantic的Field字段使用。既然已经掌握了字段的定义，也能在字段定义的时候进行字段验证。此时，你会不会有疑问如果定义字段的时候有的业务验证比较复杂怎么办?这不`field_validator`就诞生了嘛！

> 注意注意Pydantic V1版本和 V2版本，版本差异较大，以下内容仅支持V2版本

## 简单说一下Pydantic的验证器

在`Pydantic`中有三类验证器，分别对应不同级别的验证场景  
**`@field_validator`** ：字段级别验证  
**`@model_validator`** ：模型级别验证  
**`@computed_field`** ： 计算字段，当访问字段时触发  
`field_validator`、`model_validator`和 `computed_field` 的触发时机不一样，需要在定义时配置触发时机。本篇先讲`field_validator`,剩下的下回分解！

## @field\_validator

认识一个函数的功能，先看他能干什么（上面说了）、传什么参数、返回值是啥？

```
1

2

3

4

5

6

7

8

  

def field_validator(  
    field: str, # 验证的字段名称，可以是多个  
    /,  
    *fields: str,  
    mode: FieldValidatorModes = 'after',  # 验证触发模式  
    check_fields: bool | None = None, # 严格检查字段存在性  
    json_schema_input_type: Any = PydanticUndefined,  
) -> Callable[[Any], Any]
```

### 使用案例

这个固定语法

```
1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

  

from pydantic import BaseModel, field_validator  
from typing import Optional  
   
class User(BaseModel):  
    name: str  
    age: int  
    email: Optional[str] = None  
      
    @field_validator('name')  
    def validate_name(cls, v: str) -> str:  
        """验证用户名"""  
        if len(v.strip()) < 2:  
            raise ValueError('姓名长度必须至少2个字符')  
        return v.title()  # 自动将姓名首字母大写  
   
# 测试验证器  
try:  
    invalid_user = User(name="j", age=18)  # 会触发验证错误  
except ValidationError as e:  
    print(f"验证错误: {e}")
```

### 注意注意

* • 必须是类方法或者是静态方法。如果是类方法，可以不用`@classmethod`装饰，但是第一个参数是`cls`,如果使用`@classmethod`装饰，`@classmethod`必需在`field_validator`上方
* • 第一个参数是类本身
* • 第二个参数是要验证的字段值
* • 必须返回处理后的值或抛出验证错误  
  违反上面的规则会有什么问题？不要问我是怎么知道的

## 现在讲解`mode`参数

想要使用一个函数，必定需要知道什么时候调用，现在来解决这个问题

### `after` 模式（默认）

在类型转换后执行验证：

```
1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

  

from pydantic import BaseModel, field_validator  
   
class Product(BaseModel):  
    price: float  
    quantity: int  
      
    @field_validator('price', mode='after')  
    @classmethod  
    def validate_price(cls, v: float) -> float:  
        """验证价格"""  
        if v <= 0:  
            raise ValueError('价格必须大于0')  
        return round(v, 2)  # 保留两位小数  
   
# 测试  
product = Product(price=19.999, quantity=10)  
print(f"价格: {product.price}")  # 输出: 价格: 20.0
```

### `before` 模式

在类型转换前执行验证：

```
1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

  

from pydantic import BaseModel, field_validator  
   
class Configuration(BaseModel):  
    percentage: float  
      
    @field_validator('percentage', mode='before')  
    @classmethod  
    def validate_percentage_format(cls, v) -> float:  
        """验证百分比格式"""  
        if isinstance(v, str) and v.endswith('%'):  
            # 如果是字符串且以%结尾，转换为小数  
            return float(v.rstrip('%')) / 100  
        return v  
      
   
# 测试  
config2 = Configuration(percentage="50%")  # 自动转换  
print(f"配置2: {config2.percentage}")  # 输出: 0.5
```

## 现在看一下案例

我感觉可以直接上难度，感觉感觉下面的应用场景

### 验证多个字段

```
1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

  

from datetime import date  
from pydantic import BaseModel, field_validator, ValidationError  
   
class Event(BaseModel):  
    name: str  
    start_date: date  
    end_date: date  
   
    @field_validator('start_date', 'end_date')  
    @classmethod  
    def validate_future_events(cls, v) -> any:  
        if isinstance(v, date):  
            return ValueError(f'日期必需是 date 对象，实际{type(v)}')  
        return v  
   
   
# 测试  
try:  
    event = Event(  
        name="重要会议",  
        start_date=date(2024, 1, 15),    
        end_date='2024/01/15'  # 字符串验证不通过  
    )  
except ValidationError as e:  
    print(f"事件创建错误: {e}")
```

### 条件验证

```
1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

26

27

28

29

  

from pydantic import BaseModel, field_validator  
from typing import Optional  
   
class Subscription(BaseModel):  
    plan_type: str  # 'basic', 'premium'  
    card_number: Optional[str] = None  
      
    @field_validator('card_number')  
    @classmethod  
    def validate_payment_info(cls, v: Optional[str], info) -> Optional[str]:  
        """根据订阅计划验证支付信息"""  
        plan_type = info.data.get('plan_type')  
          
        if plan_type == 'premium' and not v:  
            raise ValueError('高级订阅需要提供支付卡号')  
          
        if v and len(v.replace(' ', '')) != 16:  # 移除空格后检查长度  
            raise ValueError('卡号必须是16位数字')  
          
        return v  
   
# 测试  
basic_sub = Subscription(plan_type='basic')  # 正常  
premium_sub = Subscription(plan_type='premium', card_number='1234 5678 9012 3456')  # 正常  
   
try:  
    invalid_premium = Subscription(plan_type='premium')  # 会报错  
except ValidationError as e:  
    print(f"订阅错误: {e}")
```

### 复杂数据验证

```
1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

26

27

28

29

30

31

32

33

34

35

36

37

38

39

40

41

42

43

44

45

46

47

48

49

50

51

52

53

  

from pydantic import BaseModel, field_validator  
from typing import List, Dict  
   
class Survey(BaseModel):  
    questions: List[str]  
    responses: Dict[str, str]  
      
    @field_validator('questions')  
    @classmethod  
    def validate_questions(cls, v: List[str]) -> List[str]:  
        """验证问题列表"""  
        if len(v) < 1:  
            raise ValueError('至少需要一个问题')  
          
        if len(v) > 10:  
            raise ValueError('问题数量不能超过10个')  
          
        # 验证每个问题  
        for i, question in enumerate(v):  
            if len(question.strip()) < 5:  
                raise ValueError(f'问题{i+1}至少需要5个字符')  
          
        return v  
      
    @field_validator('responses')  
    @classmethod  
    def validate_responses(cls, v: Dict[str, str], info) -> Dict[str, str]:  
        """验证回答"""  
        questions = info.data.get('questions', [])  
          
        # 检查是否回答了所有问题  
        for question in questions:  
            if question not in v:  
                raise ValueError(f'问题"{question}"未得到回答')  
          
        # 验证回答内容  
        for question, answer in v.items():  
            if question not in questions:  
                raise ValueError(f'未知问题: {question}')  
              
            if not answer.strip():  
                raise ValueError(f'问题"{question}"的回答不能为空')  
          
        return v  
   
# 测试  
survey = Survey(  
    questions=["你喜欢编程吗？", "你使用什么语言？"],  
    responses={  
        "你喜欢编程吗？": "是的",  
        "你使用什么语言？": "Python"  
    }  
)
```

## 总结一下

Pydantic 的 `field_validator`提供了强大而灵活的字段验证功能：

1. 1. **简单易用**：通过装饰器轻松添加验证逻辑
2. 2. **模式灵活**：支持前后验证模式
3. 3. **功能强大**：支持多字段验证、条件验证等复杂场景
4. 4. **错误处理**：提供清晰的验证错误信息

通过合理使用 `field_validator`，你可以构建出既安全又易维护的数据模型，确保应用程序数据的完整性和一致性。

希望本教程能帮助你更好地理解和使用 Pydantic 的字段验证功能！

 

**阅读推荐**：

[数据验证神器Pydantic：让数据清洗变得简单高效](https://mp.weixin.qq.com/s?__biz=MzYyNTM2MzkyNA==&mid=2247484771&idx=1&sn=f26e889964be6f4e3e57a640d34683a8&scene=21#wechat_redirect)

[Pydantic Field字段：让你的数据验证强大10倍](https://mp.weixin.qq.com/s?__biz=MzYyNTM2MzkyNA==&mid=2247484775&idx=1&sn=1016522ef35308ef787b88448a9c3e56&scene=21#wechat_redirect)

 

---

✨**关注我**，获取更多Python学习资源、实战项目和行业动态！在公众号后台回复"python学习"，获取Python学习电子书籍！