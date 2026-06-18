> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020504_Hermes/020504_核心知识点/Hermes协作与记忆治理边界|Hermes协作与记忆治理边界]]
---
title: Hermes Agent + playwright-cli：Linux 服务器免费自动签到，零 Token 消耗
author: AI炼金社
date:
url: https://mp.weixin.qq.com/s?__biz=MzIwMzY3Njc2MA==&mid=2247484499&idx=1&sn=b92afc2206ed1be2e5ef25d08cf24f51&chksm=9709637301641f8f49105d2459fa1610816e2c4eb51e22cf12d033c00fdc11306e198a80766e&mpshare=1&scene=24&srcid=0420Hm6sIHKd5rARd2AGaT5I&sharer_shareinfo=a998d0f33f0765ff3fac268f8937172e&sharer_shareinfo_first=a998d0f33f0765ff3fac268f8937172e#rd
---

自动签到脚本跑不起来，因为浏览器要花钱？

Browserbase $0.05/分钟，Browser Use $0.02/请求。一个月自动签到 30 个站点，云浏览器成本几百块。更坑的是，每次截图、每次点击都要消耗 Token——8000 字 accessibility tree 直接塞进 context，跑几个任务 context 就爆了。

有没有免费方案？

有。**Hermes Agent + playwright-cli，Linux 服务器无头运行，零云浏览器成本，Token 消耗降低 93%。**

## 一、playwright-cli：专为 AI Agent 设计的浏览器 CLI

Playwright 官方 2026 年推出 `playwright-cli`，不是给人用的，是给 AI Agent 用的。

核心设计理念：**token-efficient**。

传统 Playwright MCP 把整个 accessibility tree 塞进 context，一个页面 8000+ 字。playwright-cli 只输出 element refs（`e1`, `e2`, `e3`），Agent 用 ref 就能操作，不需要理解整个 DOM。

| 方案 | Token 消耗 | 成本 |
| --- | --- | --- |
| Playwright MCP | 8000+ 字/页面 | 云浏览器 $0.05/min |
| playwright-cli | 500 字/页面 | 本地 Chromium 免费 |
| Browserbase + Hermes | 8000+ 字/页面 | $0.05/min |
| agent-browser + Hermes | 500 字/页面 | 本地 Chromium 免费 |

**93% 的 context savings**，这不是小优化，这是能不能跑起来的区别。

## 二、Hermes Agent：6 种浏览器后端，本地免费

Hermes Agent 不是单点工具，是 Agent 框架。浏览器后端支持 6 种：

1. **Browserbase**

   （云浏览器，付费）
2. **Browser Use**

   （云浏览器，付费）
3. **Firecrawl**

   （云浏览器 + scraping，付费）
4. **Camofox**

   （本地 Firefox 反指纹，免费）
5. **Local Chrome via CDP**

   （本地 Chrome，免费）
6. **agent-browser**

   （本地 Chromium，免费）

视频里用的是第 6 种：**agent-browser + playwright-cli**。

不需要云浏览器 API key，不需要付费订阅。只要服务器上有 Node.js 18+，安装 `agent-browser` 和 `playwright-cli`，就能跑。

## 三、实战：Linux 服务器无头运行自动签到

### 3.1 安装 playwright-cli

```
npm install -g @playwright/cli@latest
playwright-cli --help
```

Headless by default，不需要 `--headed` 参数。

### 3.2 安装 agent-browser

```
npm install -g agent-browser
```

agent-browser 是 Vercel Labs 开源项目，零配置，自动下载 Chromium。

### 3.3 安装 Hermes Agent

```
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/install.sh | bash
```

### 3.4 配置本地浏览器模式

编辑 `~/.hermes/.env`：

```
# 不设置任何云浏览器 API key
# Hermes 自动 fallback 到 agent-browser 本地模式
```

编辑 `~/.hermes/config.yaml`：

```
toolsets:
  - hermes-cli
  - browser
```

### 3.5 Linux 服务器无头运行

```
# 启动 Hermes CLI
hermes

# 在 Hermes 里执行
/browser_navigate https://example.com/login
/browser_snapshot
/browser_click @e5  # 点击登录按钮
/browser_type @e3 "username"
/browser_type @e4 "password"
/browser_click @e6  # 点击签到按钮
```

或者用 playwright-cli 直接跑：

```
playwright-cli open https://example.com/login
playwright-cli snapshot
playwright-cli type "username"
playwright-cli press Tab
playwright-cli type "password"
playwright-cli press Enter
playwright-cli click e21  # 签到按钮
```

## 四、playwright-cli 命令速查

| 命令 | 作用 |
| --- | --- |
| `playwright-cli open <url>` | 打开页面 |
| `playwright-cli snapshot` | 获取 element refs |
| `playwright-cli click <ref>` | 点击元素 |
| `playwright-cli type <text>` | 输入文本 |
| `playwright-cli fill <ref> <text>` | 填入输入框 |
| `playwright-cli press <key>` | 按键（Enter, Tab） |
| `playwright-cli screenshot` | 截图 |
| `playwright-cli state-save` | 保存登录状态 |
| `playwright-cli state-load` | 加载登录状态 |

保存登录状态后，下次签到不用重新登录——这是自动签发的关键。

## 五、避坑指南

### 5.1 Linux 服务器没有显示器

playwright-cli headless by default，不需要显示器。但某些站点检测 headless，需要反指纹工具：

* Camofox（Firefox 反指纹）：`git clone https://github.com/jo-inc/camofox-browser && npm install && npm start`
* 设置 `CAMOFOX_URL=http://localhost:9377` 到 `~/.hermes/.env`

### 5.2 登录状态持久化

默认浏览器 session 是临时的，关闭后 cookies 丢失。

playwright-cli 用 `--persistent` 参数：

```
playwright-cli open https://example.com --persistent
playwright-cli state-save login-state.json
```

下次直接加载：

```
playwright-cli state-load login-state.json
```

Hermes Agent 用 Camofox 时，配置 `browser.camofox.managed_persistence: true` 保持登录状态。

### 5.3 Token 消耗还是高怎么办

playwright-cli 已经比 MCP 省 93%，但长任务还是会爆 context。

解决方案：

* 用 `playwright-cli snapshot --filename=f.yml` 保存到文件
* Agent 不加载整个 snapshot，只读 refs
* 用 Hermes skill 把签到逻辑固化，不走 LLM reasoning

## 六、成本对比总结

| 方案 | 云浏览器成本 | Token 消耗 | 适用场景 |
| --- | --- | --- | --- |
| Browserbase + MCP | $50-200/月 | 高 | 复杂反爬站点 |
| Browser Use + MCP | $20-100/月 | 高 | 需要 residential proxy |
| agent-browser + CLI | $0 | 低 | 常规自动签到 |
| Camofox + Hermes | $0 | 中 | 反指纹需求 |

免费方案适合大多数自动签到场景。只有需要 residential proxy 或高级反爬时才考虑付费云浏览器。

## 七、小结

* **playwright-cli**

  ：Playwright 官方 CLI，token-efficient，专为 AI Agent 设计
* **Hermes Agent**

  ：6 种浏览器后端，本地模式免费
* **agent-browser**

  ：Vercel Labs 开源，零配置本地 Chromium
* **Linux 无头运行**

  ：headless by default，不需要显示器
* **登录状态持久化**

  ：`--persistent` + `state-save`，签到不用重新登录

免费实现浏览器全自动化，不是「理论上可行」，是现在就能跑起来。

## 相关链接

* playwright-cli 文档：https://playwright.dev/docs/getting-started-cli
* Hermes Agent 浏览器自动化：https://hermes-agent.nousresearch.com/docs/user-guide/features/browser
* agent-browser GitHub：https://github.com/vercel-labs/agent-browser
* Camofox 反指纹浏览器：https://github.com/jo-inc/camofox-browser
* Hermes Agent 安装：https://github.com/NousResearch/hermes-agent