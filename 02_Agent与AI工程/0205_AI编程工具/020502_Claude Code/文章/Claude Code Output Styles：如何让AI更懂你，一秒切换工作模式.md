---
title: Claude Code Output Styles：如何让AI更懂你，一秒切换工作模式
author: 佛喜酱的AI实践
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg3NzQ4MDY1MA==&mid=2247484822&idx=1&sn=4eca83a57081587d5266b68912865bee&chksm=ce203effdd521c2938af52547bc471ff689eb5cefb35749fb9737ed42233df62e7f545527e6a&mpshare=1&scene=24&srcid=1206r2ffdAnAPIDdUbUf9ZcA&sharer_shareinfo=cdbd4521f609270f71bf469b3602095f&sharer_shareinfo_first=cdbd4521f609270f71bf469b3602095f#rd
---

## Why：你的AI真的懂你吗？

每天都在跟AI打交道，但总觉得少了点什么？

问技术问题，它给了一大堆理论。想要快速答案，它却长篇大论。想学习新知识，它又说得太过简单。

问题不在AI不够智能，而在于AI不懂切换工作模式。

你需要在不同场景下用不同的方式与AI协作：

* **开发时**

  ：需要高效解决问题
* **学习时**

  ：需要深入理解原理
* **练习时**

  ：需要互动指导

但AI始终用同一种模式回应，就像一个只会用同一种语气说话的人，让人感觉很死板。

其实问题本质就是：AI不仅需要懂你，更需要学会**一秒切换工作模式**。

## What：一秒切换工作模式的超能力

简单来说，Output Styles 就是让 Claude Code 能够在一秒内切换工作模式，用最适合当前场景的方式与你协作。

想象一下：你跟老板汇报工作、跟朋友聊天、跟客户谈生意，说话方式完全不一样，对吧？

* **跟老板汇报**

  ：简洁明了，重点突出，用数据说话
* **跟朋友聊天**

  ：轻松随意，可以开开玩笑，分享细节
* **跟客户谈生意**

  ：专业严谨，既要有说服力又要体现价值

同样的道理，不同场景下，你需要AI用不同的方式"懂你"：

* **开发时**

  ：需要快速解决问题的同事
* **学习时**

  ：需要耐心讲解的老师
* **练习时**

  ：需要带你自己动手的教练

Claude Code 的 Output Styles 就是实现这种**一秒切换**的魔法，只需要一个简单命令，AI就能立刻切换到对应的工作模式：

```
/output-style [mode]  # 一秒切换，即时生效
```

它有三种内置模式：

**Default模式（默认模式）**：懂你需要**效率**。就像经验丰富的同事，直接给答案，不啰嗦。

**Explanatory模式（解释模式）**：懂你需要**理解**。就像耐心的老师，不仅给答案，还解释为什么，让你真正学会。

**Learning模式（学习模式）**：懂你需要**成长**。就像专业的教练，既教你知识，又给你练习机会，让你举一反三。

最厉害的是，这些切换都是**一秒完成**的：

```
/output-style default      # 立即切换到高效模式  
/output-style explanatory  # 立即切换到教学模式  
/output-style learning     # 立即切换到教练模式
```

说白了，这就是让AI从"死板助手"变成"贴心伙伴"的关键——**不仅懂你，还能一秒切换工作模式**。

### 超越内置：让AI更懂你的独特需求

内置的三种模式已经很懂你，但Claude Code还有更厉害的功能：**让AI完全理解你**。

这就像教AI学习你的工作习惯、专业领域和个人偏好，让它真正成为你的专属助理。

**如何让AI更懂你？**

简单命令就能开始定制：

```
/output-style:new I want an output style that...
```

**实际例子**：

* 想让AI懂你的写作风格？`/output-style:new I want an output style that helps me write better blog posts`
* 需要AI懂你的数据分析流程？`/output-style:new I want an output style that focuses on data analysis and visualization`
* 要AI懂你的项目管理方法？`/output-style:new I want an output style that acts as a project management expert`

**定制化带来的好处**：

* **深度理解**

  ：AI懂你的专业领域和工作流程
* **个性适配**

  ：按照你的习惯和偏好进行协作
* **效率提升**

  ：AI提前知道你的需求，减少沟通成本

这样，AI就不再是通用工具，而是真正懂你的工作伙伴。

### 功能对比：Output Styles vs Subagents vs Slash Commands

简单说：Output Styles让AI更懂你，Subagents让专业AI做事，Slash Commands是快捷操作。

## How：同一请求的不同体验

### Go Hello World 实战演练：一秒切换的威力

现在用同一个请求"请帮我实现一个简单的hello world golang入门代码"来测试三种模式的一秒切换效果

### 1️⃣ Default模式：一秒切换到高效同事模式

**目标**：体验AI一秒切换到标准专业回答

我只需要输入：

```
/output-style default
```

AI**一秒内**就从默认模式切换到高效工作模式，然后询问：

```
请帮我实现一个简单的hello world golang入门代码
```

Claude Code直接给代码：

```
package main  
  
import "fmt"  
  
// main 程序的入口点  
func main() {  
    fmt.Println("Hello, World!")  
}
```

**我的感受**：很直接，给代码就完事了。就像问同事怎么写，他直接发你代码片段，大家都挺忙的，简单高效。

### 2️⃣ Explanatory模式：一秒切换到耐心老师模式

**目标**：看看AI一秒切换后如何详细讲解同一个请求

现在我用一个命令**一秒切换**到教学模式：

```
/output-style explanatory
```

AI瞬间就变成了耐心的老师，用同样的请求：

```
请帮我实现一个简单的hello world golang入门代码
```

这次Claude Code不仅给代码，还详细解释：

```
package main  
  
import (  
    "fmt"  
)  
  
// main 函数是 Go 程序的入口点  
func main() {  
    fmt.Println("Hello, World!")  
  
    // 添加一些额外的信息来展示 Go 的基本特性  
    name := "Go 初学者"  
    age := 25  
  
    // 使用 fmt.Printf 进行格式化输出  
    fmt.Printf("欢迎 %s，年龄 %d，来到 Go 的世界！\n", name, age)  
  
    // 演示基本的数学运算  
    result := add(5, 3)  
    fmt.Printf("5 + 3 = %d\n", result)  
  
    // 演示循环  
    fmt.Println("倒计时开始:")  
    for i := 5; i >= 1; i-- {  
        fmt.Printf("%d...\n", i)  
    }  
    fmt.Println("开始学习 Go 吧！")  
}  
  
// add 函数演示基本的函数定义  
func add(a, b int)int {  
    return a + b  
}%
```

**我的感受**：哇塞，Explanatory模式太贴心了！不仅给了基础的Hello World，还扩展了很多Go语言的核心概念：变量、函数、循环、格式化输出。这样一口气就能学到好多基础知识，比单独看Hello World有用多了。

### 3️⃣ Learning模式：一秒切换到专业教练模式

**目标**：体验AI一秒切换后如何像教练一样带读者一步步学习

最后，我再用一个命令**一秒切换**到教练模式：

```
/output-style learning
```

AI瞬间就变成了互动教练，用同样的请求：

```
请帮我实现一个简单的hello world golang入门代码
```

```
package main  
  
import"fmt"  
  
func main() {  
    // TODO(human): 在这里添加更多有趣的输出或互动功能  
    fmt.Println("Hello, World!")  
    fmt.Println("欢迎来到 Go 语言的世界!")  
  
    // 展示一些基本的 Go 语言特性  
    name := "Go 初学者"  
    fmt.Printf("你好, %s!\n", name)  
  
    // 简单的数学运算  
    result := 1 + 1  
    fmt.Printf("1 + 1 = %d\n", result)  
}%
```

然后AI变成了任务分配教练：

> 我已经创建了一个基本的 Hello World Go 程序，它展示了 Go 语言的核心结构：包声明、导入语句、main 函数、变量声明和基本的字符串格式化。程序运行正常，输出了一些欢迎信息和简单的数学计算。
>
> **Your Task**: 在 main.go 文件中，扩展 TODO(human) 部分，添加更多有趣的互动功能。你可以添加一个简单的用户交互功能，比如询问用户的姓名并个性化响应，或者添加更多的数学运算示例。

**我的感受**：这种Learning模式真的很特别！它不是一步步教我，而是：

1. 先给一个完整的、可以运行的程序
2. 留下明确的 `TODO(human)` 标记让我知道哪里需要动手
3. 给我具体的任务目标，让我有方向地去改进

说实话，这种方式更像真实的编程学习：

* 你先得到一个能工作的基础代码
* 然后有明确的任务要去完善
* 不会一下子给你太多压力，循序渐进

而且 `TODO(human)` 这个标记很聪明，让我一眼就知道这是"留给人类的作业"！

### 效果对比总结

三种模式响应同一个请求"请帮我实现hello world代码"，结果完全不同：

* **Default模式**

  ：10秒得到代码，适合有基础的开发者快速查代码
* **Explanatory模式**

  ：2分钟学会Go基础，适合想理解语言特性的学习者
* **Learning模式**

  ：10分钟互动学习，适合零基础入门者系统掌握

你说，AI是不是真的越来越懂你了，而且还能**一秒切换**？

* 开发时需要快速解决问题？`/output-style default` 一秒切换到高效模式
* 学习新技术需要深入理解？`/output-style explanatory` 一秒切换到教学模式
* 入门新领域需要循序渐进？`/output-style learning` 一秒切换到教练模式

真的，这种AI也太智能了！不仅懂你的需求，还能**一秒切换**到最合适的工作模式。

## Scenarios：应用场景

**新手学习阶段**：刚开始学新技术？`/output-style learning` 一秒切换到教练模式，AI懂你需要循序渐进的指导。

**日常开发工作**：需要快速解决问题？`/output-style default` 一秒切换到高效模式，AI懂你的时间宝贵，需要高效解决方案。

**深度技术研究**：想深入了解技术原理？`/output-style explanatory` 一秒切换到教学模式，AI懂你的求知欲，会把来龙去脉讲清楚。

**团队协作场景**：

* 给同事看代码：Default模式（懂你们需要效率）
* 带新人入门：Learning模式（懂新人需要耐心指导）
* 写技术文档：Explanatory模式（懂读者需要详细解释）

**专业领域定制场景**： 当内置模式不够懂你时，就可以用自定义模式让AI更懂你的专业需求

**个人使用建议**：建议先全部体验一遍，找到最懂你使用习惯的模式。当有特定专业需求时，就创建自定义模式，让AI真正懂你。

## Summary：

通过学习 Output Styles，你将掌握让AI真正懂你并能**一秒切换**的能力。这不仅是一个功能的学习，更是AI协作思维的升级。文章通过实战演示，让你体验从AI死板单一到AI灵活切换的完整转变过程，实现AI使用效率的飞跃提升。

最重要的是，你会发现：原来AI不仅可以成为真正懂你的工作伙伴，还能**一秒切换**到最合适的工作模式！

现在你已经掌握了AI协作的终极技能：**让AI更懂你，一秒切换工作模式**。

希望本文对你有所帮助，想了解更多AI实践，关注我的同名公众号:)，定期分享AI实战，一起探索AI的无限可能！

---

## 参考资源

* **官方文档**

  ：Claude Code Output Styles - 完整的功能介绍