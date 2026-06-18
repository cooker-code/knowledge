> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020202_MCP/020202_核心知识点/MCP生产接入与治理边界|MCP生产接入与治理边界]]
---
title: TS实现MCP服务
author: YugeCode
date:
url: https://mp.weixin.qq.com/s?__biz=MzAxODg4NzM4NQ==&mid=2247483879&idx=1&sn=07b9441a06cb61fbb226573eb7a9033a&chksm=9adf1d34b77ad763f6f0c6f742e1a08d31238a0ef6ce88396c06e20448ebb2642183c434264a&mpshare=1&scene=24&srcid=0925vn4q4lgIXpPOhDfEsgpU&sharer_shareinfo=f86124d698176b7384a89a73daec7a84&sharer_shareinfo_first=f86124d698176b7384a89a73daec7a84#rd
---

Model Context Protocol，模型上下文协议，简称MCP。

模型上下文协议 (MCP) 是一种开放协议，支持 LLM 应用程序与外部数据源和工具之间的无缝集成。无论您是构建 AI 驱动的 IDE、增强聊天界面，还是创建自定义 AI 工作流，MCP 都提供了一种标准化的方式，将 LLM 与其所需的上下文连接起来。

SDK地址：https://github.com/modelcontextprotocol/typescript-sdk

MCP支持三种模式，分别是：

1.标准的输入/输出(stdio)

·标准输入/输出（Standard Input/Output，简称 stdio）是 UNIX/Linux 系统和编程语言（如 C、Python）中的核心 I/O（输入/输出）模型，定义了程序与外部环境（终端、文件、管道等）交互的基本方式。它由三个默认的文件描述符组成（stdin、stdout、stderr）。

适用场景：面向传统命令行式交互，强调顺序性与确定性，通常用于命令行工具、自动化脚本等短期交互场景。它是最基础的数据传输方式，适合单次的、低延迟的交互任务，适合本地开发与调试。

2.SSE模式

· SSE（Server-Sent Events，服务器推送事件）是一种基于 HTTP 的轻量级协议，允许服务器主动向客户端（如 Web 浏览器）推送实时数据。

适用场景：它适用于需要 \*\*单向实时通信\*\*（服务器 → 客户端）的场景，如新闻更新、社交媒体通知、股票行情等。

3.Streamable HTTP模式

· Streamable HTTP，是一种基于标准 HTTP 协议的数据传输方式，允许服务器在完全生成响应之前就开始逐步向客户端发送数据流。

适用场景：这种方式适用于需要实时性、大文件传输或动态生成内容的场景，例如视频流、日志下载、实时数据 API 等。

以下将以一个简单的TS示例实现三种方式的MCP的服务端。

分别是：runSSEServer、runStdioServer、runStreamableHTTPServer

```
#!/usr/bin/env node  import express, {Request, Response} from 'express';  import {SSEServerTransport} from '@modelcontextprotocol/sdk/server/sse.js';  import {StdioServerTransport} from '@modelcontextprotocol/sdk/server/stdio.js';  import {StreamableHTTPServerTransport} from "@modelcontextprotocol/sdk/server/streamableHttp.js";  import {z} from "zod";  import {McpServer} from '@modelcontextprotocol/sdk/server/mcp.js';  import {randomUUID} from "node:crypto";  import {isInitializeRequest} from "@modelcontextprotocol/sdk/types.js"  

export const getServer = () => {  
    const server = new McpServer({          name: 'test_mcp',          version: '1.0.0',      }, {capabilities: {logging: {}}});  
    server.tool(          "test_add",          "实现两个数的相加",          {              first_num: z.number().describe("第一个数，必填"),              second_num: z.number().describe("第二个数，必填"),          },          async ({first_num, second_num}) => {              const final_num = first_num + second_num              return {                  content: [{type: "text", text: JSON.stringify(final_num, null, 2)}]              };  
        }      )  
    return server;  };  
// 使用sse的方式运行  async function runSSEServer() {      const app = express();      app.use(express.json());  
    const transports: Record<string, SSEServerTransport> = {};  
    app.get('/sse', async (req: Request, res: Response) => {          console.log('Received GET request to /sse (establishing SSE stream)')  
        try {              const transport = new SSEServerTransport('/messages', res);              const sessionId = transport.sessionId;              transports[sessionId] = transport;              transport.onclose = () => {                  console.log(`SSE transport closed for session ${sessionId}`);                  delete transports[sessionId];              };  
            const server = getServer();              await server.connect(transport);  
            console.log(`Established SSE stream with session ID: ${sessionId}`);          } catch (error) {              console.error('Error establishing SSE stream:', error);              if (!res.headersSent) {                  res.status(500).send('Error establishing SSE stream');              }          }      });  
    app.post('/messages', async (req: Request, res: Response) => {          const sessionId = req.query.sessionId as string | undefined;          if (!sessionId) {              console.error('No session ID provided in request URL');              res.status(400).send('Missing sessionId parameter');              return;          }          const transport = transports[sessionId];          if (!transport) {              console.error(`No active transport found for session ID: ${sessionId}`);              res.status(404).send('Session not found');              return;          }  
        try {              // 处理消息              await transport.handlePostMessage(req, res, req.body);          } catch (error) {              console.error('Error handling request:', error);              if (!res.headersSent) {                  res.status(500).send('Error handling request');              }          }      });  
    // 启动服务      const SERVER_PORT = process.env.SERVER_PORT || 3001;      app.listen(SERVER_PORT, () => {          console.log(`Simple SSE Server (deprecated protocol version 2024-11-05) listening on port ${SERVER_PORT}`);      });  
    // 处理服务关闭      process.on('SIGINT', async () => {          for (const sessionId in transports) {              try {                  console.log(`Closing transport for session ${sessionId}`);                  await transports[sessionId].close();                  delete transports[sessionId];              } catch (error) {                  console.error(`Error closing transport for session ${sessionId}:`, error);              }          }          console.log('Server shutdown complete');          process.exit(0);      });  }  
// 使用stdio的方式运行  async function runStdioServer() {      try {          const transport = new StdioServerTransport();          const server = getServer();          await server.connect(transport);          console.error("APIMatic Validation MCP Server running on stdio");      } catch (error) {          console.error("Fatal error in main():", error);          process.exit(1);      }  }  
// 使用StreamableHttp的方式运行  async function runStreamableHTTPServer() {      const app = express();      app.use(express.json());  
    const transports: { [sessionId: string]: StreamableHTTPServerTransport } = {};  
    app.post('/mcp', async (req, res) => {          const sessionId = req.headers['mcp-session-id'] as string | undefined;          let transport: StreamableHTTPServerTransport;  
        if (sessionId && transports[sessionId]) {              transport = transports[sessionId];          } else if (!sessionId && isInitializeRequest(req.body)) {              transport = new StreamableHTTPServerTransport({                  sessionIdGenerator: () => randomUUID(),                  onsessioninitialized: (sessionId) => {                      transports[sessionId] = transport;                  },              });  
            transport.onclose = () => {                  if (transport.sessionId) {                      delete transports[transport.sessionId];                  }              };  
            const server = getServer()              await server.connect(transport);          } else {              // Invalid request              res.status(400).json({                  jsonrpc: '2.0',                  error: {                      code: -32000,                      message: 'Bad Request: No valid session ID provided',                  },                  id: null,              });              return;          }  
        // 处理请求          await transport.handleRequest(req, res, req.body);      });  
    const handleSessionRequest = async (req: express.Request, res: express.Response) => {          const sessionId = req.headers['mcp-session-id'] as string | undefined;          if (!sessionId || !transports[sessionId]) {              res.status(400).send('Invalid or missing session ID');              return;          }  
        const transport = transports[sessionId];          await transport.handleRequest(req, res);      };  
    app.get('/mcp', handleSessionRequest);      app.delete('/mcp', handleSessionRequest);      app.listen(3000);  }  
runSSEServer().catch((error) => {      console.error("Fatal error in main():", error);      process.exit(1);  });
```

以上仅仅注册了一个工具，他还支持资源，提示词等等的注册，具体请参考官网。

---

依赖安装：npm install

依赖构建：npm run build

构建产物：dist/cjs/index.js   dist/esm/index.js

依赖包(package.json)

```
{    "name": "test-mcp-server",    "version": "0.1.0",    "description": "A Model Context Protocol server",    "private": true,    "type": "module",    "scripts": {      "build": "npm run build:esm && npm run build:cjs",      "build:esm": "tsc --outDir dist/esm && echo {\"type\": \"module\"} > dist/esm/package.json",      "build:cjs": "tsc --module CommonJS --outDir dist/cjs && echo {\"type\": \"commonjs\"} > dist/cjs/package.json",      "prepack": "npm run build:esm && npm run build:cjs",      "lint": "eslint src/",      "test": "jest",      "start": "npm run server",      "server": "tsx watch --env-file=.env --clear-screen=false src/index.ts",      "client": "tsx src/cli.ts client"    },    "dependencies": {      "@modelcontextprotocol/sdk": "^1.17.1",      "axios": "^1.9.0",      "cors": "^2.8.5",      "dotenv": "^16.5.0",      "express": "^4.18.2",      "tsx": "^4.19.4",      "url": "^0.11.4",      "zod": "^3.24.2"    },    "devDependencies": {      "@types/cors": "^2.8.16",      "@types/express": "^4.17.21",      "@types/node": "^20.11.24",      "nodemon": "^3.0.1",      "typescript": "^5.3.3"    }  }
```

客户端运行配置：

三种：test-mcp-sse、test-mcp-streamablehttp、test-mcp-stdio

注意：stdio需要有node环境的支持，args就是上面的构建产物的js。stdio还配置env环境变量。

```
{  "mcpServers": {    "test-mcp-sse": {      "url": "http://localhost:3001/sse"    },    "test-mcp-streamablehttp": {      "url": "http://localhost:3000/mcp"    },    "test-mcp-stdio": {      "command": "node",      "args": ["D:\\dist\\cjs\\index.js"],      "env": {        "xx": "xx"      }    }  }}
```

---

说明：非前端专业人士，不喜勿喷，仅为了展示TS的MCP服务如何编写，详细使用请参考官方操作。