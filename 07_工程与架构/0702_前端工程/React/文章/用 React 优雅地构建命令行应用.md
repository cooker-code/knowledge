---
title: 用 React 优雅地构建命令行应用
author: GitHub精选
date: 
url: https://mp.weixin.qq.com/s?__biz=MzAwMzE5NzM2Nw==&mid=2247493616&idx=1&sn=75ffd1028a10a61dca1587500c56b3bf&chksm=9adf62a6933f40b80101ec968a60fa858311a545fc9a83356638442eb93aac59dc457ecf1912&mpshare=1&scene=24&srcid=0410hx7QpiG46Xpl1IID3tei&sharer_shareinfo=7b53dbbfe0107c4175324c447c755b78&sharer_shareinfo_first=7b53dbbfe0107c4175324c447c755b78#rd
---

> 当 React 遇上终端，CLI 开发从未如此优雅。

命令行工具一直是开发者的好帮手，但传统的 CLI 开发方式往往充满了 `console.log` 和字符串拼接的痛苦。如果你习惯用 React 的组件化思维来构建 UI，那么 **Ink** 这个开源项目一定会让你眼前一亮。

## 什么是 Ink？

**Ink** 是一个用 React 构建命令行应用的框架。它提供了与 React 在浏览器中相同的组件化 UI 构建体验，但目标平台是终端。

简单来说：**你会 React，就会用 Ink 写 CLI**。

更棒的是，Ink 使用 Facebook 的 Yoga[1] 引擎在终端中实现 Flexbox 布局，所以大部分 CSS-like 属性都能直接使用。没错，你可以在终端里写 Flexbox！

## 谁在用 Ink？

看看这份名单，你就知道 Ink 有多受欢迎：

* **Claude Code** - Anthropic 的 AI 编程助手
* **Gemini CLI** - Google 的 AI 编程工具
* **GitHub Copilot CLI** - GitHub 的命令行 AI 助手
* **Cloudflare Wrangler** - Cloudflare Workers 的官方 CLI
* **Prisma** - 现代数据库工具链
* **Gatsby** - 现代前端框架
* **Shopify CLI** - 电商平台的开发工具

这些顶级团队都在用 Ink 构建他们的命令行工具。

## 快速上手

### 安装

```
npm install ink react
```

### 创建你的第一个 Ink 应用

使用官方脚手架快速创建：

```
npx create-ink-app my-ink-cli
```

或者 TypeScript 版本：

```
npx create-ink-app --typescript my-ink-cli
```

### 一个简单的计数器示例

```
import React, {useState, useEffect} from 'react';  
import {render, Text} from 'ink';  
  
const Counter = () => {  
  const [counter, setCounter] = useState(0);  
  
  useEffect(() => {  
    const timer = setInterval(() => {  
      setCounter(previousCounter => previousCounter + 1);  
    }, 100);  
  
    return () => {  
      clearInterval(timer);  
    };  
  }, []);  
  
  return <Text color="green">{counter} tests passed</Text>;  
};  
  
render(<Counter />);
```

运行后，你会看到一个绿色数字在终端中不断跳动。就是这么简单！

## 核心特性

### 1. 完整的 React 支持

因为 Ink 是一个 React 渲染器，所以 React 的所有特性都支持：

* Hooks（useState, useEffect, useContext...）
* 组件化思维
* 状态管理
* 生命周期管理

### 2. Flexbox 布局

在终端中使用 Flexbox？是的，你没听错：

```
import {Box, Text} from 'ink';  
  
const Layout = () => (  
  <Box flexDirection="column" padding={1}>  
    <Box marginBottom={1}>  
      <Text bold color="cyan">Header</Text>  
    </Box>  
    <Box flexGrow={1}>  
      <Text>Content here</Text>  
    </Box>  
  </Box>  
);
```

### 3. 丰富的内置组件

Ink 提供了多个实用组件：

* **`<Text>`** - 文本渲染，支持颜色、粗体等样式
* **`<Box>`** - 容器组件，用于布局
* **`<Newline>`** - 换行
* **`<Spacer>`** - 弹性空白
* **`<Static>`** - 静态内容，不会重新渲染

### 4. 实用的 Hooks

Ink 提供了多个专用 Hooks：

```
import {useInput, useApp, useStdout} from 'ink';  
  
const MyCLI = () => {  
  const {exit} = useApp();  
  const {stdout} = useStdout();  
  
  useInput((input, key) => {  
    if (key.escape) {  
      exit();  
    }  
    if (input === 'h') {  
      console.log('Hello!');  
    }  
  });  
  
  return <Text>Press ESC to exit, 'h' to say hello</Text>;  
};
```

常用 Hooks：

* `useInput` - 处理键盘输入
* `useApp` - 控制应用生命周期
* `useStdout` / `useStdin` / `useStderr` - 访问标准流
* `useFocus` - 管理焦点
* `useWindowSize` - 响应终端窗口大小变化

## 实际应用场景

### 1. 交互式命令行工具

```
import {useState} from 'react';  
import {render, Box, Text} from 'ink';  
import TextInput from 'ink-text-input';  
  
const SearchTool = () => {  
  const [query, setQuery] = useState('');  
  
  return (  
    <Box flexDirection="column">  
      <Box>  
        <Text color="cyan">Search: </Text>  
        <TextInput value={query} onChange={setQuery} />  
      </Box>  
      {query && <Text>Searching for: {query}</Text>}  
    </Box>  
  );  
};
```

### 2. 进度显示

```
import {useState, useEffect} from 'react';  
import {render, Text} from 'ink';  
  
const ProgressBar = ({percent}) => {  
  const width = 40;  
  const filled = Math.round(width * percent);  
  const empty = width - filled;  
  
  return (  
    <Text>  
      <Text color="green">{'█'.repeat(filled)}</Text>  
      <Text color="gray">{'░'.repeat(empty)}</Text>  
      <Text> {Math.round(percent * 100)}%</Text>  
    </Text>  
  );  
};
```

### 3. 选择菜单

```
import {useState} from 'react';  
import {render, Box, Text} from 'ink';  
import SelectInput from 'ink-select-input';  
  
const Menu = () => {  
  const items = [  
    {label: 'Create new project', value: 'new'},  
    {label: 'Open existing', value: 'open'},  
    {label: 'Settings', value: 'settings'},  
  ];  
  
  const handleSelect = item => {  
    console.log(`Selected: ${item.value}`);  
  };  
  
  return (  
    <Box flexDirection="column">  
      <Text bold color="yellow">Main Menu</Text>  
      <SelectInput items={items} onSelect={handleSelect} />  
    </Box>  
  );  
};
```

## 生态系统

Ink 有丰富的插件生态：

* `ink-text-input` - 文本输入组件
* `ink-select-input` - 选择菜单
* `ink-spinner` - 加载动画
* `ink-progress-bar` - 进度条
* `ink-table` - 表格展示
* `ink-link` - 可点击链接
* `ink-image` - 终端图片显示

## 测试友好

Ink 应用可以轻松测试，因为它本质上就是 React 组件：

```
import React from 'react';  
import {Text} from 'ink';  
import {render} from 'ink-testing-library';  
import Counter from './Counter';  
  
test('counter', () => {  
  const {lastFrame} = render(<Counter />);  
  // 断言输出内容  
  expect(lastFrame()).toContain('tests passed');  
});
```

## 为什么选择 Ink？

### ✅ 学习成本低

如果你已经会 React，那 Ink 几乎零学习成本。

### ✅ 开发效率高

组件化思维让代码更易组织、复用和维护。

### ✅ 布局能力强

Flexbox 布局让复杂的终端 UI 变得简单。

### ✅ 生态丰富

大量现成组件和 Hooks 可直接使用。

### ✅ 类型友好

完整的 TypeScript 支持。

## 小结

**Ink** 把 React 的优雅带到了命令行世界。如果你正准备开发一个 CLI 工具，不妨试试 Ink，也许你会发现——原来写命令行应用也可以这么享受。

---

**项目信息**

* GitHub: vadimdemedes/ink
* Stars: 26k+
* License: MIT

---

*如果觉得有用，欢迎分享给更多朋友！*