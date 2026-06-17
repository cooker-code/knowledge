---
title: Prompt Engineering (提示词)进阶必会：从“瞎聊”到“精准控制”
author: 前端AI行走
date: 
url: https://mp.weixin.qq.com/s?__biz=MzU5MjYwMDgzNQ==&mid=2247487376&idx=1&sn=dca5193c8fdeca567dfbfbfa2d63838a&chksm=ff8d1c7fd10f9b453012e2018b8bfc53c1da54d1499115c6a90426063988678f74b5152bfce0&mpshare=1&scene=24&srcid=1206Z19nj3Vu027rca82FX1h&sharer_shareinfo=4c69fea0f6761f0f9bccfc6f5964d3d1&sharer_shareinfo_first=4c69fea0f6761f0f9bccfc6f5964d3d1#rd
---

前言：

在AI编程的过程中，在实际真实项目开发过程中，在使用AI工具的时候，你是否觉得AI时而聪明绝顶，时而智商掉线？或者完全变成了傻逼了？

区别往往在于你使用的“手段”与“招式”，学会如何与它（LLM大语言模型）“对话”。

本文将深入浅出地讲解 Prompt Engineering (提示词) 的三大核心心法：**Zero-shot（零样本提示）**、**Few-shot**（少样本提示） 和 **Chain of Thought (CoT) 思维链**，并手把手教你在 DeepSeek 大模型中，尝试写结构化的 Prompt，生成完美的结构化 JSON 数据。

一、Zero-shot (零样本提示) —— “直接硬来”

**基本概念：不给 AI 任何示例，直接下达指令。这是我们最常用的方式。**

### 前端类比：

### 

**就像调用一个函数，只传参数，不给回调或配置。**

```
// Zero-shotgotoSubmit("Submit"); 
```

### 场景与局限：

### 

* **适用：通用的一些常识性问题（比如“解释什么是 Vue框架”）**
* **缺点：对于复杂任务，AI 容易“自由发挥”，导致输出格式不可控。**

### 

### 案例类比：

### 

> Prompt: 写一个 React 按钮组件。
>
> AI 回答: 可能会给你 Class Component，也可能是 Functional Component，样式可能是内联的，也可能是 CSS Modules，也有可能是根据项目引用的常用按钮组件来，完全看它心情。且不可控制的。

### 

## 二、Few-shot (少样本提示) —— “照猫画虎”

****基本概念：在Prompt指令中，根据你的要求，你自己提供一个或多个“输入XXX-输出XXX”的示例（Example），让AI模仿你的逻辑和格式，让它明白你要的结果是什么怎样要的？。****

**前端类比：**

就像写单元测试或 Storybook 文档说明一样，你告诉函数：“当输入 A 时，你应该输出 B”。

```
// Few-shotexpect(transform('red')).toBe('#ff0000');expect(transform('blue')).toBe('#0000ff');// Now transform 'green'
```

**实际上，大模型给出的结果，都是符合预期且有效的。**

****为什么有效：****

**大模型本质是“文本补全”。当你给了示例，它会捕捉示例中的**模式 (Pattern)**（如 JSON 结构、代码风格、语气），其实范例也是一种很强的提示词，结合上下文来，再统一将其应用到新的任务中。这样给出的结论就更加符合要求，且精准。**

**案例类比：生成定制的CSS类名**

> Prompt: 根据描述生成 Tailwind 类名。 示例 1: 输入: 红色背景，圆角，内边距 4 输出: bg-red-500 rounded p-4
>
> 示例2:输入: 大号字体，加粗，蓝色文字 输出: text-lg font-bold text-blue-500
>
> 任务: 输入: 绝对定位，居中，半透明黑色背景，输出满足上述示例的样式 

### 

## 三、Chain of Thought (CoT, 思维链) —— “让子弹飞一会儿”

******基本概念：要求AI在给出最终答案之前，先展示它的推理过程。通常通过加上一句“让我们一步步思考来”或者“输出你的思考步骤来”，这样会触发并强制模型输出推理思考过程。**

****前端类比：**

就像在复杂的执行函数中写注释，或者在出错了之后，调试代码的过程中通过 Debug 来打印中间变量。**

```
// CoTfunction controlNextCalc(data) {  //  1: 过滤数据...  //  2: 重新组装...  //  3: 返回新数据...  return result;}
```

******为什么有效：****

LLM 的计算就是线性的，持续的。如果直接跳到结论，容易出错（尤其是具有一定的数学或逻辑题）。强制它输出推理步骤，相当于给它**更多的计算时间**和**上下文缓存**，从而提高准确率。

四、实战演练：使用 DeepSeek 生成结构化 JSON**

****任务目标**：我们需要根据一段自然语言描述，让大模型直接生成一个用于前端动态表单渲染的 JSON 配置。

**难点**：AI经常会生成错误的JSON格式（如多余的逗号、错误的字段名），导致 `JSON.parse()` 失败。

**错误的做法：**

### 做法 1：Zero-shot (容易翻车)

> Prompt: 帮我生成一个用户注册表单的 JSON 配置，包含用户名和密码。
>
> 结果: 可能给你返回了一段带注释的 JSON，或者字段名是中文的，或者结构完全不符合你的组件库要求。

**做法1的真实的效果如下：**

**以下是模型给我的数据结构：****

```
{  "formTitle": "用户注册",  "formAction": "/api/register",  "formMethod": "POST",  "fields": [    {      "fieldId": "username",      "label": "用户名",      "type": "text",      "placeholder": "请输入4-16位字母、数字或下划线",      "required": true,      "validations": [        {          "rule": "minLength",          "value": 4,          "message": "用户名至少4个字符"        },        {          "rule": "maxLength",          "value": 16,          "message": "用户名不能超过16个字符"        },        {          "rule": "pattern",          "value": "^[a-zA-Z0-9_]+$",          "message": "只能包含字母、数字或下划线"        }      ]    },    {      "fieldId": "password",      "label": "密码",      "type": "password",      "placeholder": "请输入8-20位密码，需包含字母和数字",      "required": true,      "validations": [        {          "rule": "minLength",          "value": 8,          "message": "密码至少8个字符"        },        {          "rule": "maxLength",          "value": 20,          "message": "密码不能超过20个字符"        },        {          "rule": "pattern",          "value": "^(?=.*[A-Za-z])(?=.*\\d).+$",          "message": "必须包含至少一个字母和一个数字"        }      ]    },    {      "fieldId": "confirmPassword",      "label": "确认密码",      "type": "password",      "placeholder": "请再次输入密码",      "required": true,      "validations": [        {          "rule": "match",          "value": "password",          "message": "两次输入的密码不一致"        }      ]    }  ],  "submitButton": {    "text": "立即注册",    "type": "primary"  }}
```

****现在的大模型给出的东西，确实非常完整，但是这么复杂的数据结构，是不是你真实需要的呢？这个还是带着问号的。这串数据结构确实不错，把我没有想到的都给我考虑进去了，但是真实的项目中，我可能不需要这么复杂的，我大概率就是根据这个数据结构进行删删改改，再使用到我的项目中。**

但是我们的要求是完全符合我们预期的东西，我们不想去修修改改的。

做法 2：Few-shot + CoT + 结构定义 (完美方案)

我们在 DeepSeek (或其他 LLM) 中使用以下 Prompt：**

```
# Role你是一个资深的前端架构师，专注于低代码平台的 Schema 设计。# Context我需要根据用户的描述，生成符合特定规范的 JSON Schema，用于渲染动态表单。# Constraints1. 必须输出标准的 JSON 格式，不要包含其他不符合要求的代码块标记。2. 不要包含任何注释。3. 字段类型 (type) 只能是: 'input' | 'select' | 'checkbox' | 'date'。4. 必须包含 label, key, required 字段。# Few-shot Examples (学习样本)用户描述: "创建一个包含姓名和性别的表单。"AI 回答:[  { "key": "name", "label": "姓名", "type": "input", "required": true },  { "key": "gender", "label": "性别", "type": "select", "options": ["男", "女"], "required": true }]# Task用户描述: "请生成一个活动报名表单。需要包含：1. 活动名称（必填）2. 活动时间（日期选择，必填）3. 参与人数（数字输入，非必填）4. 是否需要接送（复选框）"# Chain of Thought (思维链)请先在心里思考每个字段对应的 type 和 key，确保符合 Constraints，然后直接输出 JSON 结果。
```

**### 预期输出 (DeepSeek):**

```
[  { "key": "activityName", "label": "活动名称", "type": "input", "required": true },  { "key": "activityDate", "label": "活动时间", "type": "date", "required": true },  { "key": "participantCount", "label": "参与人数", "type": "input", "required": false },  { "key": "needPickup", "label": "是否需要接送", "type": "checkbox", "required": false }]
```

**********做法2的真实的效果如下：**

**完全满足我们自身的需求。**

深度解析这个 Prompt：

* Role: 设定了“架构师”人设，暗示输出要专业、严谨。
* Constraints: 明确了“类型系统” (TypeScript Interface)，限制了 AI 的幻觉。

* Few-shot 给了一个简单的例子，锁定了输出的 JSON 数组结构。
* CoT: 虽然我们让它“在心里思考”（为了输出纯净的 JSON），但这个指令会暗示模型进行逻辑校验。

现在主流的一些提示词，基本上都是满足上述的要求的。所以在实际项目实践过程中，我们只需要简单掌握这些基础的就可以了。因为基本上我们开发人员不太可能去思考或者想怎么写提示词，往往我们还是会让AI来干这个事情。

但是实际上我们可以根据我们的描述，让AI大模型给我们写一些类似上面这些规范化的提示词。然后我们在根据需求的要求，进行修改，再结合大模型给出的结果是否符合，我们会有一个不断修改提示词的过程，直到满足我们的要求。

毕竟没有所谓的完美提示词，只有相对准确的提示词。

****五、总结：******

**要想让 LLM 乖乖听话，别只把它当聊天机器人。**

**把它当成一个**只需要提供详细文档和测试用例，就可以自我进行开发的高级资深程序员**，用 Zero-shot 试探，用 Few-shot 规范，用 CoT 提升逻辑。**

**虽然我们可以不需要写很高深的提示词，但是我们必须要会写提示词。一些基础概念与知识是必须要知道的。**

**因为目前现在所有的AI工具中，都是需要提示词来进行交流沟通的，不论是文字生成图片，图片生成视频，或者文字生成视频等工具。我们在日常开发过程中可以收集一些专业性很强的提示词到自己的知识库中去，方便后续可以参考并持续学习，提升自身的能力。**

**关于学习Prompt Engineering (提示词)基础这块，大家可以看下官方给出一些资料：**

* **[OpenAI Prompt Engineering Guide](https://platform.openai.com/docs/guides/prompt-engineering) (官方必读)**
* **[Learn Prompting](https://learnprompting.org/)(系统性教程)**

**或者大家去B站找一下相关的知识点学习一下，毕竟入门的门槛还是蛮低的。让我们一起加油吧！************