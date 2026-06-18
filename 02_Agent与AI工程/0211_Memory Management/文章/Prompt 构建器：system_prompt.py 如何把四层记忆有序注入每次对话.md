---
title: Prompt 构建器：system_prompt.py 如何把四层记忆有序注入每次对话
author: James的成长日记
date: JameszyhJameszyh
url: https://mp.weixin.qq.com/s?__biz=MzE5ODQyNDY4OQ==&mid=2247487222&idx=1&sn=389bc60d269e5d6476904817880ba0b5&chksm=97ceada0f375a1eac6b7ae1033a74cfb6698f6b1664d24aa49c59714a43f3a5ec3aa3eaee2f9&mpshare=1&scene=24&srcid=0605iyJqfQm12X4T2xevI4gV&sharer_shareinfo=2c7b1977a35aab8ab7b58e6ed77baa48&sharer_shareinfo_first=2c7b1977a35aab8ab7b58e6ed77baa48#rd
---

大家好，我是James。

上一篇拆解了跨会话检索，半年前的对话也能找回来。但找回来之后，这些记忆怎么塞进 system prompt？

说一个真实的坑：很多同学自己搭 Agent 时，把所有信息拼成一个大字符串，每轮对话全量重建 system prompt。结果 Claude 说「这次对话花了 3 倍的 token」。更惨的是：模型的前缀缓存（prefix cache）被彻底打散，你支付了缓存价格却一分没命中。

Hermes 把这个问题解决得很彻底：**三层架构 + 一次构建 + 末尾可变**。这篇文章带你完整读懂 `system_prompt.py` 的每一行设计决策。

---

## 01 三层架构：Stable / Context / Volatile，为什么这样分？

大多数人搭 Agent 时，system prompt 就是一个字符串，里面塞了身份、工具说明、用户记忆、当前时间……全部混在一起。每轮对话一旦有任何一块变了，整个字符串就不一样了，前缀缓存命中率 0%。

Hermes 的做法是：**按变化频率把 system prompt 分成三层**，稳定内容在前，易变内容在后。

| 层级 | 内容 | 变化频率 | 对缓存的意义 |
| --- | --- | --- | --- |
| **Stable** | 身份/SOUL.md、工具指引、Skills 索引、平台提示 | 整个进程生命周期不变 | 完全命中前缀缓存 |
| **Context** | AGENTS.md / .cursorrules、caller system\_message | 项目切换时才变 | 工作目录不变则稳定 |
| **Volatile** | MEMORY.md 快照、USER.md、外部记忆、时间戳 | 会话开始时拍快照，之后不变 | 放末尾最大化前缀命中 |

**关键洞察**：LLM 的前缀缓存只缓存"从头开始不变的部分"。把最稳定的内容放最前面，最易变的放最后面，是最大化缓存命中率的唯一正确姿势。

---

## 02 Stable 层：身份 + 工具指引 + Skills 索引的填充顺序

Stable 层是整个 system prompt 的骨架，填充顺序很讲究：

```
# agent/system_prompt.py - Stable 层核心填充逻辑（精简版）  
  
# 1. 身份：优先 SOUL.md，不存在则用硬编码默认身份  
_soul_content = _r.load_soul_md()  
stable_parts.append(_soul_content if _soul_content else DEFAULT_AGENT_IDENTITY)  
  
# 2. 工具指引：只注入当前会话拥有的工具的使用说明  
if "memory"       in agent.valid_tool_names: stable_parts.append(MEMORY_GUIDANCE)  
if "skill_manage" in agent.valid_tool_names: stable_parts.append(SKILLS_GUIDANCE)  
if "kanban_show"  in agent.valid_tool_names: stable_parts.append(KANBAN_GUIDANCE)  
  
# 3. 模型族专项指引：Gemini 用绝对路径，GPT 别停下来  
_m = (agent.model or "").lower()  
if "gemini" in _m: stable_parts.append(GOOGLE_MODEL_OPERATIONAL_GUIDANCE)  
if "gpt"    in _m: stable_parts.append(OPENAI_MODEL_EXECUTION_GUIDANCE)  
  
# 4. Skills 索引（内存 LRU + 磁盘快照双层缓存）  
skills_prompt = _r.build_skills_system_prompt(available_tools=agent.valid_tool_names)  
if skills_prompt: stable_parts.append(skills_prompt)  
  
# 5. 环境提示 + 平台提示（WSL 路径转换、wecom 媒体发送格式等）  
if env_hints := _r.build_environment_hints(): stable_parts.append(env_hints)  
if platform_key in PLATFORM_HINTS: stable_parts.append(PLATFORM_HINTS[platform_key])
```

几个值得深挖的设计细节：

**工具条件注入**：模型只需要知道它**拥有**的工具的用法。如果这次启动没有 `memory` 工具，`MEMORY_GUIDANCE` 就不注入，不浪费 token，也不让模型产生「我有这个工具但找不到」的幻觉。

**模型族分支**：Gemini 需要「强制绝对路径、并行调用工具」等专项提醒；GPT 需要「别提计划别停下来、工具不好就换策略」等提醒。这些提醒只对对应模型有意义，对其他模型注入反而是噪音。

**Skills 索引的两层缓存**：Skills 列表用 LRU 内存缓存 + 磁盘快照双层缓存，key 由 `(skills_dir, 平台, 可用工具集, disabled列表)` 组成。冷启动也不用重新扫描所有 SKILL.md 文件。

---

## 03 Context 层：cwd 感知 + 注入优先级 + 防注入扫描

Context 层负责把当前工作目录的项目上下文注入进来。`build_context_files_prompt` 里有一个优先级链，只取第一个命中：

```
优先级（高 → 低）  
  .hermes.md / HERMES.md  ← 向上查找到 git root（最高优先级）  
  AGENTS.md / agents.md   ← 仅 cwd  
  CLAUDE.md / claude.md   ← 仅 cwd  
  .cursorrules + .cursor/rules/*.mdc
```

**只取第一个命中**：多个上下文文件并存时往往互相矛盾（AGENTS.md 说用 pytest，.cursorrules 说用 unittest），取优先级最高的一个能避免指令冲突。

**注入防注入扫描**：每个 context 文件在注入前都经过 `_scan_context_content()` 检测。命中任一威胁模式则原内容被替换为 `[BLOCKED: filename 包含疑似提示词注入]`，不注入：

* `ignore previous instructions` → `prompt_injection`
* `system prompt override` → `sys_prompt_override`
* `disregard your instructions` → `disregard_rules`
* 隐形 Unicode 方向覆盖字符（`\u200b`、`\u202e` 等）

这是工程上不能省的安全层——你放在项目里的 AGENTS.md 是完全受信任的，但如果它被恶意植入注入指令，整个 Agent 就可能被劫持。

**Gateway 模式要注意**：Hermes 进程的 `os.getcwd()` 是 Hermes 安装目录，不是用户的项目目录。必须通过 `TERMINAL_CWD` 环境变量告诉它正确的 cwd，否则会把 Hermes 自己的 AGENTS.md 注进去，平白消耗 10k+ tokens。

---

## 04 Volatile 层：记忆快照的「冻结」哲学

Volatile 层是最微妙的一层，注入顺序是：MEMORY.md 快照 → USER.md 快照 → 外部记忆提供者 → 时间戳（时间戳必须放最后）。

注入记忆时，`format_for_system_prompt` 拿的是**会话开始时拍的冻结快照**，不是实时状态：

```
# tools/memory_tool.py - MemoryStore 的冻结快照设计  
  
def load_from_disk(self):  
    """会话开始时调用一次：读取 MEMORY.md，拍快照"""  
    self.memory_entries = self._read_file("MEMORY.md")  
    # 拍快照，之后 system prompt 注入用这个  
    self._system_prompt_snapshot = {  
        "memory": self._render_block("memory", self.memory_entries),  
    }  
  
def format_for_system_prompt(self, target: str) -> Optional[str]:  
    """返回冻结快照，NOT 实时状态"""  
    return self._system_prompt_snapshot.get(target) or None  
  
def add(self, content: str):  
    """工具调用：更新实时状态 + 持久化，但不更新快照"""  
    self.memory_entries.append(content)  
    self._write_file("MEMORY.md", self.memory_entries)  
    # ← 注意：不更新 _system_prompt_snapshot！
```

**为什么不实时更新？** 因为 Anthropic 的前缀缓存按「system prompt 内容哈希」匹配。你改了任何一个字符，缓存就全部失效。一次对话里多轮 tool call，每次都重建 system prompt，缓存命中率是 0%。冻结快照让 system prompt 在整个会话内保持一字不变，每轮 tool call 都命中缓存。

这意味着：

* 你在本次对话写了一条新记忆 → 写入 MEMORY.md，但本次对话的 system prompt **不变**
* 下次对话开始时，`load_from_disk()` 重新拍快照，新记忆才生效

---

## 05 一次构建、永久复用：缓存生命周期与 SQLite 持久化

System prompt 构建完后，Hermes 还把它存入 SQLite，下次 resume 时直接读取，而不是重建：

```
# agent/conversation_loop.py - 系统提示缓存策略  
  
if agent._cached_system_prompt is None:  
    # 尝试从 SQLite 读上次存的 system prompt  
    session_row = agent._session_db.get_session(agent.session_id)  
    stored_prompt = (session_row or {}).get("system_prompt")  
  
    if stored_prompt:  
        # 继续上次会话：直接复用，Anthropic 前缀缓存直接命中！  
        agent._cached_system_prompt = stored_prompt  
    else:  
        # 全新会话：构建一次，存入 SQLite  
        agent._cached_system_prompt = agent._build_system_prompt(system_message)  
        agent._session_db.update_system_prompt(  
            agent.session_id, agent._cached_system_prompt  
        )
```

执行路径一共三条：

1. **全新会话**：构建 system prompt，存入 SQLite
2. **同一会话后续轮次**：直接用内存缓存，`O(1)` 复用
3. **`hermes resume` 继续上次会话**：从 SQLite 读取旧的 system prompt，和上次完全一样，Anthropic 前缀缓存直接命中

唯一触发重建的情况是**上下文压缩**——调用 `invalidate_system_prompt()`，清除缓存 + 重新从磁盘加载记忆快照（把本次对话写入的新记忆纳入进来）。压缩意味着上下文窗口已经很长了，缓存反正失效，这时候重建是合适时机。

---

## 06 ephemeral\_system\_prompt：不进缓存的「临时注入」

这是一个容易被忽视的设计细节。源码注释写着：

> Note: ephemeral\_system\_prompt is NOT included here. It's injected at API-call time only so it stays out of the cached/stored system prompt.

什么是 ephemeral\_system\_prompt？比如：

* 这一轮执行的是 cron 任务，临时告诉模型「你现在在定时任务里，没有用户在场」
* 工具返回了一个警告，临时注入「注意这个操作不可逆」

这类信息不应该进 system prompt 缓存。如果进了，下一轮对话（没有这个情境）依然会携带这段文字，造成误导。

Hermes 的处理方式：ephemeral 只在 API 调用时临时拼接到 cached 末尾，不存入 `_cached_system_prompt`，下一轮自动消失。只有 cached 部分会被前缀缓存命中。

这对应了一个通用设计原则：**把「只对本次请求有效」和「会话内持续有效」的信息严格分离**，不要混入同一个字符串。

---

## 07 四层记忆的注入顺序：为什么时间戳必须放最后？

Volatile 层的注入顺序不是随意的，每个位置都有原因：

| 顺序 | 内容 | 为什么在这个位置 |
| --- | --- | --- |
| 1 | MEMORY.md 快照 | 最稳定，会话开始冻结；与对话最相关，模型越早读到越好 |
| 2 | USER.md 快照 | 同样稳定；用户基本信息，背景性 |
| 3 | 外部记忆提供者 | 相对稳定，但可能按会话返回不同内容 |
| 4 | 时间戳 | 每次对话都变，**必须放最后** |

**时间戳为什么必须放最后？** LLM 前缀缓存只缓存"从字符串开头的连续不变部分"。时间戳每次对话都不同。如果它放在中间，它一变，后面所有内容的缓存全失效，等于把前面积累的缓存优势全部清零。放在末尾，前面三块的 token 每轮都能命中缓存。

**实测对比**（基于 Anthropic prompt caching 数据）：

| 方案 | 缓存命中率 | Token 成本 |
| --- | --- | --- |
| 每轮重建 system prompt | 0% | 基准 × 1.0 |
| 只缓存 Stable 层 | ~60% | 基准 × 0.7 |
| Stable + Context 缓存 | ~80% | 基准 × 0.5 |
| Hermes 完整三层 + 快照冻结 | ~95% | 基准 × 0.2 |

---

## 🕳️ 常见坑

**坑 1：把时间戳放在 system prompt 前面**

时间戳每次都变，放在前面等于让整个缓存每轮失效。Hermes 把时间戳放在 Volatile 层最末尾，就是为了让前面 95% 的内容保持稳定。

**坑 2：写入记忆后期望立即生效**

调用 `memory(action="add")` 写入后，本次会话的 system prompt 不会改变（快照已冻结）。**本次会话的新记忆要下次会话才生效**。对时效性要求高的信息，直接注入对话历史，不用记忆工具；或等上下文压缩时触发 `invalidate_system_prompt()` 重建。

**坑 3：多个上下文文件互相冲突**

项目里同时有 AGENTS.md 和 .cursorrules，两个文件对同一工具有不同说法。Hermes 的做法是优先级链取第一个命中，你的项目也应该如此——只保留一个 context 文件，避免歧义。

**坑 4：Gateway 模式忘记设 TERMINAL\_CWD**

在 gateway 模式下，`os.getcwd()` 是 Hermes 安装目录。必须通过 `TERMINAL_CWD` 告诉它用户的项目目录，否则会把 Hermes 自己的 AGENTS.md 注进去，平白消耗 10k+ tokens。

**坑 5：把可变内容存入 \_cached\_system\_prompt**

自己扩展 Hermes 时，如果把每轮都会变化的内容（比如当前任务进度、实时天气）塞进 `_cached_system_prompt`，缓存就形同虚设。这类内容应该走 ephemeral 注入，只影响当前 API 调用。

---

## 总结

**三层架构（Stable / Context / Volatile）是 system prompt 设计的核心：按变化频率分层，稳定内容在前，可变内容在后，这是前缀缓存命中率从 0% 提升到 95% 的根本原因。**

**记忆注入用"快照"不用"实时状态"：会话开始时拍一次快照，之后不再重建 system prompt，确保 Anthropic 前缀缓存在一次对话的所有 N 轮 tool call 中全部命中。**

**ephemeral\_system\_prompt 是"只对本次 API 调用有效"的临时注入：它不进缓存、不进 SQLite，下一轮自动消失，不污染稳定的缓存前缀。**

**时间戳放最后不是偶然，是精心设计的缓存策略——每次都变的内容必须放在字符串末尾，让前面 95% 的内容稳定命中缓存。**

**防注入扫描是不能省的安全层：AGENTS.md 如果被植入 `ignore previous instructions`，就能劫持整个 Agent，每次注入前必须扫描。**

**下一篇进入板块二收官：《用户建模与 Insights——Honcho 辩证模型怎么让 Agent 越用越懂你》，看 Hermes 如何把每次对话提炼成用户画像，在不侵犯隐私的前提下做到个性化。**

---

关注我，James 的成长日记，持续分享干货，帮你在 AI 时代少走弯路。