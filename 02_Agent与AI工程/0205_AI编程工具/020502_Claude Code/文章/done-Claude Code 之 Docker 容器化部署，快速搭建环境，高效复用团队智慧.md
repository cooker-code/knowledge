> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Claude Code 之 Docker 容器化部署，快速搭建环境，高效复用团队智慧
author: Asher同学
date:
url: https://mp.weixin.qq.com/s?__biz=Mzk3NTcyMzA0Nw==&mid=2247484178&idx=1&sn=69501a2557a8c9d431df3f4cdb2c7fc5&chksm=c58833128df2de7c068dcf5e3964298b0979e31d238434e41c1606581405cdc2a552ec574063&mpshare=1&scene=24&srcid=1014O1LYtQbXGlpk17FoaSZA&sharer_shareinfo=29873202a5a1b59ea1e0d8e1d7c4a6ed&sharer_shareinfo_first=29873202a5a1b59ea1e0d8e1d7c4a6ed#rd
---

后台有一些粉丝朋友说自己是文科生，但确实对 AI 很感兴趣。其中一些粉丝和我说，在我看来很简单的东西，对他们来说却并不简单，尤其是环境和工具的配置。

其实即使对科班生来讲，配置环境也是一件具有一定复杂度的事情。比如有的人装 Java，一口气就装好了，但有的人就是会遇到各种各样奇奇怪怪的问题，要么是少了环境变量的配置，要么是路径配置错误。

那我们的前辈有没有思考过这种问题呢？只要有痛点就会有人去解决，所以当然有，也就是我们今天的主角：Docker。

Docker 的一个核心设计思想就是用代码去定义环境。代码是高度可自定义的，所以别人可以基于我们的代码进行二次开发，自定义自己需要的环境和工具。因此，Docker 是这个时代我觉得每个人都有必要去学习的一个工具（如果不会，就让 AI 教你）。

另外，代码可以通过文件的形式存储和传播，借助一些版本管理工具可以方便地实现文件的版本管理，比如 Git。有了版本管理机制，团队不同成员就可以通过使用相同的版本进行开发，从而解决了基础环境不一致的问题，而这正是 Docker 的巨大魅力所在。

所以我们今天要讲的主题就是如何使用 Docker 去管理 Claude Code 的环境和配置问题，实现我这边一次定义，别人就可以复用和我相同环境的效果。

另外，Claude Code 这个工具在设计时就考虑到了后续的拓展性和自定义性。

比如系统提示词，我们可以通过 Output Style 相关的文件去自定义；

对于用户提示词，我们可以通过 Claude.md 文件去自定义；

而 Agent，我们可以通过 ./claude/agents 文件夹进行管理。

这和 Augment 以及 Cursor 的设计哲学完全不一样，就像程序员喜欢使用 VS Code 一样，丰富的插件系统让程序员有了更多自定义的空间。

在 Claude Code 这个开源项目上，官方已经为我们提供了一个现成的 Dockerfile，同时这个 Dockerfile 可以配合 VS Code 的插件一起使用。最后的效果就是，对于一个计算机小白来说，只需要在本地装一个桌面版本的 Docker 和 VS Code 编辑器，然后把 VS Code 的插件装好，在 VS Code 里把代码文件一打开，就会自动提示进行一些傻瓜式的操作，可以说基本没有特别复杂的操作。

好，开始我今天的分享。

## 我们要解决什么问题？

1. 1. 计算机小白如何快速使用 Claude Code ？
2. 2. 团队如何复用共享 Claude Code 的提示词文件，Agent 工具等等？（太长了，下一讲讲，敬请期待）

## 如何解决这个问题？

1. 1. 工具和环境通过 Dockerfile 固化
2. 2. 提示词文件和 Agent 工具都可以通过 Git 仓库进行管理和传播。

## 今天的实验大纲（全部资料和工具都免费）

1. 1. 下载 Docker
2. 2. 下载 VScode
3. 3. 下载 VScode 插件
4. 4. 拉取官方代码仓库，进行简单修改
5. 5. 在Docker 容器里面使用Claude Code
6. 6. 释放 AI 的潜力

## 操作1：下载 Docker

进入官网，https://www.docker.com/

根据你的电脑下载对应的版本，然后直接安装图形化安装就好了

安装之后记得配一下国内的一些镜像（registry-mirrors），提升下载速度

```
{
  "builder": {
    "gc": {
      "defaultKeepStorage": "20GB",
      "enabled": true
    }
  },
  "experimental": false,
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://docker.unsee.tech",
    "https://docker.1panel.live",
    "http://mirrors.ustc.edu.cn",
    "https://docker.chenby.cn",
    "http://mirror.azure.cn",
    "https://dockerpull.org",
    "https://dockerhub.icu",
    "https://hub.rat.dev"
  ]
}
```

修改镜像方法如下：

## 操作2：下载 Vscode

官网：https://code.visualstudio.com/Download

根据你的电脑下载对应的版本，然后直接安装图形化安装就好了

## 操作3：下载 Vscode插件

打开VSCode

方式1：通过打开网页下载：https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers

方式2：插件里面搜索 Dev Containers

## 操作4：下载官方代码仓库，进行简单修改

官方仓库地址：https://github.com/anthropics/claude-code

这个目录下面就是我们今天用的文件：https://github.com/anthropics/claude-code/tree/main/.devcontainer

我们本地下载好之后，为了实验简单，先关掉防火墙相关的配置

修改文件 devcontainer.json

修改文件 Dockerfile

## 操作5：启动 Docker 版本的 Claude Code

通过 Vscocde 打开我们下载的仓库

如果前面操作都完成了，右下角就会弹出这个框（如图所示）

选择 『Reopen in Container』

如果右下角的弹框消失了，这样找

稍等片刻之后，等容器环境安装好之后，我们看到左下角是这个样子，基本就成功了。

## 操作5：在Docker 容器里面使用 Claude Code

### 方式 1：使用终端

打开终端

从终端进入 CC（不出意外的话，你还是会出意外，因为你要使用国内的大模型或者镜像站，国内开发道路重重，关关难过关关过！）

怎么办？解决呗

第一步：在 .claude 目录下面创建  settings.json， 如下图所示：（主要是通过 apiKeyHelper 激活 API key 的识别）

文件内容(ANTHROPIC\_BASE\_URL 使用的是智谱，根据需要自行替换)：

```
{
  "apiKeyHelper": "/workspace/.claude/anthropic_key.sh",
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "env": {
    "DISABLE_TELEMETRY": "1",
    "DISABLE_ERROR_REPORTING": "1",
    "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1",
    "DISABLE_AUTOUPDATER": "1",
    "ANTHROPIC_BASE_URL": "https://open.bigmodel.cn/api/anthropic",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "glm-4.5-air",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "glm-4.6",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "glm-4.6"
  }
}
```

第二步：在 .claude 目录下面创建 anthropic\_key.sh 文件

文件内容如下：

```
echo "替换成你的 API Key"
```

第三步，终于可以使用了

### 方式 2 ：使用 VS Code Claude Code 插件，对小白友好（API 配置同上）

安装 Claude Code for VS Code 插件

使用插件提供的界面

不出意外的话，你的这里一定会出意外！

放心，我已经给你踩过坑了，之需要配置用户目录下面的一个文件，参考这个文章 [修复 VScode Claude Code 插件界面无法打开的问题](https://mp.weixin.qq.com/s?__biz=Mzk3NTcyMzA0Nw==&mid=2247483930&idx=1&sn=34ee843f6d006e8377f8ca240a6fc4cf&scene=21#wechat_redirect)

## 内容小结

通过Docker和VS Code的完美结合，我们成功实现了Claude Code环境的快速部署和团队共享，让文科生也能轻松驾驭AI开发环境，真正做到了"一次定义，处处运行"，极大降低了技术门槛并提升了协作效率。

---

作者介绍

我是 Asher 同学，一个喜欢独立深入思考，热爱技术和编程，对代码质量有追求，喜欢分享的编程创作者。

目前在深耕人工智能领域，会长期更新人工智能和软件工程相关的内容。

写作不易，如果文章对你有帮助，记得点赞，转发，推荐一键三连哦！🌹🌹

如需交流学习，请移步公众号后台，添加小编好友，邀请进群