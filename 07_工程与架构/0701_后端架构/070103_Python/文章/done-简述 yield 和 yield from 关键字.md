> 已吸收至：[[07_工程与架构/0701_后端架构/070103_Python/070103_核心知识点/Python来源校准与降权准则|Python来源校准与降权准则]]、[[07_工程与架构/0701_后端架构/070103_Python/070103_知识地图|070103_Python知识地图]]

---
title: 简述 yield 和 yield from 关键字
author: Python技术迷
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg2NDY1NzMzNw==&mid=2247519945&idx=2&sn=a498888a15753010899bcaf1373e9573&chksm=cfa9dc24ebafdb7a0e31bebd8b84b8a3eb7dfc5e117d11ee3510d7e1904675565aacb9e06b94&mpshare=1&scene=24&srcid=1016E8K4Wuu0QEhQFysOQlYd&sharer_shareinfo=ea4b46002671f0e8a7f2e881884abda9&sharer_shareinfo_first=ea4b46002671f0e8a7f2e881884abda9#rd
---

在 Python 里，`yield` 和 `yield from` 这两个关键字经常会让初学者有点懵，其实理解了生成器的本质，它们用起来就特别自然。下面我就用比较口语化的方式，结合一些代码，来聊一聊这两个东西到底是干啥的。

### yield 的基本作用

`yield` 最大的特点是能让一个函数“暂停”。普通的函数执行完就结束了，但带有 `yield` 的函数会变成**生成器**，每次执行到 `yield` 的时候，就把一个值返回出去，同时把函数的执行状态保存住，下次再调用的时候会从上次停下来的地方继续往下跑。

举个例子：

```
def simple_generator():
    print("start")
    yield 1
    print("continue")
    yield 2
    print("end")

gen = simple_generator()

print(next(gen))  # 输出 start \n 1
print(next(gen))  # 输出 continue \n 2
print(next(gen))  # 抛 StopIteration
```

可以看到，每次调用 `next` 都会“推进”生成器到下一个 `yield`。这比普通函数返回一个列表要省内存，尤其是处理大数据的时候特别好用。

### yield from 的作用

如果说 `yield` 是手动一步步返回，那 `yield from` 就是“代理”，它会把生成器里面的 `yield` 全部自动转发出来。常见的场景是你有一个函数里需要从另一个生成器里拿值，这时候如果一个个写 `for` 去 `yield` 太麻烦了，直接用 `yield from` 就行。

先看没有 `yield from` 的写法：

```
def sub_generator():
    yield 1
    yield 2

def main_generator():
    for value in sub_generator():
        yield value
    yield 3

print(list(main_generator()))  # [1, 2, 3]
```

换成 `yield from`：

```
def main_generator():
    yield from sub_generator()
    yield 3

print(list(main_generator()))  # [1, 2, 3]
```

是不是一下子清爽很多？它还能把异常、返回值都帮你处理转发掉。

### 一个实际点的例子

比如你想做一个文件逐行读取的工具，如果直接 `readlines()`，大文件会直接爆内存，用 `yield` 就可以一行一行返回：

```
def read_file(path):
    with open(path, "r") as f:
        for line in f:
            yield line.strip()

for line in read_file("data.txt"):
    print(line)
```

如果你又想在读取的时候加个转换，比如把内容都转成大写，还可以用 `yield from` 来组合：

```
def upper_lines(path):
    yield from (line.upper() for line in read_file(path))

for line in upper_lines("data.txt"):
    print(line)
```

这样两个生成器就像乐高一样拼起来，逻辑还很清楚。

总结一下：`yield` 是把函数变成生成器，可以暂停和恢复；`yield from` 是简化代码、代理子生成器输出的利器。一个负责“产出”，一个负责“转发”。等你习惯之后，会发现这两个东西特别适合写那种“按需生产”的逻辑。

****-END****-****

我为大家打造了一份RPA教程，完全免费：songshuhezi.com/rpa.html

**🔥****虎哥私藏精品**🔥****

虎哥作为一名老码农，整理了全网最全**[《python高级架构师资料合集》](https://mp.weixin.qq.com/s?__biz=Mzg2MTYwOTM3MA==&mid=2247503198&idx=1&sn=84cfcacada10b11688da93acc9e17eeb&chksm=ce1601abf96188bd9be25e2fc412e5bc6af1230e4f33dcde0583e09ba80b3863e681e02c149d&scene=21&token=1985090817&lang=zh_CN#wechat_redirect)**，总量高达**650GB**，点击下方公众号回复关键字 python 全部**免费领**