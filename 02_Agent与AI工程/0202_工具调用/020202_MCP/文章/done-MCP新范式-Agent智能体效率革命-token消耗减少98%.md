> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020202_MCP/020202_核心知识点/MCP生产接入与治理边界|MCP生产接入与治理边界]]
---
title: MCP新范式-Agent智能体效率革命-token消耗减少98%
author: AI希望星
date:
url: https://mp.weixin.qq.com/s?__biz=Mzk4ODM3MTEwNA==&mid=2247483999&idx=1&sn=175c5fc2362634d7fc7d1d41d6bd8e00&chksm=c4e231c6c9fcae156816b519b584d705bfdbb02a974170541eccb7335d0805661b268422c58c&mpshare=1&scene=24&srcid=11104wBP9JcAqLrti3fIkn0j&sharer_shareinfo=4de69ca547247bb29559b507c18403f6&sharer_shareinfo_first=4de69ca547247bb29559b507c18403f6#rd
---

Anthropic 发布最新MCP文章、通过**代码执行结合 MCP（Model Context Protocol）** 提升 AI 智能体效率的方案。

1、MCP 作为 2024 年 11 月推出的 AI 智能体连接外部系统的开放标准协议，已成为行业事实标准，拥有数千个社区构建的 MCP 服务器及全主流编程语言 SDK。

2、传统MCP直接工具调用存在**工具占用上下文窗口**和**中间结果消耗额外 tokens** 两大问题。（如连接数千工具需处理数十万 tokens、2 小时会议转录文本重复传输或超上下文限制）；

3、通过代码执行（如将 MCP 工具封装为代码 API、按文件树组织结构工具），**可实现工具按需加载（使 token 消耗从 150,000 降至 2,000，节省 98.7%）、数据预处理、复杂控制流执行**，还能带来隐私保护（中间结果默认留在执行环境、PII 自动 token 化）、状态持久化（通过文件系统保存中间结果与可复用技能代码）等优势。

---

---

# 二、MCP（Model Context Protocol）核心介绍

MCP 是连接 AI 智能体与外部系统的**开放标准协议**，解决传统智能体与工具 / 数据连接需 “一对一自定义集成” 的问题，避免碎片化与重复开发，实现 “一次集成 MCP，解锁全生态工具”。

# 三、传统直接工具调用的两大核心问题

# 问题 1：工具定义过载上下文窗口

**表现：多数 MCP 客户端将所有工具定义（含描述、参数、返回值）直接加载到模型上下文，占用大量空间，导致响应时间延长、成本上升。**

**示例：单个工具定义（如gdrive.getDocument、salesforce.updateRecord）需包含完整说明，若连接数千工具，智能体需先处理****数十万 tokens**才能读取用户请求。

# 问题 2：中间结果消耗额外 tokens

**表现：模型直接调用工具时，所有中间结果需传入上下文，导致重复传输、大文件超限制、数据复制易出错。**

**示例：**

用户需求 “从谷歌云盘下载会议转录文本并附加到 Salesforce 线索”，流程中：gdrive.getDocument返回的转录文本（如 2 小时会议约 50,000 tokens）需入上下文；salesforce.updateRecord调用时，需再次将完整转录文本写入上下文；风险：大文件可能超出上下文窗口限制，导致工作流中断；模型复制数据时易出错。

**表现：模型直接调用工具时，所有中间结果需传入上下文，导致重复传输、大文件超限制、数据复制易出错。**

**四、代码执行结合 MCP 的解决方案**

# 1. 核心思路

将 MCP 服务器以**代码 API 形式呈现**，智能体通过编写代码与 MCP 工具交互，而非直接调用工具，实现 “工具按需加载” 与 “数据在执行环境预处理”，解决传统方案的 token 消耗问题。

# 2. 工具组织与代码实现

**工具文件树结构**

：按服务器分类组织工具代码，示例（TypeScript）：

```
servers
├── google-drive
│ ├── getDocument.ts // 谷歌云盘文档获取工具
│ └── index.ts
├── salesforce
│ ├── updateRecord.ts // Salesforce记录更新工具
│ └── index.ts
└── ...（其他服务器）
```

* **单个工具代码示例**

  （google-drive/getDocument.ts）：

```
import { callMCPTool } from"../../../client.js";
interface GetDocumentInput {
 documentId: string;
}
interface GetDocumentResponse {
 content: string;
}
/* Read a document from Google Drive */
exportasyncfunctiongetDocument(input: GetDocumentInput): Promise<GetDocumentResponse> {
return callMCPTool<GetDocumentResponse>('google_drive__get_document', input);
}
```

* **用户需求代码实现**

  （谷歌云盘→Salesforce 数据传输）：

```
import * as gdrive from'./servers/google-drive';
import * as salesforce from'./servers/salesforce';
const transcript = (await gdrive.getDocument({ documentId: 'abc123' })).content;
await salesforce.updateRecord({
objectType: 'SalesMeeting',
recordId: '00Q5f000001abcXYZ',
data: { Notes: transcript }
});
```

# 3. 工具发现机制

智能体通过**探索文件系统**发现工具：

列出./servers/目录，找到可用服务器（如google-drive、salesforce）；

读取所需工具文件（如getDocument.ts），仅加载当前任务需的工具定义；

可选优化：添加search\_tools工具，支持按关键词搜索（如 “salesforce”），并通过 “细节级别参数” 选择加载内容（名称仅 / 名称 + 描述 / 完整定义），进一步节省上下文。

# 五、代码执行结合 MCP 的六大核心优势

# 1. 渐进式披露（工具按需加载）

核心价值：智能体仅加载当前任务需的工具定义而非全部大幅减少 token 消耗。

数据支撑：token 使用量从 150,000 降至 2,000，**时间与成本节省 98.7%**。

# 2. 上下文高效的工具结果处理

核心价值：大数据集在执行环境内过滤 / 聚合 / 提取，仅将关键结果传入上下文。

示例：获取 10,000 行表格时，代码内过滤 “待处理订单”，仅向智能体返回前 5 行，避免 10,000 行数据占用上下文：

```
// 代码内过滤，仅返回5行
const allRows = await gdrive.getSheet({ sheetId: 'abc123' });
const pendingOrders = allRows.filter(row => row["Status"] === 'pending');
console.log(`Found ${pendingOrders.length} pending orders`);
console.log(pendingOrders.slice(0, 5));
```

# 3. 更强大的上下文高效控制流

核心价值：用代码实现循环、条件、错误处理，替代工具调用链，减少

latency 与 token 消耗。

示例：Slack 部署通知监听，用循环 + 延迟实现持续检测，无需交替调用工具与睡眠命令：

```
let found = false;
while (!found) {
const messages = await slack.getChannelHistory({ channel: 'C123456' });
 found = messages.some(m => m.text.includes('deployment complete'));
if (!found) awaitnewPromise(r => setTimeout(r, 5000));
}
console.log('Deployment notification received');
```

额外收益：减少 “首 token 生成时间”，因条件判断由执行环境处理，无需等待模型评估。

# 4. 隐私保护操作

核心机制 1：中间结果默认留在执行环境，仅显式日志 / 返回值传入模型上下文。

核心机制 2：敏感数据自动 token 化（MCP 客户端拦截处理），真实数据不进模型。

示例：导入客户数据到 Salesforce 时，PII（邮箱、电话）被转为[EMAIL\_1]、[PHONE\_1]，仅在工具调用时通过 MCP 客户端解 token 化，避免敏感数据泄露。

MCP客户端拦截数据并在数据到达模型之前对PII进行标记化：

```
// What the agent would see, if it logged the sheet.rows:
[
{ salesforceId: '00Q...', email: '[EMAIL_1]', phone: '[PHONE_1]', name: '[NAME_1]' },
{ salesforceId: '00Q...', email: '[EMAIL_2]', phone: '[PHONE_2]', name: '[NAME_2]' },
...
]
```

扩展价值：支持定义确定性安全规则，控制数据流向。

# 5. 状态持久化与技能复用

# （1）状态持久化

实现方式：通过文件系统保存中间结果，支持任务断点续传。

示例：查询 Salesforce 线索后保存为 CSV，后续执行可直接读取：

```
// 保存中间结果
const leads = await salesforce.query({ query: 'SELECT Id, Email FROM Lead LIMIT 1000' });
const csvData = leads.map(l =>`${l.Id},${l.Email}`).join('\n');
await fs.writeFile('./workspace/leads.csv', csvData);
// 后续读取续用
const saved = await fs.readFile('./workspace/leads.csv', 'utf-8');
```

# （2）技能复用

实现方式：将任务代码封装为可导入函数，搭配SKILL.md形成结构化 “技能”，供后续调用。

示例：封装 “表格转 CSV” 技能，后续直接导入使用：

```
// 技能封装（./skills/save-sheet-as-csv.ts）
import * as gdrive from'./servers/google-drive';
exportasyncfunctionsaveSheetAsCsv(sheetId: string) {
const data = await gdrive.getSheet({ sheetId });
const csv = data.map(row => row.join(',')).join('\n');
await fs.writeFile(`./workspace/sheet-${sheetId}.csv`, csv);
return`./workspace/sheet-${sheetId}.csv`;
}
// 后续调用
import { saveSheetAsCsv } from'./skills/save-sheet-as-csv';
const csvPath = await saveSheetAsCsv('abc123');
长期价值：智能体可积累 “工具库”，提升特定任务性能。
```

# 6. 行业实践验证

Cloudflare 已推出类似方案（“Code Mode”），核心结论一致：LLM 擅长编写代码，利用该优势可提升 MCP 交互效率。

# 六、注意事项与权衡

**额外成本：代码执行需构建****安全执行环境**，包括沙箱隔离、资源限制、实时监控，增加运维与安全成本。

**核心建议：需权衡 “代码执行的效率收益（降 token 成本、减 latency、优工具组合）” 与 “安全环境的实现成本”，并向 MCP 社区共享实践经验。**

# 七、总结：

# MCP 为智能体连接多工具 / 系统提供基础协议，但传统直接调用存在 token 消耗过高问题；代码执行将软件工程的成熟模式（如模块化、文件系统、代码控制流）应用于智能体，实现 MCP 交互效率跃升，同时带来隐私、状态管理等附加价值；未来需在效率与安全成本间找到平衡，推动社区协作优化。

---

# 关键问题：

# 问题 1：MCP（Model Context Protocol）作为行业事实标准，其核心价值与传统智能体 - 工具连接方式相比，最大差异是什么？

**答案：MCP 的核心价值是解决传统连接方式的 “碎片化与重复开发” 问题。传统方式中，智能体与每个外部工具 / 数据系统连接都需单独自定义集成，导致开发效率低、生态碎片化；而 MCP 作为开放标准，开发者只需在智能体内****实现一次 MCP 集成**，即可解锁整个 MCP 生态的工具（截至 2025 年 11 月，社区已构建数千个 MCP 服务器，SDK 覆盖全主流编程语言），大幅降低集成成本，实现智能体的规模化连接。

# 问题 2：代码执行结合 MCP 为何能将工具交互的 token 消耗降低 98.7%（从 150,000 tokens 降至 2,000 tokens）？其核心机制是什么？

# 答案：核心原因是解决了传统直接工具调用的 “全量加载” 问题，通过 “**工具按需加载**” 机制实现 token 节省。传统方式中，MCP 客户端会将所有工具的完整定义（含描述、参数、返回值）一次性加载到模型上下文，若连接数千工具需处理数十万 tokens；而代码执行模式下，MCP 工具被封装为按服务器分类的代码文件（如servers/google-drive/getDocument.ts），智能体通过探索文件系统，仅读取当前任务所需的工具代码（如仅加载getDocument.ts和updateRecord.ts），无需加载全部工具定义，因此 token 消耗从 150,000 降至 2,000，节省 98.7%。

# 问题 3：在处理含敏感数据（如客户邮箱、电话）的任务时，代码执行结合 MCP 如何保障数据隐私？与传统直接工具调用相比，隐私保护能力有何提升？

**答案：代码执行结合 MCP 通过 “****中间结果隔离**” 和 “**敏感数据自动 token 化**” 两大机制保障隐私，相比传统方式有显著提升：中间结果隔离：传统方式中，所有工具调用的中间结果（如客户数据）需传入模型上下文；而代码执行模式下，中间结果默认留在执行环境，仅显式日志或返回值才进入模型上下文，减少敏感数据暴露范围。智能体效率革命，

敏感数据自动 token 化：MCP 客户端会拦截代码中的敏感数据（如 PII），自动将其转为无意义 token（如邮箱user@example.com转为[EMAIL\_1]），真实数据仅在工具调用时通过 MCP 客户端解 token 化传递（如从谷歌云盘到 Salesforce），从未进入模型上下文；而传统方式中，敏感数据会完整传入上下文，存在被模型误处理或泄露的风险。