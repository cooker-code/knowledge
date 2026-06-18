---
title: Claude Code 接入 Chrome DevTools：让 AI 直接调试你的网页！
author: 小团子Java
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzk5MDMxMTk4Mg==&mid=2247483733&idx=1&sn=377a75bf4d22e50eab25852eaa343fbe&chksm=c451e555c627f1ee3ae65d9994ecd5d7c4d0df48426853b22e4bf9a3b08c817c9f961ecb87a0&mpshare=1&scene=24&srcid=0414UoGUopNHzB9ZHq0rgmQX&sharer_shareinfo=876d5665f40b41e3f1ef47a7c4ccc812&sharer_shareinfo_first=876d5665f40b41e3f1ef47a7c4ccc812#rd
---

# 🔥 Claude Code 接入 Chrome DevTools：让 AI 直接调试你的网页！

想让 AI 帮你调试网页、检查元素、分析网络请求？今天这篇教程，带你用 **Claude Code + Chrome DevTools MCP**，让 AI 拥有"浏览器超能力"！

---

## 一、Chrome DevTools MCP 是什么？

### 简单理解

**Chrome DevTools MCP** 就是让 Claude Code 能直接控制 Chrome 浏览器的桥梁！

没有 MCP 时：

* • 你："AI，帮我看看这个网页有什么问题"
* • AI："你把网页截图发给我，或者描述一下..."

有了 MCP 后：

* • 你："AI，打开这个网页，检查登录按钮为什么不工作"
* • AI：（直接打开浏览器 → 点击按钮 → 查看 Console → 分析 Network → 找出问题）

### 它能做什么？

| 功能 | 说明 |
| --- | --- |
| 🌐 **打开/导航网页** | 访问任意 URL，刷新、后退、前进 |
| 🔍 **检查元素** | 读取 DOM 结构、CSS 样式 |
| 📝 **修改内容** | 点击按钮、填写表单、输入文本 |
| 📊 **查看 Console** | 读取日志、错误信息 |
| 🔗 **分析网络** | 查看请求、响应、加载时间 |
| 📱 **模拟设备** | 切换移动端、桌面端视图 |
| 📸 **截图** | 截取页面或特定元素 |

### 核心特性（官方版）

| 特性 | 说明 |
| --- | --- |
| **性能分析** | 使用 Chrome DevTools 录制 trace，提取性能洞察 |
| **高级调试** | 分析网络请求、截图、检查 Console（带 source-map 堆栈跟踪） |
| **可靠自动化** | 使用 Puppeteer 自动化 Chrome 操作 |

---

## 二、准备工作

在开始之前，确保你有：

1. 1. ✅ **Claude Code 已安装**
2. 2. ✅ **Node.js 已安装**（版本 18+）
3. 3. ✅ **Chrome 浏览器**（稳定版）

> ⚠️ 注意：如果你之前已经安装过 Chrome DevTools MCP，请先从配置文件中移除旧的配置！

---

## 三、安装 Chrome DevTools MCP（官方最新方法）

官方提供了 **两种安装方式**，推荐用 **Plugin 方式**（包含 MCP + Skills）！

---

### 方法一：Plugin 方式（推荐 ⭐⭐⭐⭐⭐）

这种方式不仅安装 MCP，还会安装配套的 Skills，让 AI 更懂怎么用 Chrome DevTools！

#### 步骤 1：添加 Marketplace Registry

在 Claude Code 中输入：

```
/plugin marketplace add ChromeDevTools/chrome-devtools-mcp
```

#### 步骤 2：安装 Plugin

接着输入：

```
/plugin install chrome-devtools-mcp
```

#### 步骤 3：重启 Claude Code

完全退出 Claude Code，然后重新启动。

#### 步骤 4：验证安装

重启后，输入：

```
/skills
```

你应该能看到 Chrome DevTools 的 Skills 已经加载了！

---

### 方法二：CLI 命令行方式

如果你更喜欢用命令行，可以用 `claude mcp add` 命令：

```
claude mcp add chrome-devtools --scope user npx chrome-devtools-mcp@latest
```

然后重启 Claude Code 即可。

---

### 方法三：手动配置（备用方案）

如果以上方法都不行，可以手动编辑配置文件。

#### 步骤 1：找到配置文件

配置文件位置：

* • **macOS/Linux**: `~/.claude/settings.json`
* • **Windows**: `%USERPROFILE%\.claude\settings.json`

#### 步骤 2：添加 MCP 配置

在 `settings.json` 中添加：

```
{  
  "mcp": {  
    "servers": {  
      "chrome-devtools": {  
        "command": [  
          "npx",  
          "-y",  
          "chrome-devtools-mcp@latest",  
          "--autoConnect",  
          "--channel=stable"  
        ]  
      }  
    }  
  }  
}
```

**参数说明：**

* • `--autoConnect`: 自动连接到 Chrome 浏览器
* • `--channel=stable`: 使用 Chrome 稳定版（可选：`beta`、`dev`、`canary`）

#### 步骤 3：重启 Claude Code

保存配置文件后，完全退出再重新启动。

---

## 四、验证连接是否成功

重启 Claude Code 后，输入：

```
/mcp
```

你应该能看到 `chrome-devtools` 在已连接的 MCP 服务器列表中！

如果看到类似这样的输出，说明成功了：

```
✓ chrome-devtools (connected)  
  - navigate_page  
  - take_screenshot  
  - click  
  - fill  
  - ...
```

---

## 五、实战：让 AI 帮你调试网页

### 例子 1：打开网页并截图

对 Claude Code 说：

```
打开 https://example.com，然后截取整个页面的截图
```

AI 会：

1. 1. 打开 Chrome 浏览器
2. 2. 导航到 example.com
3. 3. 截取页面截图
4. 4. 展示给你看

### 例子 2：检查页面元素

```
打开 https://example.com，找到页面上的所有按钮，告诉我它们的文本和位置
```

AI 会：

1. 1. 打开网页
2. 2. 扫描 DOM 找出所有 `<button>` 元素
3. 3. 列出每个按钮的文本、CSS 选择器、位置

### 例子 3：填写表单并提交

```
打开 https://your-login-page.com：  
1. 在邮箱输入框填入 test@example.com  
2. 在密码输入框填入 password123  
3. 点击登录按钮  
4. 查看 Console 有没有错误
```

AI 会一步步帮你完成！

### 例子 4：分析网络请求

```
打开 https://example.com，刷新页面，然后告诉我：  
1. 页面加载用了多长时间  
2. 有哪些网络请求  
3. 有没有请求失败
```

AI 会分析 Network 面板，给你详细报告！

### 例子 5：模拟移动端

```
打开 https://example.com，切换到 iPhone 14 视图，然后截图
```

---

## 六、常用功能清单

### 🌐 页面导航

| 操作 | 示例提示词 |
| --- | --- |
| 打开 URL | "打开 https://google.com" |
| 刷新页面 | "刷新当前页面" |
| 后退 | "返回上一页" |
| 前进 | "前进到下一页" |

### 🔍 元素操作

| 操作 | 示例提示词 |
| --- | --- |
| 点击元素 | "点击那个写着'提交'的按钮" |
| 填写输入框 | "在搜索框输入'hello world'" |
| 悬停 | "鼠标悬停在导航菜单上" |
| 拖拽 | "把这个元素拖到那边" |

### 📊 信息获取

| 操作 | 示例提示词 |
| --- | --- |
| 查看 Console | "读取 Console 中的所有日志" |
| 查看 Network | "列出所有网络请求" |
| 获取页面源码 | "给我当前页面的 HTML" |
| 获取元素样式 | "告诉我这个按钮的 CSS 样式" |

### 📸 截图

| 操作 | 示例提示词 |
| --- | --- |
| 全页截图 | "截取整个页面" |
| 元素截图 | "只截取登录表单那个区域" |
| 视口截图 | "截取当前可见区域" |

---

## 七、高级技巧

### 技巧 1：用 --autoConnect 连接你正在用的 Chrome（推荐！）

不想让 AI 开新窗口？让 AI 直接用你已经打开的 Chrome！

**在哪加 `--autoConnect` 参数？**

取决于你用哪种安装方式：

#### 方式 A：CLI 命令行方式（加在命令最后）

```
claude mcp add chrome-devtools --scope user npx chrome-devtools-mcp@latest --autoConnect
```

#### 方式 B：手动配置方式（加在 settings.json 的 args 里）

```
{  
  "mcp": {  
    "servers": {  
      "chrome-devtools": {  
        "command": [  
          "npx",  
          "-y",  
          "chrome-devtools-mcp@latest",  
          "--autoConnect"  
        ]  
      }  
    }  
  }  
}
```

**使用步骤：**

1. 1. 在 Chrome 中打开 `chrome://inspect/#remote-debugging`
2. 2. 按提示开启远程调试
3. 3. 配置好 `--autoConnect` 参数
4. 4. 重启 Claude Code

这样 AI 就会直接连接到你正在用的 Chrome，而不是开新窗口！

> ⚠️ 注意：需要 Chrome 144 或更高版本

---

### 技巧 2：指定 Chrome 通道

如果你想用 Chrome Beta 或 Dev 版本：

```
{  
  "mcp": {  
    "servers": {  
      "chrome-devtools": {  
        "command": [  
          "npx",  
          "-y",  
          "chrome-devtools-mcp@latest",  
          "--channel=beta"  
        ]  
      }  
    }  
  }  
}
```

可选值：`stable`（默认）、`beta`、`dev`、`canary`

---

### 技巧 3：关闭使用统计（隐私选项）

这个工具默认会收集使用数据，可以关掉：

```
{  
  "mcp": {  
    "servers": {  
      "chrome-devtools": {  
        "command": [  
          "npx",  
          "-y",  
          "chrome-devtools-mcp@latest",  
          "--no-usage-statistics"  
        ]  
      }  
    }  
  }  
}
```

---

### 技巧 4：用"精简模式"（Slim Mode）

如果你只需要基础功能，可以用精简模式，只保留 3 个工具：

```
{  
  "mcp": {  
    "servers": {  
      "chrome-devtools": {  
        "command": [  
          "npx",  
          "-y",  
          "chrome-devtools-mcp@latest",  
          "--slim"  
        ]  
      }  
    }  
  }  
}
```

精简模式只有：

* • 导航网页
* • 运行脚本
* • 截图

---

### 技巧 5：配合性能分析

```
打开 https://your-site.com：  
1. 开始性能录制  
2. 刷新页面  
3. 停止录制  
4. 分析性能瓶颈在哪里
```

---

## 八、常见问题

### Q: Plugin 安装失败，提示 "Failed to clone repository"？

A: 这通常是网络问题（比如企业防火墙）。可以：

1. 1. 尝试用 CLI 命令行方式（方法二）
2. 2. 或者用手动配置方式（方法三）
3. 3. 配置代理后再试

### Q: 提示 "Chrome not found" 怎么办？

A: 确保：

1. 1. Chrome 浏览器已安装
2. 2. 如果是 macOS，Chrome 在 `/Applications/Google Chrome.app`
3. 3. 可以尝试指定 Chrome 路径：`--chrome-path=/path/to/chrome`

### Q: 连接后没反应？

A: 检查：

1. 1. 确认配置文件格式正确（JSON 语法）
2. 2. 重启 Claude Code
3. 3. 输入 `/mcp` 确认连接状态

### Q: 能同时连接多个浏览器吗？

A: 可以！为每个浏览器配置不同的 MCP 服务器：

```
{  
  "mcp": {  
    "servers": {  
      "chrome-stable": {  
        "command": ["npx", "chrome-devtools-mcp", "--autoConnect", "--channel=stable"]  
      },  
      "chrome-beta": {  
        "command": ["npx", "chrome-devtools-mcp", "--autoConnect", "--channel=beta"]  
      }  
    }  
  }  
}
```

### Q: 国内访问 npm 很慢？

A: 可以使用淘宝镜像：

```
# 临时使用  
npx --registry=https://registry.npmmirror.com chrome-devtools-mcp  
  
# 或者配置 npm 镜像  
npm config set registry https://registry.npmmirror.com
```

---

## 九、官方资源

* • 📚 **GitHub 仓库**: https://github.com/ChromeDevTools/chrome-devtools-mcp
* • 🎯 **Chrome DevTools 文档**: https://developer.chrome.com/docs/devtools/
* • 🔧 **DevTools Protocol**: https://chromedevtools.github.io/devtools-protocol/

---

## 写在最后

以前调试网页，你要自己开 DevTools、查 Console、看 Network；现在让 Claude Code 帮你做这些重复性工作，你专注于解决问题本身。

这就是 AI 时代的调试——**让 AI 当你的"浏览器助手"**。

快去试试吧！有问题评论区留言～