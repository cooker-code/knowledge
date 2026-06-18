> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020202_MCP/020202_核心知识点/MCP生产接入与治理边界|MCP生产接入与治理边界]]
---
title: Aiops探索：基于 n8n + Ansible MCP Server 的智能运维实践
author: 阿铭linux
date:
url: https://mp.weixin.qq.com/s?__biz=MjM5MTk1ODE0MA==&mid=2648441963&idx=1&sn=99a84955df6bb88bd1dd2f8c18a0dcaa&chksm=bfb248438e49940692c20ee4c1f9ed438fffd8932cfa009c06eda229d7a440d3323c3a5030f5&mpshare=1&scene=24&srcid=12092Nd0B0KKBXGvxzxHvA6b&sharer_shareinfo=83ec1362a66fe64091872404d08f6230&sharer_shareinfo_first=83ec1362a66fe64091872404d08f6230#rd
---

↑↑↑ 点击关注，分享IT技术|职场晋升技巧|AI工具

研究Aiops有一段时间了，目前手里有不少可落地的方案了，接下来会把这些方案全部整理到我的[大模型课程](https://mp.weixin.qq.com/s?__biz=MjM5MTk1ODE0MA==&mid=2648440799&idx=1&sn=1cbb1fe8d8a2f5f7719c7f960feba925&scene=21#wechat_redirect)里。同时，欢迎大家把你遇到的场景在评论区留言。我会在能力范围内给你提供思路和建议。

上个月给大家整理过一篇[Dify+Ansible MCP的案例](https://mp.weixin.qq.com/s?__biz=MjM5MTk1ODE0MA==&mid=2648441752&idx=1&sn=f4720b7a006af698882e87cb657884ca&scene=21#wechat_redirect)，而今天这个案例是基于n8n的，有同学可能会有疑问，为什么要用n8n呢？因为n8n很强，它做的事情更多，比如将多个智能体串起来，做更复杂的场景！

首先说下n8n调用远程MCP服务器的实现思路。早期n8n的版本会比较麻烦，但新版本已经原生支持，只需要增加一个AI Agent节点，并且在Tools那里增加一个MCP Client即可，而MCP Client里的设置跟在Dify中没有太大差异。

下面我们来做一个Ansible的MCP Server，然后在n8n里去调用。步骤如下：

步骤 1: 准备Ansible环境

这里假设已经有了一个可用的Ansible控制节点，并且其它机器已经可以通过Ansible管理。

#### 步骤 2: 部署 Ansible MCP Server

这里需要直接部署到Ansible控制机上。

1、克隆代码

```
git clone https://github.com/aminglinux/ansible-mcp.gitcd ansible-mcp
```

2、安装依赖库

```
pip3 install -r requirements.txt
```

3、启动服务

```
uvicorn main:app --reload --host 0.0.0.0 --port 8080
```

步骤 3: 在n8n中配置 MCP 工具

在n8n画布中增加一个AI Agent节点，大模型选择DeepSeek，Tool这里添加MCP Client

双击MCP Client，Endpoint填写Ansible MCP的地址：http://<host>:8080/sse（这里host地址就是你部署Ansible MCP服务的IP地址）。

Server Transport选择：Server Sent Events，这个其实就是sse模式。

点击右上角的“Execute step”，可以看到Tool name里有可选的工具列表

选择list\_hosts，然后测试

返回画布，双击AI Agent节点，设置提示词：

```
你是一个Linux运维专家，擅长Ansbile的各种操作，尤其是擅长撰写Ansible的playbook
你有诸多ansbile相关的工具，其中工具的功能如下：1. list_inventory : 列出inventory2. list_hosts: 列出所有主机3. validate_playbook : 验证playbook是否有错误4. ping_hosts : 检查主机是否存活5. run_ad_hoc : 临时运行ansible任务6. generate_playbook: 生成playbook文件7. run_playbook: 运行指定的playbook
另外请遵循以下规则：1. 默认你会调用./inventory.ini文件，如果有指定可以使用指定inventory文件，没有就用默认的。2. 用户需求如果比较复杂，请拆解任务，并使用合适的工具来落地需求，比如用户给一个主机名，你需要去查inventory，然后再去调用别的工具3. 当用户需要生成playbook时，请你自动生成一个playbook文本，然后赋值给data参数，并传递给generate_playbook工具4. 当用户执行playbook时，要先检查用户给的playbook文件名是否在playbooks/目录里存在，如果存在直接调用，如果不存在则需要自动生成5. 在执行playbook之前，请先确认该playbook是否有问题6. 调用工具所需要的参数，需要你根据User Message的信息并结合调用其它工具来自动获取到
```

1. 列出所有主机

2. 查看某主机的系统负载

3. 执行命令

4. 给指定机器安装redis

此时报错了，据说这是官方的一个bug，到目前为止还未修复。

看来，当前阶段想要用n8n愉快地玩耍还是有点阻碍，只是希望n8n官方尽快修复此问题。

---

最后介绍下我的大模型课：[我的运维大模型课上线了](https://mp.weixin.qq.com/s?__biz=MjM5MTk1ODE0MA==&mid=2648440799&idx=1&sn=1cbb1fe8d8a2f5f7719c7f960feba925&scene=21#wechat_redirect)，目前还在预售期，有很大优惠。AI越来越成熟了，大模型技术需求量也越来越多了，至少我觉得这个方向要比传统的后端开发、前端开发、测试、运维等方向的机会更大，而且一点都不卷！

扫码咨询优惠（粉丝优惠力度大）

**··············  END  ··············**

哈喽，我是阿铭，《跟阿铭学Linux》作者，曾就职于腾讯，有着18年的IT从业经验，现全职做IT类职业培训：运维、k8s、大模型。日常分享运维、AI、大模型相关技术以及职场相关，欢迎围观。