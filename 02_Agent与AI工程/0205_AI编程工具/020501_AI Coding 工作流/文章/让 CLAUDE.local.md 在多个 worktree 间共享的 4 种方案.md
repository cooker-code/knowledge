---
title: 让 CLAUDE.local.md 在多个 worktree 间共享的 4 种方案
author: AI码人士
date: AICoderAICoder
url: https://mp.weixin.qq.com/s?__biz=MzAwNTg3MDM1Nw==&mid=2650254228&idx=2&sn=eacc1b6406544db78c21b73a6e5db3cf&chksm=82406336048f1fb65e479e5cb2ca754f317b9e7440e95c3525a1369547f157a1d5eaf345ae23&mpshare=1&scene=24&srcid=0604NzWxpbrbAdD0nI2AiGeF&sharer_shareinfo=cd47fc1078e9e7bff81edacfdfb903b6&sharer_shareinfo_first=cd47fc1078e9e7bff81edacfdfb903b6#rd
---

先说背景，让概念清楚：

**问题本质**：`CLAUDE.local.md`是 gitignored 的物理文件，存在于具体目录里。git worktree 是**独立的工作目录**（虽然共享 `.git`历史），所以你在 `main`worktree 里建的 `CLAUDE.local.md`，切到 `feature-x`worktree 就**根本不存在**——它压根没被复制过去。

Anthropic 官方文档对此有明确说法：

> "If you work across multiple git worktrees of the same repository, a gitignored `CLAUDE.local.md`only exists in the worktree where you created it. To share personal instructions across worktrees, import a file from your home directory instead."

下面是从"官方推荐"到"工程派"4 种解法，按推荐度排序。

---

## 方案一：官方推荐 —— @import 语法引用 home 目录文件 ⭐⭐⭐⭐⭐

这是 Anthropic 官方明确推荐的做法，也是**最简单、最稳健**的方案。

### 思路

把"真正的内容"放在 home 目录（一个**所有 worktree 都能访问**的位置），每个 worktree 的 `CLAUDE.local.md`只是一个**指针文件**，用 `@`语法 import 那个真身。

### 步骤

**1. 在 home 目录建一个"项目专属"的个人指令文件：**

```
mkdir -p ~/.claude/per-project/  
nvim ~/.claude/per-project/myapp-instructions.md
```

文件内容写你真正的本地配置：

```
# myapp 项目的个人指令  
  
## 调试标记规则  
  
- 所有调试标记必须以 `[DEBUG-xxx]` 开头  
- 提交前主动跑 `grep -rn "DEBUG-" src/`  
  
## 我的工作习惯  
  
- 先看 plan 再 approve  
- 强制 pnpm,看到 npm 必须改  
  
## ……(其他所有 local 配置)
```

**2. 在每个 worktree 的 `CLAUDE.local.md`里，只写一行 import：**

```
# 项目个人指令(实际内容在 home 目录,所有 worktree 共享)  
  
@~/.claude/per-project/myapp-instructions.md
```

**3. 新建 worktree 时，复制这个"指针文件"即可：**

```
git worktree add ../myapp-feature-x feature-x  
cp myapp/CLAUDE.local.md ../myapp-feature-x/CLAUDE.local.md
```

### 优点

* ✅ **官方支持**，未来兼容性最好
* ✅ 改一处，所有 worktree 立刻生效
* ✅ 不依赖 OS 特性（Windows、macOS、Linux 都能用）
* ✅ Claude 第一次遇到外部 import 会弹"是否信任"对话框，安全感强

### 缺点

* ⚠️ 每个新 worktree 还是需要**手动建一个 `CLAUDE.local.md`指针文件**（虽然只有 1 行）
* ⚠️ 内容放 home 目录，离项目本身有点远

### 适合谁

**90% 的人都该用这个。**简单、官方、跨平台。

---

## 方案二：进阶 —— "公共部分 + 私有部分"分层 ⭐⭐⭐⭐

方案一的升级版。如果你有些配置**应该所有 worktree 共享**（比如个人偏好、安全协议），但又有些配置**应该 worktree 独有**（比如 `feature-x`分支的实验豁免、当前任务的 scratchpad），就用分层。

### 思路

```
~/.claude/per-project/myapp-shared.md  ← 跨 worktree 共享的部分  
myapp/CLAUDE.local.md                  ← main worktree 专属(import shared + 追加)  
myapp-feature-x/CLAUDE.local.md        ← feature-x worktree 专属(import shared + 追加不同内容)
```

### 步骤

**1. 把"共享部分"放 home 目录：**

```
<!-- ~/.claude/per-project/myapp-shared.md -->  
  
## 敏感信息处理协议  
  
- 永远不要读 .env、~/.ssh/、\*.pem  
- ……  
  
## 我的工作习惯  
  
- 先看 plan 再 approve  
- ……
```

**2. 每个 worktree 的 `CLAUDE.local.md`自己写不一样的内容：**

```
<!-- myapp/CLAUDE.local.md (main 分支) -->  
  
@~/.claude/per-project/myapp-shared.md  
  
## 本 worktree 专属  
  
- 这是 main worktree,优先处理生产 hotfix  
- 当前没有进行中的长任务
```

```
<!-- myapp-feature-x/CLAUDE.local.md (feature-x 分支) -->  
  
@~/.claude/per-project/myapp-shared.md  
  
## 本 worktree 专属  
  
- 这是 feature-x 实验分支  
- 允许临时使用 any 类型  
- 当前任务进度: @./.claude/scratchpad/current.md
```

### 优点

* ✅ 共享部分改一次到处生效
* ✅ 每个 worktree 又能有自己的"私房配置"
* ✅ 结构清晰，符合"不同关注点分离"

### 适合谁

**长期同时维护多个 worktree、且各 worktree 用途明显不同的人**（比如：main 用于 hotfix，feature-x 是实验，release-1.2 是版本维护）。

---

## 方案三：自动化 —— 包装 git worktree add 命令 ⭐⭐⭐

如果你嫌每次手动 `cp`麻烦，写个 shell 函数自动化。

### 步骤

在 `~/.zshrc`或 `~/.bashrc`加：

```
# 创建 worktree 时,自动生成指针式 CLAUDE.local.md  
gwt() {  
local path="$1"  
local branch="$2"  
  
# 创建 worktree  
  git worktree add "$path""$branch" || return 1  
  
# 自动生成 CLAUDE.local.md(指针文件)  
local project_name=$(basename "$(git rev-parse --show-toplevel)")  
local shared_file="$HOME/.claude/per-project/${project_name}-shared.md"  
  
if [ -f "$shared_file" ]; then  
    cat > "$path/CLAUDE.local.md" <<EOF  
# 本 worktree 的本地配置  
# 共享部分自动 import 自 home 目录  
  
@$shared_file  
  
## 本 worktree 专属(按需添加)  
- worktree 路径: $path  
- 创建时间: $(date '+%Y-%m-%d')  
- 分支: $branch  
EOF  
    echo"✓ 已自动生成 $path/CLAUDE.local.md"  
else  
    echo"⚠ 没找到共享配置文件 $shared_file,跳过"  
fi  
}
```

使用：

```
gwt ../myapp-feature-x feature/x  # 自动建 worktree + 自动生成指针文件
```

### 优点

* ✅ 一行命令搞定，零摩擦
* ✅ 可以在 shell 函数里加更多自动化（比如自动 `pnpm install`、自动 `cp .env`）

### 缺点

* ⚠️ shell 函数需要自己维护
* ⚠️ 团队成员之间不共享（每人各自维护自己的）

### 适合谁

**重度使用 worktree、每周创建多个**的人。一次写好，长期受益。

---

## 方案四：硬核派 —— Symlink 软链接 ⭐⭐

不推荐为主方案，但在某些场景有用，列出来让你心里有数。

### 思路

直接在每个 worktree 里把 `CLAUDE.local.md`做成符号链接，指向 home 目录的"真身"：

```
# 在每个 worktree 里  
ln -s ~/.claude/per-project/myapp-shared.md CLAUDE.local.md
```

### 优点

* ✅ 完全零延迟同步，改一处所有 worktree 立刻看到

### 缺点

* ❌ **Windows 默认不支持**symlink（需要开发者模式或管理员权限）
* ❌ 某些工具（rsync、tar、IDE 索引器）对 symlink 处理不一致
* ❌ 容易被误删——`rm CLAUDE.local.md`在某些 shell 里会顺着 link 删源文件
* ❌ `.gitignore`对 symlink 的行为偶尔会让人困惑

### 适合谁

**只有 Linux/macOS、追求极致简洁、且很懂 symlink 行为**的人。比方案一**不**推荐——方案一用 `@`import 能拿到几乎一样的效果，但更安全。

---

## 一个特别的坑：Claude Code 的 `--worktree`旗标自带"半同步"机制

补充一个容易踩的细节：

如果你用 `claude --worktree`命令让 Claude Code 自己创建 worktree（而不是手动 `git worktree add`），它会**部分同步**`.claude/`目录——但有 bug：

* ✅ 会同步 `settings.local.json`
* ❌ **不会**同步 `.claude/skills/`、`.claude/agents/`、`.claude/rules/`（这是已知 issue）
* ❌ **不会**同步 `CLAUDE.local.md`

所以即便用了 `--worktree`，你仍然需要上面的方案之一来同步 `CLAUDE.local.md`。

---

## 给你的最终推荐路径

按你的使用强度，照下面这张表选：

| 使用强度 | 推荐方案 | 投入时间 |
| --- | --- | --- |
| 偶尔用 worktree（一个月几次） | **方案一** （官方 import） | 5 分钟 |
| 经常用，多 worktree 长期并存 | **方案二** （共享+私有分层） | 15 分钟 |
| 重度使用，每周建多个 | **方案二 + 方案三** （分层 + shell 函数） | 30 分钟 |
| 你是 Linux 老哥，懂 symlink | **方案二** （仍然推荐它，别用 symlink） | 15 分钟 |

**我个人用的就是方案二 + 方案三**：home 目录放共享部分，shell 函数自动生成 worktree 时的指针文件。设置一次，半年没再操心过。

---

## 一句话总结

> **永远把 `CLAUDE.local.md`当作"指针文件"，让真正的内容住在 `~/.claude/`里。**这样 worktree 之间不需要同步任何东西——它们都指向同一份真身。