---
title: Vercel官方组件组合模式：让React代码告别"布尔地狱"
author: AI小集市
date: 
url: https://mp.weixin.qq.com/s?__biz=MzcxMDA5NzI4MA==&mid=2247484506&idx=1&sn=de19a8f2e93d552efdf4c5a8ace74539&chksm=f4c2fe91f9068bf97acd8d20878b1244d072925106a120e997f330f83f2432b0438b3a21c836&mpshare=1&scene=24&srcid=0310nTNwncYBqUT4NK4NfvXk&sharer_shareinfo=6b2268ea9ddc15b8730bb7b75952c904&sharer_shareinfo_first=6b2268ea9ddc15b8730bb7b75952c904#rd
---

最近逛GitHub，看到Vercel的agent-skills仓库更新了。这项目挺有意思，官方说是"AI的npm"，其实就是把Vercel团队十年React/Next.js经验打包成AI能理解的技能包。今天聊的是`vercel-composition-patterns`技能，安装量已经冲到40.9K了——看来不少开发者都在用。

看到"composition patterns"这词儿，想起自己刚入行时写的那些组件——动不动`isLoading`、`isDisabled`、`isError`一堆布尔值，代码跟迷宫似的。有一次为了改个显示逻辑，我花了整整一下午梳理各种布尔组合。现在好了，Vercel直接把避坑指南做成了技能，AI能帮你自动重构。

## 技能是什么？

说白了，这是一套教你如何构建灵活、可维护React组件的设计模式。核心就一句话：**用组合代替配置**。

比如这种常见的组件写法：

```
<Composer  
  isThread={true}  
  isEditing={false}  
  channelId='abc'  
  showAttachments={true}  
  showFormatting={false}  
/>
```

看着都头疼对吧？光是理解这个组件在不同布尔组合下会渲染什么，就得花半天时间。更可怕的是，每加一个新的布尔属性，可能的状态数就翻一倍——6个布尔属性就有64种可能状态，这谁维护得了？而且测试起来简直是噩梦。

`vercel-composition-patterns`要解决的就是这个问题。它告诉你，别再用布尔属性来控制组件的行为了，改用组合的方式。

## 三大核心模式

### 1. 避免布尔属性泛滥

这是最高优先级规则："Don't add boolean props like `isThread`, `isEditing`, `isDMThread` to customize component behavior."

布尔属性本质上是**配置**，配置越多系统越复杂。正确做法是创建明确的变体组件：

```
// 原来：一个组件，N个布尔属性  
<Composer isThread isEditing={false} />  
  
// 现在：明确的不同组件  
<ThreadComposer channelId="abc" />  
<EditMessageComposer messageId="xyz" />  
<ForwardMessageComposer messageId="123" />
```

这样做代码更清晰、更容易测试、更易维护、复用性更好。

### 2. 使用复合组件

精髓是用共享的上下文把复杂组件拆成小块，让使用者自己组合。

以前写法接口不清晰，灵活性差。现在建议做成复合组件：

```
const ComposerContext = createContext<ComposerContextValue | null>(null)  
  
function ComposerFrame({ children }: { children: React.ReactNode }) {  
  return <form>{children}</form>  
}  
  
function ComposerInput() {  
  const { state, actions: { update }, meta: { inputRef } } = use(ComposerContext)  
  return <TextInput ref={inputRef} value={state.input} onChangeText={(text) => update((s) => ({ ...s, input: text }))} />  
}  
  
// 这样导出，让使用者自由组合  
const Composer = {  
  Provider: ComposerProvider,  
  Frame: ComposerFrame,  
  Input: ComposerInput,  
  Submit: ComposerSubmit,  
}
```

用起来像搭积木，非常灵活。

### 3. 状态管理解耦

把状态管理细节封装在Provider里，UI组件只管消费上下文接口。好处是同样的UI组件，可以配合完全不同的状态实现。

这就是依赖注入的威力——UI组件只依赖接口，不依赖具体实现。这种设计让代码的复用性和可测试性都大大提升。

## 实际应用场景

### 重构"布尔地狱"组件

当组件布尔属性超过3个时，就该考虑用组合模式重构了。比如一个评论组件有`isEditing`、`isReplying`、`showActions`等多个布尔属性，可以拆成`CommentView`、`CommentEdit`、`CommentReply`等独立组件。

### 构建可复用组件库

如果你在设计一套组件库给其他团队用，用复合组件模式能让使用者更灵活地组合。比如一个`DataTable`组件，可以让用户自由组合`DataTable.Header`、`DataTable.Row`、`DataTable.Cell`等子组件，而不是通过一堆`showHeader`、`showFooter`这样的布尔属性来控制。

### 设计灵活API

复合组件模式比一堆配置参数更友好。使用者可以像搭积木一样组合功能，而不是记住各种参数组合。这降低了学习成本，也减少了出错的可能。

### 代码审查

这些规则可以作为检查清单。在审查React组件架构时，发现违反规则的代码就可以建议重构。这有助于保持代码质量的一致性。

## 具体转换示例

让我举个具体的例子。假设你有一个`Button`组件，随着功能增加，变成了这样：

```
<Button  
  primary={true}  
  secondary={false}  
  loading={false}  
  disabled={true}  
  rounded={false}  
  size="medium"  
  iconPosition="left"  
/>
```

这种组件用起来痛苦，维护起来更痛苦。按照Vercel的组合模式，可以这样重构：

```
// 拆分成不同的变体组件  
<PrimaryButton>Click me</PrimaryButton>  
<SecondaryButton disabled>Disabled</SecondaryButton>  
<LoadingButton size="large">Loading...</LoadingButton>  
  
// 或者用复合组件方式  
<Button>  
  <Button.Icon position="left" />  
  <Button.Text>Click me</Button.Text>  
</Button>
```

这样每个组件职责单一，接口清晰，测试和维护都容易得多。

## 踩坑提醒

### 上下文值要memo化

如果在Provider的value里直接传对象字面量，每次渲染都创建新对象，导致所有消费上下文的组件重新渲染。正确做法用`useMemo`包一下：

```
const contextValue = useMemo(() => ({  
  state,  
  actions: { update: setState, submit },  
  meta: { inputRef }  
}), [state, submit])
```

### React 19 API变化

这技能包含了React 19新API规则，但很多人可能还在用React 18。注意：在React 19里，`ref`现在是普通prop，不再需要`forwardRef`包装；`use()`替代`useContext()`。用React 18记得跳过这部分，否则代码会报错。

## 安装使用

技能是Vercel官方`agent-skills`仓库一部分。仓库现在20.8k star，1.9k fork。你可以在GitHub上找到它：vercel-labs/agent-skills

安装：

```
npx skills add vercel-labs/agent-skills --skill composition-patterns
```

AI会自动识别什么时候该用这技能。比如让AI重构React组件，或设计新组件架构时，它会触发技能按最佳实践生成代码。

## One More Thing

技能还藏了个实用技巧：**如何让自定义UI访问组件状态和操作**。

比如设计表单组件，但希望表单外面有预览区域和提交按钮。传统思路得把状态提上来，或用ref同步，都很麻烦。

Vercel方案：只要自定义UI组件在同一个Provider里，就能直接访问状态和操作，不管是不是在组件的"视觉嵌套"里。

组件的边界是Provider，不是视觉嵌套。这样就能很灵活地组合UI了。

## 最后说两句

看到Vercel把这些最佳实践做成AI技能，挺感慨。以前这些经验都得靠时间积累，或跟团队里资深工程师一点一点学。现在一个技能包就能让AI掌握这些模式，对新手是好事。

不过技能再好也只是工具。真正重要的还是理解这些模式背后的设计思想——为什么要用组合代替配置？为什么要解耦状态管理？理解了这些，用起工具才能得心应手。

话说，你最近写React组件时，有没有遇到过"布尔地狱"的情况？或者你有什么好的组件设计经验？欢迎留言聊聊。

觉得这篇文章有帮助？关注 **「AI小集市」**，后续会有更多深度解读和实战案例。我们明天见！