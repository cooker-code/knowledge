---
title: mise：告别版本管理地狱，一个工具搞定所有语言环境
author: 万事屋的银时
date: 
url: https://mp.weixin.qq.com/s?__biz=MzUzMzY5MTg5NQ==&mid=2247484048&idx=1&sn=7a7989349a7a1b0544c5240b594eb0a4&chksm=fbd4e1f1a397f079aca4f6921a9465080af56b841a0c5fb8cfcc5be881565f376964b9763ef7&mpshare=1&scene=24&srcid=0414zE4h0cmEUkJLehlK0XTW&sharer_shareinfo=393048169dac65b5eb424e21d59ab4e3&sharer_shareinfo_first=393048169dac65b5eb424e21d59ab4e3#rd
---

> 你的电脑里有几个版本管理工具？nvm 管 Node.js、pyenv 管 Python、rbenv 管 Ruby……每个工具有自己的安装方式、配置文件和命令风格。切换项目时忘记切换版本，报错了还不知道为什么。这个痛苦，该结束了。

---

## 🍳 什么是 mise？

**mise**（发音"meez"，法语"摆好"之意，出自料理术语 *mise en place*——"一切就绪"）是一款用 Rust 编写的多语言开发工具版本管理器。

它能替代：

* `nvm` / `fnm`（Node.js 版本管理）
* `pyenv`（Python 版本管理）
* `rbenv` / `rvm`（Ruby 版本管理）
* `asdf`（通用版本管理）
* `direnv`（目录级环境变量）

**一个工具，统管一切。**

---

## ⚡ 为什么选 mise 而不是 asdf？

mise 的设计灵感来自 asdf，且完全兼容 `.tool-versions` 文件格式，但它有几项关键优势：

| 对比项 | asdf | mise |
| --- | --- | --- |
| 实现语言 | Shell 脚本 | Rust |
| 启动速度 | 较慢 | 极快 |
| PATH 注入方式 | Shim 文件 | 直接修改 PATH |
| 环境变量管理 | ❌ 不支持 | ✅ 内置支持 |
| 任务运行器 | ❌ 不支持 | ✅ 内置支持 |
| 配置格式 | `.tool-versions` | `.mise.toml` （更丰富） |

> mise 直接修改 `PATH`，而不是依赖 shim 脚本转发。这意味着 `which node` 返回的是真实二进制路径，且工具调用无额外开销。

---

## 🚀 安装与初始化

### 安装

```
1. curl https://mise.run | sh
```

安装完成后，二进制文件位于 `~/.local/bin/mise`。

### 激活（写入 shell 配置）

**Bash：**

```
1. echo 'eval "$(~/.local/bin/mise activate bash)"'>>~/.bashrc
2. source ~/.bashrc
```

**Zsh：**

```
1. echo 'eval "$(~/.local/bin/mise activate zsh)"'>>~/.zshrc
2. source ~/.zshrc
```

**Fish：**

```
1. echo '~/.local/bin/mise activate fish | source'>>~/.config/fish/config.fish
```

### 验证安装

```
1. mise --version
2. # mise 2026.x.x

4. mise doctor
5. # 检查环境配置是否正常
```

---

## 🛠️ 核心用法

### 安装运行时

```
1. # Node.js
2. mise use --global node@lts

4. # Python
5. mise use --global python@3.12

7. # Go
8. mise use --global go@latest

10. # Rust
11. mise use --global rust@latest

13. # Java (Temurin 发行版)
14. mise use --global java@temurin-21
```

`--global` 表示全局生效，不加则只对当前目录有效。

### 查看已安装版本

```
1. mise list           # 列出所有已安装工具
2. mise ls --current   # 只显示当前生效的版本
3. mise ls-remote node # 查询 Node.js 所有可用版本
```

### 版本切换

```
1. mise use node@20          # 当前项目使用 Node 20
2. mise use --global node@22 # 全局切换到 Node 22
3. mise shell python@3.11# 仅当前 shell 会话生效
```

---

## 📁 项目级版本锁定：.mise.toml

这是 mise 最核心的功能。在项目根目录执行：

```
1. mise use node@20 python@3.11
```

mise 会自动生成（或更新） `.mise.toml`：

```
1. [tools]
2. node ="20"
3. python ="3.11"
```

**把这个文件提交到 Git。** 团队成员克隆项目后只需执行：

```
1. mise install
```

mise 会自动安装文件中声明的所有工具版本，再也不会出现"在我机器上能跑"的问题。

> mise 同样兼容 asdf 的 `.tool-versions` 文件，以及 `.node-version`、 `.python-version` 等传统格式，迁移零成本。

---

## 🌍 环境变量管理

mise 内置了 direnv 的核心功能——按目录自动注入环境变量。

在 `.mise.toml` 中添加 `[env]` 节：

```
1. [tools]
2. node ="20"
3. python ="3.11"

5. [env]
6. DATABASE_URL ="postgresql://localhost/myapp_dev"
7. API_BASE_URL ="https://api.dev.example.com"
8. NODE_ENV ="development"

10. # 也可以加载外部 .env 文件
11. _.file =[".env",".env.local"]
```

进入目录时这些变量自动生效，离开目录后自动清除。首次使用需执行 `mise trust` 授权该配置文件（防止执行不受信任的配置）。

---

## ⚙️ 内置任务运行器

mise 还可以替代 `Makefile` 和 `npm scripts`，在 `.mise.toml` 中定义任务：

```
1. [tasks.dev]
2. description ="启动开发服务器"
3. run ="node server.js"

5. [tasks.test]
6. description ="运行测试"
7. run ="pytest tests/ -v"

9. [tasks.build]
10. description ="构建项目"
11. run ="cargo build --release"

13. [tasks.lint]
14. description ="代码检查"
15. run =["eslint src/","prettier --check src/"]
```

执行任务：

```
1. mise run dev
2. mise run test
3. mise tasks list   # 列出所有可用任务
```

---

## 📊 配置层级

mise 支持多层配置，从宽泛到具体逐级覆盖：

```
1. ~/.config/mise/config.toml    # 全局默认配置
2. ~/work/mise.toml              # 工作目录通用配置
3. ~/work/project/mise.toml      # 项目专属配置（最高优先级）
```

每一层可以继承或覆盖上一层的设置，无需重复声明。

---

## 🔄 常用命令速查

```
1. # 安装 & 更新
2. mise install                  # 安装 .mise.toml 中声明的所有工具
3. mise upgrade                  # 升级所有工具到最新兼容版本
4. mise self-update              # 升级 mise 本身

6. # 查询
7. mise list                     # 查看已安装工具
8. mise ls-remote node           # 查询可用版本列表
9. mise doctor                   # 检查环境健康状态

11. # 版本管理
12. mise use node@20              # 项目级安装并使用
13. mise use --global node@lts    # 全局安装并使用
14. mise shell node@18            # 临时切换（当前 shell）
15. mise uninstall node@18        # 卸载指定版本

17. # 任务
18. mise run <task># 执行任务
19. mise tasks list               # 列出所有任务
```

---

## 💡 WSL2 使用小贴士

在 WSL2 Ubuntu 环境下，有几点需要注意：

1. **安装依赖**：Python 编译安装前建议先执行 `sudo apt install-y build-essential libssl-dev zlib1g-dev`，避免构建失败。
2. **权限问题**：如果遇到 `~/.local/share/mise` 权限报错，执行：

   ```

   ```

1. `sudo chown -R $USER ~/.local/share/mise`

3. **mise doctor 是你的好朋友**：遇到任何异常先执行它，通常能直接告诉你哪里出了问题以及如何修复。

---

## 📝 小结

mise 并不是在重新发明轮子，而是把散落在各处的轮子拼成了一辆完整的车：

* **版本管理** → 替代 nvm / pyenv / asdf
* **环境变量** → 替代 direnv
* **任务运行** → 替代 Makefile / npm scripts

对于在多个项目间切换的开发者，尤其是团队协作场景，mise 能显著降低"环境不一致"带来的沟通成本。值得一试。

---

## 参考资料

1. mise 官方文档：https://mise.jdx.dev/
2. mise GitHub 仓库（@jdx）：https://github.com/jdx/mise
3. mise Dev Tools 文档：https://mise.jdx.dev/dev-tools/
4. Towards AI — Introducing mise: A Fast and Dev-Friendly Version Manager：https://towardsai.net/p/l/introducing-mise-a-fast-and-dev-friendly-version-manager-for-your-toolchain
5. OneUptime Blog — How to Use mise for Tool Version Management（2026-01）：https://oneuptime.com/blog/post/2026-01-25-mise-tool-version-management/view
6. DEV Community — Mise: The Ultimate Dev Tool Manager for Seamless Workflows：https://dev.to/jdxlabs/mise-the-ultimate-dev-tool-manager-for-seamless-workflows-2lm0
7. HARIL Blog — Managing Development Tool Versions with mise（2025-12）：https://haril.dev/en/blog/2024/06/27/Easy-devtools-version-management-mise
8. Renovate Docs — Automated Dependency Updates for mise-en-place：https://docs.renovatebot.com/modules/manager/mise/