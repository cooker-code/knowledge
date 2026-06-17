---
title: 一次pip install，你的所有密钥就被偷走了——LiteLLM供应链投毒事件深度复盘
author: 老杨的AI杂货铺
date: 
url: https://mp.weixin.qq.com/s?__biz=MzY4MTAzMDAwNw==&mid=2247483821&idx=1&sn=d599a57d42255f4693ddae05f68cd001&chksm=f2a2a8d4bb3ab74e62a40f8318e424af10205b79e6571dcb276fc189c867bb92de34e1ee3f16&mpshare=1&scene=24&srcid=04170FFOSTYhbhH0n1duwHBP&sharer_shareinfo=3a4755edf358569d94653d5eeef3f1c9&sharer_shareinfo_first=3a4755edf358569d94653d5eeef3f1c9#rd
---

2026年3月 · 安全事件复盘

2026年3月24日，一个每月下载量超过9500万次的Python库被植入了窃密后门。一条简单的`pip install litellm`命令，就足以让你的SSH密钥、云平台凭证、Kubernetes配置、加密货币钱包全部外泄。这不是演习，这是真实发生的事。

···

## 一、LiteLLM是什么？为什么它如此重要

在大模型应用开发的世界里，**LiteLLM**几乎是"基础设施级"的存在。

它是由BerriAI开发的开源Python库和代理网关，核心功能是提供一个**统一的OpenAI兼容接口**，让开发者用一套代码调用100多个LLM提供商——OpenAI、Anthropic、Google Vertex AI、AWS Bedrock、Azure、Cohere、HuggingFace……你能想到的几乎都支持。

它解决的痛点非常实际：

* ▪**统一API格式**

  不用为每个模型商写不同的调用代码
* ▪**自动负载均衡和故障转移**

  主模型挂了自动切到备选
* ▪**成本追踪和限流**

  精细化管理Token消耗和API支出
* ▪**可自托管**

  敏感数据不出内网

正因如此，LiteLLM不仅被个人开发者广泛使用，还深度嵌入了大量企业级AI应用的技术栈。更关键的是，像**DSPy**这样的知名框架也将LiteLLM作为依赖项——这意味着你可能根本没有直接安装LiteLLM，但它已经在你的环境里了。

### 1.1 同类工具一览

LiteLLM并非没有替代品，市场上有多种类似定位的工具：

| 工具 | 定位 | 特点 |
| --- | --- | --- |
| **OpenRouter** | 托管型API网关 | SaaS服务，无需自建基础设施 |
| **Portkey** | 企业级AI网关 | 强调可观测性和治理能力 |
| **Kong AI Gateway** | API网关扩展 | 基于成熟的Kong网关生态 |
| **Helicone** | 可观测性平台 | 专注LLM调用的监控和分析 |
| **Vercel AI SDK** | 前端AI集成 | 与Vercel生态深度绑定 |

但LiteLLM凭借**开源免费、功能全面、社区庞大**三大优势，占据了绝对的市场份额。这也让它成为了攻击者眼中的"高价值目标"。

···

## 二、攻击全链路还原：一场精心策划的"套娃"入侵

攻击者不是直接攻击LiteLLM，而是先攻陷了它的安全扫描工具，再借此攻入LiteLLM。

图1：TeamPCP供应链攻击完整链路

### 2.1 第一步：攻陷Trivy安全扫描器

**Trivy**是Aqua Security开发的开源安全扫描器，广泛用于CI/CD流水线中扫描容器镜像和代码漏洞。讽刺的是，这个本应保护你安全的工具，反而成了攻击入口。

2026年2月底，攻击组织**TeamPCP**利用Trivy GitHub Actions环境中的一个配置错误，提取了特权访问令牌。3月19日，他们对trivy-action的75个版本标签进行了强制推送（force-push），注入了恶意代码，并在Trivy v0.69.4发布版中植入了窃密程序。

### 2.2 第二步：窃取PyPI发布凭证

3月23日，LiteLLM的CI流水线照常运行，拉取了**已被投毒的Trivy版本**。Trivy中的窃密程序在LiteLLM的CI环境中执行，收集了所有能找到的秘密信息——其中包括维护者账号**krrishdholakia**的**PyPI发布密码（PYPI\_PUBLISH\_PASSWORD）**。

### 2.3 第三步：发布恶意版本

拿到PyPI凭证后，攻击者连续发布了两个被篡改的版本：

**v1.82.7**——手法相对隐蔽：

▪ 在`litellm/proxy/proxy_server.py`中插入了**仅12行**混淆代码

▪ 只有当开发者显式`import litellm.proxy`时才会触发

▪ 解码一个大型Base64载荷并通过子进程启动

**v1.82.8**——手法直接升级：

▪ 新增了一个`litellm_init.pth`文件

▪ .pth文件会在Python解释器每次启动时自动执行，甚至不需要import LiteLLM

▪ 这意味着只要LiteLLM存在于你的环境中，每一个Python进程都会触发恶意代码

### 2.4 恶意载荷：一个"生产级"窃密套件

这不是业余黑客的作品。载荷是一个**三阶段攻击体系**：

**第一阶段——凭证收割：**

* ▪ SSH私钥和配置文件
* ▪ AWS、GCP、Azure云凭证
* ▪ Kubernetes secrets和配置
* ▪`.env文件（包含所有API密钥）`
* ▪ Git凭证和Shell历史记录
* ▪ 加密货币钱包
* ▪ SSL私钥、CI/CD秘密信息、数据库密码

**第二阶段——Kubernetes横向移动：**

* ▪ 在 kube-system 命名空间中创建特权Alpine Pod
* ▪ 向集群中的每个节点部署特权DaemonSet
* ▪ 非伊朗节点安装CanisterWorm后门
* ▪ 伊朗节点直接执行集群擦除和强制重启（通过名为'kamikaze'的容器）

**第三阶段——持久化后门：**

▪ 在`~/.config/sysmon/sysmon.py`安装持久化后门

* ▪ 创建对应的systemd服务
* ▪ 定期轮询C2（命令与控制）服务器获取更新载荷
* ▪ 所有外泄数据使用AES-256+RSA-4096加密传输

图2：生产级窃密套件的三阶段攻击体系

···

## 三、一个Bug拯救了所有人

故事中最戏剧性的转折是——**恶意代码里有一个Bug**。

FutureSearch的**Callum McMahon**当时正在Cursor编辑器中测试一个MCP插件，该插件将LiteLLM作为传递依赖自动安装。当litellm 1.82.8安装后，`.pth`文件开始在每次Python解释器启动时执行，通过`subprocess.Popen`生成子进程。

但问题来了：子进程启动时，又会触发同一个`.pth`文件，子进程再生子进程……形成了一个**指数级增长的fork bomb（Fork炸弹）**。McMahon的机器瞬间RAM耗尽，彻底卡死。

正是这个异常引起了他的注意，进而发现了恶意代码。**如果攻击者修复了这个递归Bug，窃密程序将在完全静默的情况下运行**，可能数周甚至数月才会被发现。

从恶意版本发布到PyPI隔离，前后不到**3小时**。但考虑到LiteLLM每天340万次的下载量，以及通过DSPy等框架的传递依赖安装，受影响的开发者和系统数量依然难以估量。

图3：从漏洞利用到事件响应不到3小时

···

## 四、Andrej Karpathy发出警告

前OpenAI/Tesla AI总监**Andrej Karpathy**在X（Twitter）上发帖称这是"软件恐怖故事"：

> “Software horror: litellm PyPI supply chain attack. Simple `pip install litellm` was enough to exfiltrate SSH keys, AWS/GCP/Azure creds, Kubernetes configs, git credentials, env vars (all your API keys), shell history, crypto wallets, SSL private keys, CI/CD secrets, database passwords...
>
> —— Andrej Karpathy (@karpathy)

这条帖子引发了整个AI开发社区的震动。Hacker News、Reddit、各大安全博客的讨论爆炸式增长。人们突然意识到，**自己每天随手pip install的那些包，任何一个都可能在下一秒变成后门**。

···

## 五、开发者自查清单

如果你在3月24日前后安装或更新过LiteLLM，请立即执行以下操作：

### 5.1 检查是否受影响

```
# 检查已安装的litellm版本  
pip show litellm | grep Version  
  
# 检查是否存在恶意.pth文件  
find $(python -c "import site; print(site.getsitepackages()[0])") \  
  -name "litellm_init.pth"  
  
# 检查pip安装日志  
pip install --log /tmp/pip.log litellm 2>/dev/null  
grep "1.82.7\|1.82.8" /tmp/pip.log
```

### 5.2 确认受影响后的应急响应

1. **立即卸载并重装安全版本**

   `pip install litellm==1.82.6`
2. **轮换所有凭证**

   ——不是"可能泄露的"，是**所有**：

* ▪ SSH密钥（生成新密钥对，吊销旧密钥）

* ▪ AWS/GCP/Azure访问密钥和服务账号

* ▪ Kubernetes集群凭证和服务令牌

* ▪ 所有.env中的API密钥
* ▪ 数据库密码和Git令牌

3. **检查Kubernetes集群**

   查找异常的DaemonSet、特权Pod、可疑systemd服务
4. **审查系统**

   检查`~/.config/sysmon/`目录是否存在后门文件

···

## 六、深层反思：AI时代的供应链安全危机

### 6.1 "安全工具"本身成了攻击向量

这次事件最深刻的教训是：**你的安全工具可能是最大的安全隐患**。Trivy——一个安全扫描器——被攻陷后，成了攻击者进入其他项目的跳板。CI/CD流水线中运行的每一个第三方Action、每一个扫描器，都拥有对你构建环境的完全访问权限。

### 6.2 AI生态的特殊脆弱性

AI/LLM领域的供应链尤其脆弱，原因有三：

* ▪**依赖链深且杂**

  一个AI项目动辄几十上百个依赖，多数开发者根本不知道自己间接依赖了什么
* ▪**更新频率极高**

  LLM工具库几乎每天发版，开发者习惯性地`pip install --upgrade`
* ▪**凭证密度极高**

  AI应用天然需要大量API密钥——OpenAI、Anthropic、各种云服务，单个`.env`文件可能价值数万美元

### 6.3 防御纵深：没有银弹，只有层层设防

**锁定依赖版本（最重要）：**

使用`poetry.lock`或`uv.lock`的项目在这次事件中**完全未受影响**——锁文件将litellm钉在了安全版本上，无论PyPI上发生了什么。

```
# pyproject.toml - 精确锁定版本  
litellm = "==1.82.6"  # 不要用 >=1.79.2 这样的宽松约束
```

**哈希校验：**

```
# 使用uv生成带哈希的锁文件  
uv pip compile --generate-hashes requirements.in -o requirements.txt
```

**延迟安装策略：**

```
# uv支持时间过滤，只安装指定日期前发布的包  
uv pip install --exclude-newer 2026-03-23 litellm  
  
# pip v26+支持类似功能  
pip install --uploaded-prior-to 2026-03-23 litellm
```

**其他关键措施：**

* ▪  在CI中运行`pip-audit`检查已知漏洞
* ▪ 使用 Trusted Publishing（基于OIDC的发布机制），替代长期有效的PyPI API令牌
* ▪ 生成SBOM（软件物料清单），掌握依赖全貌
* ▪ 为CI/CD中的secrets设置最小权限和最短有效期

···

## 七、写在最后

TeamPCP的这次攻击展示了一种令人不安的新范式：**不攻击目标本身，而是攻击目标信任的工具，再借信任链逐级渗透**。从Trivy到LiteLLM的PyPI凭证，再到全球数百万开发者的本地环境——每一步都利用了"默认信任"的惯性。

作为开发者，我们需要接受一个不舒服的现实：**`pip install`不是无害的操作**。每一次安装和更新，都是在将一段你没有审查过的代码引入你的系统，赋予它访问你所有文件和网络的权限。

锁定依赖版本、使用哈希校验、延迟安装新版本、最小化CI/CD权限——这些措施单独来看都无法提供完美防护，但叠加起来，就构成了一道足够坚实的纵深防线。

在这个AI工具爆发式增长的时代，慢一步更新，可能就是最好的安全策略。

···

## 附录：.pth文件——Python中最危险的「隐形后门」机制

这次攻击的核心技术手段是一个名为`litellm_init.pth`的文件。很多开发者对`.pth`文件完全没有概念，但它可能是Python生态中**最被低估的安全风险**。

### 什么是.pth文件？

`.pth`文件（Path Configuration File）是Python的一种路径配置机制，存放在`site-packages`目录中。它的**设计初衷**很简单：每行写一个目录路径，Python启动时会自动将这些路径加入`sys.path`，从而让额外的库可以被找到。

但关键在于Python官方文档中的一句话：

以 import 开头（后跟空格或制表符）的行会被当作代码执行。

这意味着`.pth`文件不仅仅是一个路径清单——它可以**执行任意Python代码**。

### 执行时机：比你想象的更早、更频繁

`.pth`文件由Python的`site`模块在**解释器初始化阶段**自动加载。这意味着：

* ▪**不需要import任何模块**

  ——只要Python解释器启动，就会执行
* ▪**不需要显式运行任何脚本**

  ——`python -c "print(1)"`也会触发
* ▪**IDE语言服务器、pip自身、pytest**

  ——全部会触发
* ▪**每次启动都执行**

  ——不是一次性的，是每一次

用一个类比来说：如果普通的恶意代码是「你打开了一个文件才中招」，那么`.pth`恶意代码就是「你只要走进这个房间就中招」。

### litellm\_init.pth的具体作恶方式

攻击者在`litellm_init.pth`中写入了如下结构的代码：

```
import subprocess;subprocess.Popen(  
  ['python','-c',  
   __import__('base64').b64decode(  
     b'...(超长base64编码)...'  
   ).decode()]  
)
```

这一行代码做了以下事情：

1. **以import开头**

   ——满足`.pth`文件的执行条件
2. **调用subprocess.Popen**

   ——启动一个新的Python子进程
3. **双重Base64解码**

   ——解码出真正的窃密脚本
4. **子进程执行窃密逻辑**

   ——收集SSH密钥、云凭证、K8s配置等

### Fork Bomb：恶意代码的「自我毁灭」

这里有一个致命的Bug（也是救命的Bug）：

当`.pth`通过`subprocess.Popen`启动子进程时，**子进程也是一个Python解释器**。子进程启动时，`site`模块再次初始化，再次读取同一个`.pth`文件，再次执行`import`行，再次启动新的子进程……

```
Python启动 → 执行.pth → 启动子进程  
                         └→ Python启动 → 执行.pth → 启动子进程  
                                                     └→ Python启动 → ...  
                                                         （指数级增长，几秒内耗尽内存）
```

这就是经典的**Fork Bomb（Fork炸弹）**：进程数以指数级增长，在几秒内耗尽所有内存，导致系统完全卡死。

如果攻击者加一个简单的环境变量锁（比如检查`LITELLM_PTH_RUNNING`是否已设置），就能避免递归——窃密程序将在**完全静默**的情况下运行，极难被发现。

### 为什么.pth机制如此危险？

| 特征 | 普通Python恶意代码 | .pth恶意代码 |
| --- | --- | --- |
| 触发条件 | 需要import或运行 | **Python启动即执行** |
| 可见性 | 在.py文件中，可被审查 | **隐藏在site-packages深处** |
| 执行频率 | 按需执行 | **每次Python启动** |
| 影响范围 | 导入该模块的程序 | **环境中所有Python进程** |
| 防护难度 | 代码审查可发现 | **多数安全工具不检查** |

### 自查：检查你的环境

这个安全隐患并非首次被提出。早在CPython的Issue #113659中，就有开发者指出`.pth`文件的自动执行机制是一个严重的安全风险。但由于`setuptools`、`virtualenv`等大量工具依赖这一机制，短期内难以废除。

**作为开发者，你可以定期执行以下检查：**

```
# 查看site-packages中所有.pth文件  
ls -la $(python -c "import site; print(site.getsitepackages()[0])")/*.pth  
  
# 检查.pth文件中是否有可执行行（以import开头）  
grep -n "^import" \  
  $(python -c "import site; print(site.getsitepackages()[0])")/*.pth
```

定期检查你的`site-packages`目录中的`.pth`文件，特别是那些包含`import`语句的——它们可能就是下一个隐形后门。

···

本文基于FutureSearch、Snyk、ARMO、Sonatype、The Hacker News等多方安全研究报告整理。  
事件仍在持续调查中，建议关注LiteLLM官方GitHub和各大安全平台的后续通报。