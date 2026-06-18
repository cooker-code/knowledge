---
title: 工具到位了效率自然到位｜OpenWebUI + OpenCode + Ghostty
author: 超记ChaosNote
date: 
url: https://mp.weixin.qq.com/s?__biz=MzA3NDI0MjYzNQ==&mid=2650356988&idx=1&sn=c3647db93c2f9a51efe606aabd8a11e9&chksm=8674de748431c26cabac9db26d42c17d19161f9740f839bab85f51b561b570f386a788af812c&mpshare=1&scene=24&srcid=0409AxWt5XsKXTBWkrRBS3bT&sharer_shareinfo=1cd84e4517a012d0da1c32ad421ce21e&sharer_shareinfo_first=1cd84e4517a012d0da1c32ad421ce21e#rd
---

## 1. OpenWebUI

如果你经常在[多个大模型](https://mp.weixin.qq.com/s?__biz=MzA3NDI0MjYzNQ==&mid=2650355612&idx=1&sn=1e1febb3f2ec82141ff770221c920fea&scene=21#wechat_redirect)之间切换使用，其实 OpenWebUI 是非常推荐的，这样你与各个大模型之间的会话记录数据，不会被各个大模型平台锁定。

当然缺点也很明显，因为是走大模型开放平台的 API，每一次会话都会产生一些费用。随之而来的问题就是要求使用者对各个模型的能力边界比较清楚，知道什么样的任务用什么样的模型，毕竟越贵的模型 API 费用越贵，比如只是信息检索用推理模型就比较贵，当然如果你是大户就当我没说。

 OpenWebUI 现在已支持「工作空间」，支持定制的知识库配置，基本上可以满足个人定制某垂直领域专家的诉求。

如果用 Docker 去跑 OpenWebUI，对机器性能还是有一定要求的，内存至少要 12GB 以上，16GB 比较合适。当然如果你内存有 128GB，就可以安装本地大模型就不需要给 LLM API 掏钱了。

我用 24GB 内存跑 8B 的本地模型，Token 吐字速度很慢，只能说图一乐呵，实际体验上还是接 API 体验最好，除非你的 Token 调用量很大，业务场景不复杂，才需要考虑本地部署一定参数规模的大模型。

用 Mac 系统安装 Docker 和 OpenWebUI 是比较顺畅的，基本 20 分钟以内搞定吧。但用 Windows 安装就比较麻烦，在 Docker 安装 OpenWebUI Image 碰到无法下载镜像的问题。

具体的坑是这样的：打开 Docker Desktop 终端，执行拉取镜像的命令：

```
docker pull ghcr.io/open-webui/open-webui:main 
```

然后就卡住了，一堆 `Pulling fs layer` 纹丝不动，典型因为网络环境拉不动 `ghcr.io` 的镜像。

解决办法有两个：

用国内同步的镜像源直接拉取，有人把 `ghcr.io` 的镜像同步到了华为云，先拉下来再打个标签就行：

```
docker pull swr.cn-north-4.myhuaweicloud.com/ddn-k8s/ghcr.io/open-webui/open-webui:main  docker tag swr.cn-north-4.myhuaweicloud.com/ddn-k8s/ghcr.io/open-webui/open-webui:main ghcr.io/open-webui/open-webui:main 
```

如果你想一劳永逸，也可以在 Docker Engine 里配置镜像加速：Docker Desktop → **Settings** → **Docker Engine**，在 JSON 里加上：

```
{"registry-mirrors":["https://swr.cn-north-4.myhuaweicloud.com","https://docker.mirrors.ustc.edu.cn"]}
```

改完点 **Apply & Restart**，重启后重新 pull 就行了。

## 2. OpenCode

Claude Code 使用的隐形门槛很高。CLI （Command Line Interface， 命令行界面）大部分普通用户是不习惯的，更偏好 ChatUI 界面。 

如果你对 CLI 这类工具第一反应是抗拒，先抛弃这个执念，CLI 相比 Web 端大模型平台不一样的地方就是可以操作本地文件，当然龙虾在这一步上走的更远，你把本地文件的所有权限都交给了龙虾，理论上无尽的信任 + 无穷的 Token，可以让任务代理可以走很远。

CLI 跟龙虾的区别我理解是控制权还是在用户自己手里，LLM 在本地的任何操作都需要用户授权，尤其是敏感操作，比如删除文件、执行某个程序或者脚步。其实对用户常规的计算机知识还是有要求的，你的知道哪些操作是危险的，LLM 在 CLI 交互过程不符合你的期望时要及时干预和调整，澄清自己的诉求。

一个很好的替代品就是 OpenCode，模型你也可以选择自己偏好的，当然花费也是需要走 API 的。

尝试在 OpenCode 里使用 [Deepseek-Reasoner Model](https://mp.weixin.qq.com/s?__biz=MzA3NDI0MjYzNQ==&mid=2650356043&idx=1&sn=f813a5481ff8d462df477bac760b0b3c&scene=21#wechat_redirect) 去写 Python 脚本， 虽然最终效果也能达到效果，但中间等待的时间太长了，至少 10 分钟。同样的任务用 Claude Code 估计 5 分钟内就可以完成。

 Deepseek-Reasoner 更像是用时间去换效果，优点是成本便宜，缺点就是时间长。一旦最终输出效果未达预期，这个沉默的时间成本就很高。

## 3. Obsidian + CLI

Obisidian 作为知识库文件都是以 .md 形式存在，天然对 LLM 是友好的，结合 CLI 插件，你可以通过 CLI 完成个人知识库的各种操作，比如整理文档，编写文档，因为天然与知识库结合，你对 Context 的控制成本变低了很多，Obsidian + CLI 就是个人知识库版本的 Cursor。

Obisidian 插件的自由度很高，无论集成 OpenCode 还是 Claude Code，都需要在 Obisidian 插件市场需要先搜索和安装 **BRAT**。

如果想在 Obsidian 集成 Claude Code，在 BRAT 输入 Github 地址 https://github.com/YishenTu/claudian 。

如果想集成 OpenCode，在BRAT 输入 https://github.com/mtymek/opencode-obsidian 。

在 Windows 系统的 Obsidian 集成 OpenCode，碰到了服务器无法启动的问题，让 AI 排查了很久才定位到问题：OpenCode 本地服务器不允许跨域请求。

Obsidian 的 origin 是 `app://obsidian.md`，OpenCode 服务器默认不认这个来源，请求直接被拦了。

解决方法是手动创建一个 OpenCode 的配置文件，把 Obsidian 加到白名单里。Windows 下用 PowerShell 打开记事本：

```
notepad "$env:USERPROFILE\.config\opencode\opencode.json"
```

提示文件不存在就直接新建，写入以下内容保存：

```
{"server":{"cors":["app://obsidian.md"]}}
```

回到 Obsidian 点 Retry，OpenCode 服务器就能正常连上了。

## 4. Ghostty

CLI 工具有了以后， 肯定要窗口多开的，Mac 的 Terminal 窗口是没法平铺的，我看不少人在用 Tmux，我用了 Ghostty，用下来还不错，配置文件可以参考：https://github.com/BruceLanLan/bruceblue-ghostty-config

当然你也可以根据自己的需要修改配置，剩下的就是你的 Token 能不能跟上了。