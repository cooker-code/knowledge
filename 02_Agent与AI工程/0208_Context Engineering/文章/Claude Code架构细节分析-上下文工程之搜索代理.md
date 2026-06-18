---
title: Claude Code架构细节分析-上下文工程之搜索代理
author: 锦康灵感铺
date: 
url: https://mp.weixin.qq.com/s?__biz=MzE5ODM0MTIyNQ==&mid=2247483765&idx=1&sn=e75d9a8cae8c1e43c5a3e518859f5d99&chksm=97b28a69577d0d324e4b976978a86dfaabb80c256755049c511e19cc1bbe331312a8abed7253&mpshare=1&scene=24&srcid=1028FDg7HXezwQTDYK8HMEiE&sharer_shareinfo=be5923dc64523f79bd42bb4cc7295558&sharer_shareinfo_first=be5923dc64523f79bd42bb4cc7295558#rd
---

# ClaudeCode架构细节分析-上下文工程之搜索代理

> 在具体实现的部分中，为了方便阅读，代码我只保留核心的结构，如果需要完整代码可以前往github仓库查看

## 一、什么是搜索代理

在聊搜索代理之前，就需要先搞清楚代理是什么，简单理解可以是：**LLM 在自主循环中使用工具**

🌟 那么搜索代理的简单定义：**LLM 使用各种搜索或检索工具，动态地，按需地获取相关上下文**

我在测试各种模型对于检索工具的调用的时候，发现模型能力差异导致的搜索结果不同，优秀的模型的搜索路径规划的更加合理，并且搜索的方向非常准确，而能力较差的模型，方向飘忽不定，结果也很一般

> 如果需要测试日志的可以前往github仓库下载查看

所以我发现影响搜索代理成功的因素之一是：更智能的模型，可以使搜索代理自主应对复杂问题并从错误中恢复

## 二、搜索代理和 RAG 的区别

搜索代理和 RAG 的区别

搜索代理和 RAG 的最主要的区别在于工作执行流程：

* • RAG 是预推理检索：系统预先从向量数据库检索出来结果之后输入给 LLM，这个时候 **LLM 是被动的接受**
* • 搜索代理是即时检索：**LLM 是主动决策者**，由 LLM 来决定搜索什么，怎么搜索，并且可以根据中间结果来主动的调整搜索路径

搜索代理和 RAG 还有一个在**优化行为方式的区别：**

搜索代理中有一种工具是 `glob`，这个工具是用来搜索和匹配文件名的，这个时候文件名也可以成为有效搜索因素，当一个代码库中的文件结构非常清晰，并且文件命名合理，在这种工作空间下搜索代理会非常有效，搜索代理的影响因素如下：

* • 文件夹的结构： 例如：`tests/test_agent.ts` 这个就表示测试模块中的测试 agent 功能的文件的隐性含义
* • 命名约定：例如：`*.config.js` 表示配置文件 、`README.md` 表示文档说明
* • 文件大小：可以隐含复杂性的背景信息
* • 时间戳：可以表示文件是最近修改的，隐含该文件相关性比较强(因为这个文件是最近修改的，那用户输入的问题很大概率是因为他刚刚修改这个文件产生的，我们可以仔细想想自己使用 ClaudeCode 或者 Cursor 的 Agent 模式的时候是不是这样的呢）

在这种构建方式下，代理可以逐层构建理解，仅在工作记忆中保留必要信息，**这种自我管理的上下文窗口使代理专注于相关子集，而不是被大量但可能无关的信息淹没。**

而对于 RAG 来说，只能重新构建嵌入文档块了，而不是像搜索代理这样随心所欲的动态更换一下文件名就可以，这也是一个关键区别**搜索代理的维护成本比较低，开发起来比较快**

当然，事物总是相对的，RAG 还是有优秀的地方的

* • 在大量数据时，RAG 的检索会比搜索代理快很多
* • RAG 的开发构建策略是平稳的，有成熟的构建搜索方案：分叉索引、父文档索引等，但是搜索代理要有效的话，需要有深思熟虑的工程实践，要有成熟的指导开发方案，不如搜索代理会很容易进入死循环和无效检索来浪费上下文，在这个方向上，搜索代理的开发难度比 RAG 大多了

那我们来总结一下 RAG 和搜索代理的区别点

1. 1. **工作执行流程的区别**，RAG 是预推理检索，搜索代理是即时检索
2. 2. **优化行为方式的区别**，搜索代理的优化方向更多，策略是可以动态调整的，RAG 需要重新构建文档块
3. 3. **开发和维护成本的区别**：搜索代理的维护成本比较低，文件系统工具即时开发，不需要像 RAG 那样整理文档块，然后构建向量数据库等，并且搜索代理调整和维护比较容易
4. 4. **大量数据的检索速度区别**：在大量数据的前提下，RAG 检索速度会比搜索代理快很多
5. 5. **有效搜索代理开发难度大**：要想搜索代理在实际的场景中有效，需要有合适的策略指导

## 三、搜索代理的应用场景

1、 **对于上下文信息密度有较高的要求**：例如编程开发领域，开发者使用代码来实现自己严密的业务逻辑，这种场景下使用代理搜索会非常合适

倘若我们采用 RAG 的方式来检索，我们需要对代码库进行文档块的分割，就不能像文本那样简单的按照行数来分割了，因为代码上下文之间有很强的逻辑依赖，如果文档块中少一行 if 判断，表达的意思就是天差地别了，比较合适的的方法是结合语法树和代码依赖关联来进行分割，

所以这种情况下，不仅存入向量数据库的过程更麻烦，检索时也因为内部机制不透明，导致开发者很难看清检索的全貌，调试起来也更困难。

2、**动态上下文不多**：要动态传入给 LLM 的动态外部数据不多的时候，搜索代理很适用，例如：中小型代码库、一些公司内部客服手册这种相对固定的数据

我非常推荐开发者在构建大模型应用初期的时候，完全可以优先考虑使用搜索代理来实现外部数据的引入，在应用构建初期的时候，遵循着快速迭代的原则，搜索代理的开发难度和开发效率相比复杂的 RAG 是有优势的

3、**偏向简单有效原则构建 Agent**：大模型应用开发这个领域是在快速发展的，而且大部分都是向着“LLM 越来越智能”这个方向前行，构建简单的模块和应用是最合适的，方便应用和模块进行迭代，像渐进式构建有效应用的原则一样

## 四、具体实现

具体实现

我是使用 Ts 来实现的，我比较熟悉这个语言，我的实现比较简单，没有做过多的兼容，只保证自己本地运行成功，在实现的过程中我发现一些复杂的地方：

* • 一个成熟的执行工具需要兼容多种系统的版本，这个时候是比较复杂的
* • 工具的描述和名称要定义的合理，表达的清晰，不然模型会混乱调用

### 4.1、工具实现

#### 4.1.1、GrepTool 工具实现

实现 GrepTool 工具对象时，需要一些工具的参数定义，比较重要的是工具参数

* • pattern：搜索的正则表达式
* • path：搜索路径（目录）
* • include：文件模式过滤

```
// 因为源代码太多了，我这里保留核心的代码结构  
import { InternalTool, InternalToolContext } from '../types.js';  
import { ripGrep } from '../../utils/ripgrep.js';  
  
export const TOOL_NAME_FOR_PROMPT = 'GrepTool';  
export const DESCRIPTION = `  
- 适用于任何代码库大小的快速内容搜索工具  
- 使用正则表达式搜索文件内容  
- 支持完整的正则表达式语法（例如 "log.*Error"、"function\\s+\\w+" 等）  
- 使用 include 参数按模式过滤文件（例如 "*.js"、"*.{ts,tsx}"）  
- 返回按修改时间排序的匹配文件路径  
- 当你需要查找包含特定模式的文件时使用此工具  
- 当你进行可能需要多轮 glob 和 grep 的开放式搜索时，请改用 Agent 工具  
`;  
  
  
/**  
 * GrepTool 参数定义  
 */  
export interface GrepToolArgs {  
  /** 搜索的正则表达式模式 */  
  pattern: string;  
  /** 搜索路径（目录） */  
  path?: string;  
  /** 文件模式过滤 */  
  include?: string;  
}  
  
/**  
 * GrepTool 返回结果  
 */  
export interface GrepToolResult {  
  /** 匹配的文件路径 */  
  matches: string[];  
  /** 匹配总数 */  
  count: number;  
}  
  
/**  
 * GrepTool 处理函数  
 */  
const grepToolHandler = async (  
  args: GrepToolArgs,  
  context?: InternalToolContext  
): Promise<GrepToolResult> => {  
  // ...  
  const results = (await ripGrep(rgArgs, path, abortSignal)) as string[];  
  
  return {  
    matches: results,  
    count: results.length,  
  };  
};  
  
/**  
 * GrepTool 工具定义  
 */  
export const GrepTool: InternalTool<GrepToolArgs, GrepToolResult> = {  
  name: 'grep_search',  
  category: 'search',  
  internal: true,  
  description: DESCRIPTION,  
  version: '1.0.0',  
  parameters: {//...},  
    required: ['pattern'],  
  },  
  handler: grepToolHandler,  
};
```

在实现这个 grepTool 工具函数的时候，使用的是 ripgrep 命令，这个命令执行的速度非常快，比一般的搜索命令要快很多，在构建这个命令执行的函数需要注意两点：

1. 1. Mac 系统需要构建执行文件的签名
2. 2. 命令执行的时候需要判断，如果本地电脑没有 rg 这个命令，就需要使用 vendor 里面的二进制文件来执行了

> 使用命令 ripgrep：https://github.com/BurntSushi/ripgrep

```
// 因为源代码太多了，方便阅读查看，我这里保留核心的代码结果  
  
// 获取rg命令执行路径  
function ripgrepPath() {//...}  
  
export async function ripGrep(  
  args: string[],  
  target: string,  
  abortSignal: AbortSignal  
) {  
  //mac电脑的签名  
  await macSignature();  
  const rgPath = ripgrepPath();  
  
  return new Promise(resolve => {  
    execFile(  
      ripgrepPath(),  
      [...args, target],  
      {  
        maxBuffer: 1_000_000,  
        signal: abortSignal,  
        timeout: 10_000,  
      },  
      (error, stdout) => {  
        if (error) {  
          if (error.code !== 1) {  
            console.log(error);  
          }  
          resolve([]);  
        } else {  
          resolve(stdout.split('\n').filter(Boolean));  
        }  
      }  
    );  
  });  
}  
  
//mac电脑的签名  
let alreadSign = false;  
async function macSignature() {//...}
```

#### 4.1.2、GlobTool 工具实现

实现 GlobTool 工具对象的时候，相应的工具参数的定义：

* • pattern：Glob 匹配模式
* • path：搜索路径

```
export const TOOL_NAME_FOR_PROMPT = 'GlobTool';  
export const DESCRIPTION = `- 适用于任何代码库大小的快速文件模式匹配工具  
- 支持像 "**/*.js" 或 "src/**/*.ts" 这样的 glob 模式  
- 返回按修改时间排序的匹配文件路径  
- 当你需要通过名称模式查找文件时使用此工具  
- 当你进行可能需要多轮 glob 和 grep 的开放式搜索时，请改用 Agent 工具  
`;  
/**  
 * GlobTool 参数定义  
 */  
export interface GlobToolArgs {  
  /** Glob 匹配模式 */  
  pattern: string;  
  /** 搜索路径（目录） */  
  path?: string;  
}  
  
/**  
 * GlobTool 返回结果  
 */  
export interface GlobToolResult {  
  /** 匹配的文件路径 */  
  files: string[];  
  /** 是否被截断 */  
  truncated: boolean;  
  /** 匹配总数 */  
  count: number;  
}  
  
/**  
 * GlobTool 处理函数  
 */  
const globToolHandler = async (  
  args: GlobToolArgs,  
  context?: InternalToolContext  
): Promise<GlobToolResult> => {  
  // 执行 glob 搜索  
  const result = await glob(  
    pattern,  
    path,  
    { limit: 100, offset: 0 },  
    abortSignal  
  );  
  
  return {  
    files: result.files,  
    truncated: result.truncated,  
    count: result.files.length,  
  };  
};  
  
/**  
 * GlobTool 工具定义  
 */  
export const GlobTool: InternalTool<GlobToolArgs, GlobToolResult> = {  
  name: 'glob_search',  
  category: 'search',  
  internal: true,  
  description: DESCRIPTION,  
  version: '1.0.0',  
  parameters: {}  
  required: ['pattern'],  
  },  
  handler: globToolHandler,  
};
```

这个命令的实现相对来说比较简单，使用 glob 框架就可以搞定

```
import { glob as globLib } from 'glob';  
  
//匹配搜索文件目录下的文件  
export async function glob(  
  filePattern: string,  
  cwd: string,  
  { limit, offset }: { limit: number; offset: number },  
  abortSignal: AbortSignal  
) {  
  //....  
}
```

#### 4.1.3、FileReadTool工具实现

实现这个文件内容读取的工具对象，增加了一个类似于分页读取内容的功能，避免一次读取太多，相应的工具参数定义：

* • file\_path：文件路径
* • offset：起始行号
* • limit：最大读取行数

```
const MAX_LINES_TO_READ = 2000;  
const MAX_LINE_LENGTH = 2000;  
  
export const DESCRIPTION = '从本地文件系统读取文件。';  
export const PROMPT = `从本地文件系统读取文件。file_path 参数必须是绝对路径，而不是相对路径。默认情况下，它从文件开头读取最多 ${MAX_LINES_TO_READ} 行。你可以选择指定行偏移量和限制（对于长文件特别有用），但建议通过不提供这些参数来读取整个文件。任何超过 ${MAX_LINE_LENGTH} 个字符的行将被截断。对于图像文件，该工具将为你显示图像。`;  
  
/**  
 * FileReadTool 参数定义  
 */  
export interface FileReadToolArgs {  
  /** 文件路径 */  
  file_path: string;  
  /** 起始行号（从0开始） */  
  offset?: number;  
  /** 最大读取行数 */  
  limit?: number;  
}  
  
/**  
 * FileReadTool 返回结果  
 */  
export interface FileReadToolResult {  
  /** 文件内容 */  
  content: string;  
  /** 返回的行数 */  
  lineCount: number;  
  /** 文件总行数 */  
  totalLines: number;  
  /** 文件路径 */  
  filePath: string;  
}  
  
/**  
 * FileReadTool 处理函数  
 */  
const fileReadToolHandler = async (  
  args: FileReadToolArgs,  
  context?: InternalToolContext  
): Promise<FileReadToolResult> => {  
  //...  
};  
  
/**  
 * FileReadTool 工具定义  
 */  
export const FileReadTool: InternalTool<FileReadToolArgs, FileReadToolResult> =  
  {  
    name: 'read_file',  
    category: 'filesystem',  
    internal: true,  
    description: DESCRIPTION,  
    version: '1.0.0',  
    parameters: {  
      //...  
    },  
      required: ['file_path'],  
    },  
    handler: fileReadToolHandler,  
  };
```

这个文件内容读取工具使用的是 Node 的 fs 模块中的文件读取命令，需要重点注意的是文件编码格式要获取，不一定文件编码格式都是 utf-8

```
//读取文件内容  
export async function readFileContent(  
  filePath: string,  
  offset: number = 0,  
  maxLines?: number  
): Promise<{ content: string; lineCount: number; totalLines: number }> {  
  //...  
}
```

#### 4.1.4、ListDirectoryTool 工具实现

该工具是文件目录结构查询，工具函数的参数只有一个：

* • path：要列出目录的路径

```
export const DESCRIPTION =  
  '列出指定路径中的文件和目录。path 参数必须是绝对路径，而不是相对路径。如果你知道要搜索哪些目录，通常应优先使用 Glob 和 Grep 工具。';  
  
/**  
 * ListDirectoryTool 参数定义  
 */  
export interface ListDirectoryToolArgs {  
  /** 要列出的目录路径 */  
  path?: string;  
}  
  
/**  
 * ListDirectoryTool 返回结果  
 */  
export interface ListDirectoryToolResult {  
  /** 文件树格式输出 */  
  tree: string;  
  /** 文件路径列表 */  
  files: string[];  
  /** 总数 */  
  count: number;  
}  
  
/**  
 * ListDirectoryTool 处理函数  
 */  
const listDirectoryToolHandler = async (  
  args: ListDirectoryToolArgs,  
  context?: InternalToolContext  
): Promise<ListDirectoryToolResult> => {  
  const files = listDirectory(path, context?.cwd || process.cwd(), abortSignal);  
  // 创建文件树  
  const tree = createFileTree(files);  
  const treeOutput = printTree(tree);  
  return {  
    tree: treeOutput,  
    files,  
    count: files.length,  
  };  
};  
  
/**  
 * ListDirectoryTool 工具定义  
 */  
export const ListDirectoryTool: InternalTool<  
  ListDirectoryToolArgs,  
  ListDirectoryToolResult  
> = {  
  name: 'list_directory',  
  category: 'filesystem',  
  internal: true,  
  description: DESCRIPTION,  
  version: '1.0.0',  
  parameters: {  
    //...  
  },  
  handler: listDirectoryToolHandler,  
};
```

在实现该工具函数的中，没有使用递归函数去实现，担心栈溢出，使用的是一个广度搜索，需要注意的是两个格式化的函数

* • 将搜索出来的文件数组转换为 Tree 格式的数据结构
* • 将 Tree 格式的数据结构转换为空格隔开的目录字符串

```
export function listDirectory(  
  initialPath: string,  
  cwd: string,  
  abortSignal: AbortSignal  
): string[] {  
  //...  
}  
  
  
/**  
 * 为了让结果更加可读和节省Token，要创建两个对于结果进行格式化的函数  
 *  - createFileTree  
 *  - printTree  
 */  
  
type TreeNode = {  
  name: string;  
  path: string;  
  type: 'file' | 'directory';  
  children?: TreeNode[];  
};  
export function createFileTree(sortedPaths) {  
  //...  
}  
  
/**  
 * eg.  
 * - src/  
 *   - index.ts  
 *   - utils/  
 *     - file.ts  
 * @param tree  
 * @param level  
 * @param prefix  
 * @returns  
 */  
export function printTree(tree: TreeNode[], level = 0, prefix = ''): string {  
  //...  
}
```

### 4.2、工具管理

我们定义好这些工具之后，需要将这个工具提供给 LLM，并且 LLM 返回工具执行命令的时候，需要调用相应工具的函数执行，最后将结果返回给 LLM，所以这里需要创建一个工具管理器

我的实现思路：

* • 提供一个方法用来注册工具函数：`registerAllTools`
* • 提供一个方法用来获取注册好的工具函数：`getTools`
* • 提供一个工具执行函数：`execute`

```
const toolsList = [GrepTool, GlobTool, ListDirectoryTool, FileReadTool];  
  
/**  
 * 工具管理类  
 * 负责工具的注册、查询和执行  
 */  
export class ToolManager {  
  private tools: Map<string, InternalTool> = new Map();  
  
  constructor() {  
    // 在构造函数中自动注册所有工具  
    this.registerAllTools();  
  }  
  
  /**  
   * 注册所有工具  
   * 显式列出所有要注册的工具  
   */  
  private registerAllTools() {  
    const tools = [...toolsList];  
  
    tools.forEach(tool => {  
      this.tools.set(tool.name, tool);  
    });  
  }  
  
  /**  
   * 执行指定工具  
   * @param name 工具名称  
   * @param args 工具参数  
   * @param context 可选的上下文参数（如 abortSignal）  
   * @returns 工具执行结果  
   */  
  async execute<TArgs = any, TResult = any>(  
    name: string,  
    args: TArgs,  
    context?: InternalToolContext  
  ): Promise<TResult> {  
    const tool = this.tools.get(name);  
  
    if (!tool) {  
      throw new Error(  
        `Tool '${name}' not found. Available tools: ${this.getToolNames().join(', ')}`  
      );  
    }  
  
    try {  
      const result = await tool.handler(args, context);  
      return result;  
    } catch (error) {  
      console.error(`Tool '${name}' execution failed:`, error);  
      throw error;  
    }  
  }  
  
  /**  
   * 获取所有已注册的工具  
   * @returns 工具数组  
   */  
  getTools(): InternalTool[] {  
    return Array.from(this.tools.values());  
  }  
}
```

在使用的时候，因为供应商的不同，对于 api 中的 tool 参数的格式也是不一样的，所以这个需要在定义 LLM 类的时候，自己提供一个工具格式转换方法，例如 Openai 的参数格式是

```
  /**  
   * Format tools for OpenAI API format  
   * Converts ToolSet to array of OpenAI tool definitions  
   */  
  private formatToolsForAPI(tools: ToolSet): OpenAITool[] {  
    return Object.values(tools).map(tool => ({  
      type: 'function' as const,  
      function: {  
        name: tool.name,  
        description: tool.description || '',  
        parameters: tool.parameters || {  
          type: 'object' as const,  
          properties: {},  
        },  
      },  
    }));  
  }
```

### 4.3、LLM 工具调用

LLM 工具调用

关于 LLM 的调用中，最核心的思路是：**LLM 循环执行并且判断，并且设置最大的循环数，不要陷入死循环**

```
  async generate(  
    userInput: string,  
    imageData?: any,  
    stream: boolean = false  
  ): Promise<string> {  
    try {  
      // Build messages array  
      const messages = this.formatMessages(userInput);  
  
      // Get tools if available  
      const tools = await this.getAllTools();  
      const hasTools = tools && Object.keys(tools).length > 0;  
  
      // Tool calling loop  
      let iteration = 0;  
      let finalResponse = '';  
  
      while (iteration < this.maxIterations) {  
        // Make API call  
        const response = await this.openai.chat.completions.create({  
          model: this.model,  
          messages,  
          stream,  
          ...(hasTools && { tools: this.formatToolsForAPI(tools) }),  
        });  
          
        //...  
  
        const message = response.choices[0]?.message;  
        const finishReason = response.choices[0]?.finish_reason;  
  
        // Check if model wants to call tools  
        if (finishReason === 'tool_calls' && message?.tool_calls) {  
          // Add assistant message with tool_calls to context  
          messages.push({  
            role: 'assistant',  
            content: message.content || '',  
            tool_calls: message.tool_calls,  
          });  
  
          // Execute each tool call  
          for (const toolCall of message.tool_calls) {  
            const toolName = toolCall.function.name;  
            const toolArgs = JSON.parse(toolCall.function.arguments);  
            try {  
              // Execute tool using tool manager  
              const toolResult = await this.toolManager?.execute(  
                toolName,  
                toolArgs  
              );  
              // Add tool result to messages  
              messages.push({  
                role: 'tool',  
                tool_call_id: toolCall.id,  
                content: JSON.stringify(toolResult),  
              });  
            } catch (error) {}  
          }  
  
          // Continue loop to send tool results back to model  
          continue;  
        }  
  
        // No more tool calls, return final response  
        finalResponse = message?.content || '';  
        break;  
      }  
  
      if (iteration >= this.maxIterations) {  
        //...  
      }  
  
      return finalResponse;  
    } catch (error) {  
      this.handleError(error, 'DeepseekService.generate');  
    }  
  }
```

## 五、最后

在执行这个搜索代理的时候，我更换了不同的模式进行测试，并且打印出来了搜索策略，我有一些发现可以分享给大家  
我简单测试了这几种模型：DeepSeekV3.2、Qwen3、Glm-4.6、Gemini-2.5Pro、Gemini-2.5flash、Claude-sonnet-4.5、GPT-5  
我发现表现最好的确实是 Claude 和 GPT，但是里面有一些细微的差别，我的感觉是

* • Claude 搜索的策略方向非常正确，而且可以感觉到有步骤的在执行，不会过度的去无脑的搜索，像是一位非常聪明的几何图形问题论证家，解一步，验证一步，在搜索一步，就是感觉优雅
* • GPT 的表现也很好，我感觉到它的小心翼翼，是小心翼翼的确定方向，一旦方向确定就会大胆的探索，像是一位稳重成熟的探险家，有谨慎又有强大的执行力

本文节选自我正在整理的 **「上下文工程实践」** 项目，该项目已完整发布在 GitHub 上。  
如果你希望阅读更多相关的章节与案例，可以前往项目仓库查看：

> 👉 https://github.com/WakeUp-Jin/Practical-Guide-to-Context-Engineering