> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Claude Code CLI支持LSP了，编程向智能再进一步
author: 程序员小溪
date:
url: https://mp.weixin.qq.com/s?__biz=MzU5Njg3ODUzOA==&mid=2247494574&idx=1&sn=14578d4686f282a1eaddfaca9c56c8f5&chksm=ff9562bfc8d9a23e0be6b4eaf453c5f96d662d3b2774a990f8500ca80c4de0bbe0a6d7c17d45&mpshare=1&scene=24&srcid=04107Dx2NJLaRYja8zsFdgqt&sharer_shareinfo=ef46e351cabd56494f8c06d0e5698538&sharer_shareinfo_first=ef46e351cabd56494f8c06d0e5698538#rd
---

前言

小伙伴们大家好，我是小溪，见字如面。Claude Code CLI V0.27.4版本加入了LSP支持，从此AI编程不再是盲目的通过Grep检索，而是像IDE一样可以做到精准跳转定义、查找引用和实现。

当前使用版本

2.0.76 (Claude Code)

省流版本

* 在命令行终端使用 ENABLE\_LSP\_TOOLS=1 claude 命令开启LSP功能
* 启动Claude Code CLI，运行 /plugin 安装你的语言插件
* 安装对应的LSP语言服务器
* 重启Claude Code
* “使用 LSP 查找 XXX 的所有引用”

了解LSP

LSP简介

LSP（Language Server Protocol）是微软于2016年推出的一种标准化JSON-RPC通信协议，用于连接代码编辑器（如VS Code、Vim）与独立的语言服务器，提供自动补全、跳转定义、语法检查、重构等智能功能 。它将语言分析逻辑从编辑器中解耦，避免了为每种语言和编辑器重复开发（从n×m复杂度降至n+m），支持双向通信，服务器可主动推送诊断结果 。

用白话说就是一套可以让代码变得智能的语言服务协议，通过LSP可以实现一系列的功能，例如编辑器中我们每天都会用到但是不知道如何实现的 跳转到定义、悬停显示文档、实时报错和警告 等功能。

LSP工作原理

LSP的工作原理大致如下：

更多LSP相关内容可以查看LSP官方文档：https://microsoft.github.io/language-server-protocol

英文不太好以及对LSP语言服务器开发感兴趣的小伙伴也可以查看LSP中文文档：https://lsp.fosp.cc/lsp/language-features/goto-definition.html

LSP的优势

在Claude Code CLI V0.27.4版本之前，Claude Code检索内容的方式主要是 grep（Grep 是一个 Linux/Unix 系统的文本搜索工具，全称为 "global regular expression print"，用于在文件或输入流中查找匹配指定模式（字符串或正则表达式）的行），翻译成白话就是通过全文本检索的形式进行查找。

例如我们让Cluade Code “查找项目中saveProgress函数的所有调用”，可以看到Claude Code的匹配流程：

* 读取项目文件，通过搜索匹配包含“saveProgress”的文件
* 读取匹配到的文件内容，从文件内容中匹配“saveProgress”关键词所在的代码行

到这里应该有不少小伙伴会有疑问了，AI查找一个函数的调用有这么费劲吗？AI模型训练时使用了海量资源和代码，目前已经能写出相当不错的代码了，它还能不懂代码？还真是，对于AI来说它没有代码的概念，我们所提供的代码上下文以及AI输出的所有代码在AI看来都是文本，这就是为什么在引入LSP之前AI查找代码或者关键词时耗时的原因。

那么引入LSP后Claude Code会有什么变化呢？

* 有了LSP，代码相关的问题可以直接问语言服务器，相当于为Claude Code装上了语言中枢，Cluade Code将真正能读懂代码，准确、快速定位代码。
* 避免全局Grep查找，在一定程度上可以降低toknes消耗

LSP的核心能力

此次Claude Code引入LSP主要带来了以下 7大 核心能力：

* goToDefinition：跳转到定义，一键跳转跳到函数定义
* goToImplementation：跳转实现，一键跳转跳到函数实现
* findReferences：查找引用，查找所有引用的位置
* hover：悬停信息，鼠标悬停展类型信息、文档注释、函数签名等信息
* documentSymbol：文档符号，以结构化列表形式展示所有符号信息
* workspaceSymbol：工作区符号搜索，在整个项目中检索符号信息
* incomingCalls/outgoingCalls：调用链，详细展示函数前后调用关系

LSP使用场景

LSP不是万能的，选对使用场景才能事半功倍。LSP更适合在以下场景中使用，不仅提效还是节省tokens：

* 大型项目：文件数超过50，代码行超过5000
* 复杂项目：项目业务关系、依赖、调用复杂的项目
* 重构项目：需要重构、频繁改名或者转技术栈的项目
* 跨文件调用：跨文件调用多的项目

在Claude Code中使用LSP

Claude Code可以在IDE和命令行终端等不同的场景中使用，Claude Code搭配LSP的使用方式也略有不同。

在IDE中使用

这里以Cursor为例，首先在命令行终端输入 /claude 命令启动Claude Code CLI

```
ENABLE_LSP_TOOL=1 claude或者export ENABLE_LSP_TOOLS=1claude
```

输入交互式命令 /config 查看配置，调整【Auto-connect to IDE (external terminal)】为【true】，该配置主要用来检查和安装【Claude Code for VS Code】插件的，对该插件还不了解的小伙伴可以看往期内容：[Claude Code CLI初体验](https://mp.weixin.qq.com/s?__biz=MzU5Njg3ODUzOA==&mid=2247492929&idx=1&sn=043e21a18f31f1c4c68bf4ee63e28685&scene=21#wechat_redirect)

正常情况下，Claude Code CLI会自动执行如下操作：

* 检查IDE环境
* 安装 Claude Code for VS Code 插件
* 通过 Claude Code for VS Code 插件连接到IDE的LSP服务

如果发现IDE没有自动下载插件，也可以自行在插件市场下载

配置完成后，可以输入提示词验证Claude Code是否具有LSP调用能力

接着输入提示词验证

```
查看 receiveCredit 的所有引用
```

可以看到很快Claude Code就给出了准确的结果，给出了函数的 定义位置、导入位置 和 调用位置，并且所有的文件路径都是可以点击的

点击文件路径可以跳转到对应的代码行

经多次使用发现这种方式不是很稳定，即使指定Claude Code CLI强制使用LSP也无法限制Claude Code CLI每次都使用LSP。

在Claude CLI中使用

1）安装语言服务器插件

|  |
| --- |
| 语言服务器插件是语言服务器和模型之间沟通的桥梁 |

打开 Claude Code CLI，输入交互式命令 /plugin，搜索lsp看是否可以检索到语言服务器插件

如果没有检索到lsp相关内容，需要先安装 anthropics/claude-plugins-official 插件市场，安装方式可以查看往期内容：[Claude Code上线插件系统，AI编程模式再次升级](https://mp.weixin.qq.com/s?__biz=MzU5Njg3ODUzOA==&mid=2247493269&idx=1&sn=4775c90a4fab04126502bf79b0f75d70&scene=21#wechat_redirect)，安装完成后就可以检索到语言服务器插件了。

Anthropic官方插件：https://github.com/anthropics/claude-plugins-official

如果官方也没有找到也可以尝试使用社区插件：https://github.com/Piebald-AI/claude-code-lsps?tab=readme-ov-file#claude-code-lsps

根据描述选择自己需要的语言服务器插件，回车进入插件安装详情

选择自己需要的安装模式，对Claude Code CLI基础还不了解的小伙伴可以看往期内容：[Claude Code CLI初体验](https://mp.weixin.qq.com/s?__biz=MzU5Njg3ODUzOA==&mid=2247492929&idx=1&sn=043e21a18f31f1c4c68bf4ee63e28685&scene=21#wechat_redirect)

安装成功后可以在【Installed】查看

2）安装语言服务器

|  |
| --- |
| 语言服务器是真正为语言提供能力的核心 |

下面是一些常用语言服务器的安装方式

Python:

```
$ pip install pyright或者$ npm install -g pyright
```

TypeScript/JavaScript:

```
$ npm install -g typescript-language-server typescript或者$ npm install -g @vtsls/language-server typescript
```

HTML和CSS

```
$ npm install -g vscode-langservers-extracted
```

Go:

```
$ go install golang.org/x/tools/gopls@latest
```

Rust:

```
$ rustup component add rust-analyzer
```

Ruby:

```
$ gem install ruby-lsp
```

这里我以typescript服务器为例进行安装，直接在命令行终端输入安装命令

使用下面命令验证语言服务器是否安装成功

```
$ which typescript-language-server$ typescript-language-server --version
```

3）基本使用

安装完成后使用以下命令重启Claude Code CLI

```
$ ENABLE_LSP_TOOL=1 claude或者$ export ENABLE_LSP_TOOL=1$ claude
```

直接输入提示词或者在提示词中添加“使用LSP”关键词向Claude Code CLI提问，可以看到Claude Code CLI先后调用了LSP的 goToDefinition、hover 和findReferences 能力获取信息

然后根据获取到的信息进行提炼总结

cclsp社区方案

如果觉得手动配置LSP语言服务器麻烦，也可以选择使用社区提供的LSP方案，使用交互式命令一键配置。在命令行终端输入如下命令进行初始化配置

```
$ npx cclsp@latest setup
```

选择支持的语言类型，支持多选

配置完成后将会在项目根目录生成 ./claude/cclsp.json 配置文件，同时会添加 cclsp MCP服务

初始化完成后就可以使用cclsp提供的LSP服务了

自定义语言插件

|  |
| --- |
| 目前版本，LSP通过单独插件形式可以生效，在已有插件中配置好像不生效 |

创建一个Claude Code插件，添加配置如下：

```
{  "name": "typescript-lsp",  "description": "TypeScript/JavaScript language server",  "version": "0.0.1",  "author": {    "name": "程序员小溪",    "email": "邮箱"  },  "source": "./typescript-lsp",  "strict": false,  "lspServers": {    "typescript": {      "command": "typescript-language-server",      "args": ["--stdio"],      "extensionToLanguage": {        ".ts": "typescript",        ".tsx": "typescriptreact",        ".js": "javascript",        ".jsx": "javascriptreact",        ".mts": "typescript",        ".cts": "typescript",        ".mjs": "javascript",        ".cjs": "javascript"      }    }  }}
```

对于Claude Code插件还不了解的小伙伴可以看往期内容：[如何从零开始创建一个Claude Code插件](https://mp.weixin.qq.com/s?__biz=MzU5Njg3ODUzOA==&mid=2247493288&idx=1&sn=0274d7f3a741cbfde4d09adf7a595efb&scene=21#wechat_redirect)，更多参数可以参阅官方文档：https://code.claude.com/docs/en/plugins-reference#lsp-servers

添加应用市场，选择LSP语言服务器插件安装

安装完成后，在【Installed】中可以查看安装状态

重启Claude Code CLI，输入提示词即可使用（注意这里也是需要提前安装语言服务器的）

常见问题

语言服务器是否启动

查看语言服务器是否启动，可以在命令行终端中输入如下命令

```
$ grep "LSP notification handlers registered" ~/.claude/debug/latest
```

Claude Code CLI没有LSP工具调用权限

如果发现Claude Code没有调用LSP，可以尝试询问AI是否有调用LSP工具的权限

```
do you hava access to LSP tools? 
```

如果没有调用LSP工具权限可以手动进行开启

```
ENABLE_LSP_TOOL=1 claude
```

也可以使用命令永久开启

```
# 以zsh 为例echo 'export ENABLE_LSP_TOOL=1' >> ~/.zshrcsource ~/.zshrc
```

开启后再次询问Claude Code CLI验证

 

---

点击关注，及时接收最新消息