> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020507_Codex/020507_核心知识点/Codex工程使用与沙箱边界|Codex工程使用与沙箱边界]]
---
title: 别再折腾 cc-switch 了，Codex 可以这样配中转站，更优解
author: 老秉AI
date: 老秉AI老秉AI
url: https://mp.weixin.qq.com/s?__biz=MzcwODE3ODk3OA==&mid=2247484125&idx=1&sn=f5584f20ac96db0de3a1aaef123fe31d&chksm=f44e75eb2398583de46f6d7cf09577baed0d9b87584340b320c4a1c267645b901ae87533fb77&mpshare=1&scene=24&srcid=0526VMJ7hjoU03UosFgdTyQL&sharer_shareinfo=70c442d6edb2fd086d1b344aee4c07c1&sharer_shareinfo_first=70c442d6edb2fd086d1b344aee4c07c1#rd
---

你明明有 Codex 账号，也登录好了，但又嫌充值会员贵，会员不想开。最后只能买中转站，结果一折腾，账号掉线、插件不能用。

我之前试过 `cc-switch` 这类方案，能用，但不舒服。

账号容易掉线，插件也不稳定，还要配合别的插件才能把功能补回来。对我来说，这就不是“省事”，而是给自己多装了几个开关。

后来我换了一个办法：保留 Codex 原账号登录，同时在 `config.toml` 里配置中转站。

目前这个方案对我最顺手。

【1】先说适合谁

如果你已经登录了 Codex 账号，只是没法充值、开不了会员，又想继续正常用 Codex 插件，那这篇就适合你。

核心思路很简单：

账号还是你的 Codex 账号，插件也继续走原来的登录状态；模型请求则通过你配置的中转站走。

这样就不用频繁切账号，也不用为了插件再额外折腾一套东西。

当然，前提是你已经有一个可用的中转站，并且对方给了你 `base_url`、模型名和 token。

【2】找到 config.toml

先确认你已经登录 Codex。

然后找到 Codex 的配置文件 `config.toml`。

以 Windows 为例，路径一般是：

```
C:\Users\你的用户名\.codex\config.toml
```

比如我的用户目录类似这样：

```
C:\Users\86134\.codex
```

Mac 用户可以先看：

```
~/.codex/config.toml
```

如果你是特殊安装，也可能在类似下面的位置：

```
/usr/local/bin/codex/.codex
```

找不到也别慌，直接在电脑里搜 `config.toml`，重点看 `.codex` 目录下面那个。

改之前建议先复制一份备份。配置文件这种东西，改错了不一定大事，但回滚会省很多时间。

【3】把中转站配置放到最上面

打开 `config.toml`，把下面这段放到文件最上方。

注意，是最上方。

```
model_provider="beefapi_codex"
model="gpt-5.5"

[model_providers.beefapi_codex]
name="beefapi_codex"
base_url="https://beefapi.com/v1"
wire_api="responses"
requires_openai_auth=true
experimental_bearer_token="..."
```

这里有几个地方要换成你自己的：

`model` 换成中转站支持的模型名。

`base_url` 换成中转站给你的地址。

`experimental_bearer_token` 里的 `...` 换成你自己的 token。

不要把 token 截图发群里，也不要写进公开文章或仓库里。这个东西就像钥匙，丢了别人就可能拿去用你的额度。

【4】最容易踩的坑：位置不对就不生效

这个坑我踩了好几次。

一开始我把配置随手塞到文件后面，重启 Codex 以后怎么都不对。看起来像是配置写了，但实际没有生效。

后来我才发现，关键是这几行要放在 `config.toml` 的最上面。

放对位置以后，再重启 Codex，就能看到中转站配置生效了。

这也是我觉得这个方案比切账号舒服的地方：你不用反复登录、掉线、修插件，只要让 Codex 继续认你的账号，再把模型供应商指到中转站。

【5】最后测试一下

配置完以后，关掉 Codex，重新打开。

然后新建一个会话，随便问一个问题，看是否能正常返回。

如果没反应，优先检查这几个地方：

1. 配置是不是放在文件最上方。
2. `base_url`

   有没有写错。
3. token 有没有多复制空格。
4. 模型名是不是中转站真实支持的。
5. Codex 有没有彻底重启。

这套配置不是最花哨的方案，但胜在稳定。

我的判断是：如果你只是想正常用 Codex，又不想每天跟登录状态、插件状态较劲，那就别把事情搞复杂。

账号归账号，中转归中转。

能稳定写代码，才是最重要的。