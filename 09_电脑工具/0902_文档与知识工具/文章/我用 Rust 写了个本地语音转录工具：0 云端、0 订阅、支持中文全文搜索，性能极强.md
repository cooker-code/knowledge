---
title: 我用 Rust 写了个本地语音转录工具：0 云端、0 订阅、支持中文全文搜索，性能极强
author: 老码小张
date: 小张小张
url: https://mp.weixin.qq.com/s?__biz=MzkxNzY0OTA4Mg==&mid=2247493272&idx=1&sn=83e6ab6cf03878de9fece1085199f123&chksm=c073ac1b1c83074771d88d43d6a0fbb19c402d4c304d8756732d2a0ffe26a4a157c871e03ff2&mpshare=1&scene=24&srcid=0531ELCHhB7duTmqELbmJcVf&sharer_shareinfo=9c17d2e68346dfca2c8bd4c08951b8bb&sharer_shareinfo_first=9c17d2e68346dfca2c8bd4c08951b8bb#rd
---

VoiceVault 是一个 CLI工具，后面会考虑给其添加一个界面，他是解决我日常开会转录的和总结待办用的一个纯离线的工具，其核心原理是采用，whisper.cpp 做转录，tantivy 做全文搜索，SQLite 存元数据，Ollama 跑本地 LLM 提行动项。录音、转录、索引、摘要全在本地完成。**一次编译，永久免费，零网络请求**（除了你主动下模型的那一下）。

### 废话不多说，直接看效果

四个核心能力：

| 能力 | 做了什么 |
| --- | --- |
| 🎙 **离线转录** | cpal 实时录麦克风 / symphonia 解码任意格式文件 → whisper-rs 本地推理（支持 Apple Metal 加速） |
| 🔍 **全文搜索** | tantivy 倒排索引 + CJK NgramTokenizer → 10 万段落 50ms 出结果，支持布尔 / 短语 / 字段限定语法 |
| 🤖 **本地 LLM 摘要** | 一个 `LlmBackend` trait 同时支持 Ollama（本地 http://localhost:11434）和 OpenAI 兼容 API（DeepSeek / 智谱 / 月之暗面都跑得了） |
| 🔒 **完全本地** | 除了你主动执行 `models download` 拉模型，没有任何网络请求；SQLite + 本地 tantivy 索引 + 本地音频，随时可带走 |

一个真实的会议录音跑完端到端（23 秒录音，Apple Silicon，基础 `base` whisper 模型）：

```
$ voicevault transcribe ./meeting.mp3 -l zh  
  
✓ 会话已创建：be4c2663-4146-4fc4-836a-5e691f99fad0  
  语言: zh  
  段落: 1，总长 00:00:23  
  
📝 摘要：会议讨论了产品第二季度的路线图，确定了将月活目标从 50 万提升至 80 万。  
        张三负责在下周五前完成竞品分析报告，李四需更新技术方案并与供应商沟通价格。  
        预算问题决定推迟到下周再讨论。  
  
✓ 识别到 2 条行动项
```

```
$ voicevault actions list  
┌────┬─────────────┬────────┬────────────────────────────────┬──────┐  
│ ID │ Session     │ 负责人 │ 任务                            │ 完成 │  
├────┼─────────────┼────────┼────────────────────────────────┼──────┤  
│  1 │ be4c2663-4… │ 张三   │ 在下周五前完成竞品分析报告      │      │  
│  2 │ be4c2663-4… │ 李四   │ 更新技术方案并与供应商沟通价格   │      │  
└────┴─────────────┴────────┴────────────────────────────────┴──────┘  
  
$ voicevault search "季度目标"  
Score 1.67  Q2产品周会  00:00:00  今天的会议主题是产品第二【季度】的路线图...  
  
$ voicevault export be4c2663 -f markdown -o notes.md  
✓ 已写入 notes.md
```

录音 → 转录 → 存库 → 索引 → LLM 提行动项 → 搜索 → Markdown 导出，整条流水线在你的 MacBook 上跑完。**没碰一次云端**。

### 我的架构设计，就是一个 workspace + 两个 crate

```
voicevault/  
├── Cargo.toml                   # workspace  
├── crates/  
│   ├── voicevault-core/         # 核心库（无 UI 依赖）  
│   │   ├── audio/               # cpal 录制 + symphonia 解码 + rubato 重采样  
│   │   ├── transcribe/          # whisper-rs 封装  
│   │   ├── llm/                 # LlmBackend trait + Ollama / OpenAI 实现  
│   │   ├── action/              # 基于 LLM 的摘要/行动项提取  
│   │   ├── search/              # tantivy 索引  
│   │   ├── storage/             # rusqlite 会话/段落/行动项  
│   │   └── export/              # md / txt / json / srt / vtt  
│   └── voicevault-cli/          # CLI 入口 (clap)  
└── README.md
```

把核心逻辑沉淀到 `voicevault-core`，让未来的 Tauri 桌面端、VS Code 插件甚至 Neovim 插件可以共用同一套代码。CLI 只是一个薄薄的 clap 壳。

**依赖选型**（每一个都是纯 Rust，没有 Python 生态的 pip 地狱）：

| 模块 | Crate | 为什么 |
| --- | --- | --- |
| 转录 | `whisper-rs 0.16` | whisper.cpp 的 Rust 绑定，Metal/CoreML 加速开箱即用 |
| 搜索 | `tantivy 0.22` | 纯 Rust 的 Lucene，不需要 JVM，ripgrep 级速度 |
| 音频 | `cpal 0.15` + `symphonia 0.5` + `rubato 0.15` | 录制、解码、重采样，全平台 |
| 存储 | `rusqlite 0.32` + `r2d2` | 本地 SQLite，bundled feature 零运行时依赖 |
| CLI | `clap 4.5` + `indicatif` + `comfy-table` | 参数解析、进度条、表格渲染 |
| HTTP | `reqwest 0.12` + `tokio 1` | Ollama / OpenAI 兼容 API 的 HTTP 客户端 |

总计 **38 个源文件，3800 行 Rust**。14 个单元测试，release 构建后二进制 13MB。

### 我的踩坑复盘，让我怀疑人生的坑

#### 1：tantivy 默认分词器不认中文

MVP 刚跑通第一天，搜索功能测试通过了 —— 然后我把它喂给一段 23 秒的中文会议，结果：

```
$ voicevault search "季度"  
未找到结果
```

明明转录文本里白纸黑字写着「今天的会议主题是产品第二**季度**的路线图」，怎么会搜不到？

**原因**：tantivy 的默认 `SimpleTokenizer` 只按空格和标点切词。中文段落没有空格，整段「今天的会议主题是产品第二季度的路线图」被当成 **一个 token**。你搜「季度」，对应的 token 从未存在，当然命中 0。

这是所有 Lucene 家族搜索引擎对 CJK 的经典问题。一般解决方案有三种：

1. 1. 接入专门的中文分词器（`jieba-rs` / `cang-jie`）—— 效果最好，但要词表、要维护
2. 2. **Ngram 分词**（1-2 gram）—— 内置方案，效果够用，零依赖
3. 3. 按单字切分 —— 最粗暴，索引爆炸

我选了方案 2。代码改动很小：

```
use tantivy::schema::{IndexRecordOption, TextFieldIndexing, TextOptions};  
use tantivy::tokenizer::{LowerCaser, NgramTokenizer, TextAnalyzer};  
  
const TOKENIZER_NAME: &str = "cjk_ngram";  
  
fn build_schema() -> Schema {  
    let mut b = Schema::builder();  
    // ... 其他字段 ...  
    let text_indexing = TextFieldIndexing::default()  
        .set_tokenizer(TOKENIZER_NAME)  
        .set_index_option(IndexRecordOption::WithFreqsAndPositions);  
    let text_opts = TextOptions::default()  
        .set_indexing_options(text_indexing)  
        .set_stored();  
    b.add_text_field("text", text_opts);  
    b.build()  
}  
  
fn register_tokenizers(index: &Index) {  
    // 1-2 字符 n-gram，覆盖中英文常见情形  
    let ngram = NgramTokenizer::new(1, 2, false).expect("valid ngram config");  
    let analyzer = TextAnalyzer::builder(ngram).filter(LowerCaser).build();  
    index.tokenizers().register(TOKENIZER_NAME, analyzer);  
}
```

这样 `"季度"` 被拆成 bigram `季度` 存进倒排；搜索 `"季度"` 直接命中。`"季度目标"` 拆成 `季`, `季度`, `度`, `度目`, `目`, `目标`, `标`，任选一 gram 都能召回。

**附送 bonus**：schema 变更后旧索引目录会失效。我加了一个版本号文件 `.voicevault_index_version`，检测到不一致就自动清空重建，并从 SQLite 回填全部段落。用户不需要手动跑任何迁移。

#### 坑 2：qwen3.5:9b 让我多等了 90 秒

第一次跑 LLM 行动项提取，一个 23 秒的录音竟然等了 95 秒才拿到结果。我还以为是网络问题（虽然是本地 Ollama）。

打开 Ollama 的 API 响应一看：

```
{  
  "message": {  
    "content": "{\"summary\":\"…\",\"actions\":[…]}",  
    "thinking": "好的，我现在需要处理用户的请求...（3000 字的内心戏）"  
  },  
  "total_duration": 95495729459,  
  "eval_duration": 4653098673  
}
```

`qwen3.5:9b` 是个**推理模型**，它在给答案前会先输出一大段 `thinking` 思维链。总耗时 95s 里，真正生成 JSON 只用了 4.6s，剩下 90 秒全在「思考」—— 这对一个结构化抽取任务来说，完全是浪费。

Ollama 的 API 支持一个 `think: false` 参数直接关掉推理。我更新 `OllamaBackend` 请求结构：

```
#[derive(Serialize)]  
struct ChatReq<'a> {  
    model:    &'a str,  
    messages: Vec<ChatMsg<'a>>,  
    stream:   bool,  
    /// 禁用 qwen3 / deepseek-r1 等推理模型的 <think>…</think> 段，  
    /// 对结构化抽取无价值，显著增加延迟。  
    think:    bool,  
    format:   Option<&'static str>,  // "json" 强制 JSON 模式  
    options:  ChatOpts,  
}
```

请求时固定 `think: false`，时长从 95s 降到 **7s**，约 13 倍加速。

这一条留给所有接 LLM 的同学：**如果你只要结构化输出，永远记得关掉 reasoning**。reasoning 模型非常强，但不是每一个任务都需要它去"思考"。

### 双 LLM 后端，一个 Trait 同时接 Ollama 和 DeepSeek

MVP 讨论时内心纠结过：是坚持 llama-cpp-2 打包进二进制做到极致自包含，还是做个抽象层让用户选？

最终选了抽象层 —— 因为现实里：

* • 有人习惯本地跑 Ollama（M1/M2/M3 本地推理完全够用）
* • 有人公司封了本地推理（公司只认云 API，但有 DeepSeek 或 Moonshot 的 key）
* • 有人想用 DeepSeek 的质量 + 本地隐私兼得（转录在本地，LLM 走低成本国产 API）

一个 trait 解决所有问题：

```
#[async_trait]  
pub trait LlmBackend: Send + Sync {  
    async fn generate(&self, prompt: &str, opts: &GenerateOptions) -> Result<String>;  
    fn model_name(&self) -> &str;  
    fn backend_name(&self) -> &'static str;  
}  
  
pub fn build_from_config(cfg: &LlmConfig) -> Result<Box<dyn LlmBackend>> {  
    match cfg.backend_kind()? {  
        BackendKind::Ollama => Ok(Box::new(OllamaBackend::new(/*…*/)?)),  
        BackendKind::OpenAi => Ok(Box::new(OpenAiBackend::new(/*…*/)?)),  
    }  
}
```

用户只需要改一行 config：

```
# 本地 Ollama  
[llm]  
backend  = "ollama"  
base_url = "http://localhost:11434"  
model    = "qwen3.5:9b"  
  
# 切到 DeepSeek  
[llm]  
backend  = "openai"  
base_url = "https://api.deepseek.com/v1"  
model    = "deepseek-chat"  
api_key  = "sk-…"  
  
# 切到 OpenAI  
[llm]  
backend  = "openai"  
base_url = "https://api.openai.com/v1"  
model    = "gpt-4o-mini"  
api_key  = "sk-…"
```

一行都不用改代码。**同一份 prompt，同一份 JSON 解析逻辑，适配任何 OpenAI 兼容服务**。

### 性能数据（M1 Pro 16G）

| 场景 | 模型 | 结果 |
| --- | --- | --- |
| 转录 23 秒中文会议 | whisper `base` (150MB) | **0.86 秒** |
| 转录 1 小时中文会议（CPU） | whisper `small` (500MB) | ~18 分钟 |
| 转录 1 小时（启用 Metal feature） | whisper `small` | **~6 分钟** |
| tantivy 搜索 10 万段落 | - | **< 50 ms** |
| 行动项提取（Ollama qwen3.5:9b） | - | 7–15 秒 / 小时会议 |
| 行动项提取（DeepSeek API） | - | 3–10 秒 |
| 应用冷启动 | - | **< 200 ms** |
| 二进制大小 (release) | - | 13 MB |

作为参考：同样的 1 小时音频，Python 生态的 `whisper` 在 CPU 上需要约 1 倍实时（≈60 分钟），是 whisper-rs 的 10 倍慢。

### 开箱即用

#### 前置依赖（macOS）

```
brew install cmake  
xcode-select --install       # clang 工具链
```

#### 构建

```
git clone <repo>  
cd voicevault  
cargo build --release --features metal   # Apple Silicon 启用 Metal 加速  
cp target/release/voicevault ~/.local/bin/
```

第一次编译要等 5–10 分钟，whisper.cpp 需要现场构建。后续增量秒级。

#### 5 分钟上手

```
# 1. 下载 whisper 模型（75MB 起）  
voicevault models download tiny  
  
# 2. 启动 Ollama（如果用本地 LLM）  
ollama serve &  
ollama pull qwen3.5:9b   # 或者你喜欢的其他模型都行  
  
# 3. 转录第一个文件  
voicevault transcribe ./meeting.mp3 -l zh  
  
# 4. 搜索历史会议  
voicevault search "季度目标"  
  
# 5. 导出 Markdown  
voicevault export <session_id> -f markdown -o notes.md  
  
# 6. 列出所有行动项  
voicevault actions list
```

或直接录麦克风：

```
voicevault record -d 60 -l zh -t "今晚周会"  
# 60 秒后自动停止并转录
```

### 为什么是 Rust

这个项目可以用 Python/Go/TypeScript 写。选 Rust 有几个实打实的理由：

1. 1. **whisper-rs / tantivy 就是 Rust** —— 原生绑定，没有 FFI overhead，没有 GIL 限制
2. 2. **一个二进制跑遍 macOS / Linux / Windows**，用户不用装 Python 虚拟环境、pip install、编译 wheel
3. 3. **tokio 做好音频采集 + LLM HTTP 请求的并发**，天然适合流式场景
4. 4. **内存和启动时间**：13MB 二进制、<200ms 冷启动，远好于 Electron 或 Python 方案
5. 5. **后面可以复用同一份 core crate 套 Tauri**（Rust 后端 + Web 前端），相比 Electron 体积能砍一个量级

### 最后几句话

这个项目从 0 到 MVP 用了一个周末。38 个文件，3800 行 Rust，14 个单元测试，全部跑绿。

云 SaaS 的便利是真实的，但当你发现自己为了 300 分钟免费额度，把一个包含薪资讨论的录音上传到了美西某个数据中心时，那种便利就开始发苦。

离线工具不是怀旧。离线是一种**选择权**：我可以随时把 voicevault 关机、把硬盘拔下来带走、在一台断网的笔记本上继续使用它的全部功能。这种选择权，过去十年被云端订阅悄悄拿走了。我想拿回来一点。

**VoiceVault**，小群内部开源，欢迎 Star / PR / 吐槽。

开源地址：http://github.com/coder-brzhang/voicevault

注意，本项目仅在小张的400 多个人的小群（公众号菜单-联系我-加群）中分享。