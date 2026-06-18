---
title: Cursor 进阶配置指南：Rules/Skills/MCP/Hooks/Commands/Modes 详解
author: 汐刻
date: 
url: https://mp.weixin.qq.com/s?__biz=MzU2NzgxMjY2Nw==&mid=2247483739&idx=1&sn=26cc7342ea42b0627e0162e68b1ea5c8&chksm=fd687eeadb3c307f4d4bdcd5f51a9f8675514314072ce9b0227458e34f5c44131ba3e652b098&mpshare=1&scene=24&srcid=04101pXlEqSN603fDuYEXGKX&sharer_shareinfo=a9f29807d18d3ae2225511e152570894&sharer_shareinfo_first=a9f29807d18d3ae2225511e152570894#rd
---

# Cursor 进阶配置指南

> 深入掌握 Cursor AI 编辑器的核心高级功能，打造个性化开发工作流

## 核心特性速览

| 特性 | 说明 | 快捷键 |
| --- | --- | --- |
| AI Chat | 侧边栏对话，理解项目上下文 | `Cmd+L` |
| 行内编辑 | 直接生成/修改代码 | `Cmd+K` |
| Composer | 多文件协同编辑 | `Cmd+I` |
| Tab 补全 | 智能代码补全 | `Tab` |
| @ 引用 | 精准引用文件、代码 | `@` |

---

## 一、Rules：自定义 AI 行为规则

Rules 是 Cursor 最强大的功能，允许你定义 AI 的行为规范和代码风格。

### Rules 文件位置

```
项目根目录/  
├── .cursor/  
│   └── rules/  
│       ├── default.md          # 默认规则  
│       ├── java.md             # 语言特定规则  
│       └── project-specific.md # 项目特定规则
```

### 完整示例：Java 项目 Rules

创建 `.cursor/rules/java.md`：

```
# Java 项目开发规范  
  
## 项目信息  
- 框架：Spring Boot 3.2  
- Java 版本：17  
- 构建工具：Maven  
  
## 代码规范  
  
### 命名规范  
- 类名：PascalCase (UserService)  
- 方法名：camelCase (getUserById)  
- 常量：UPPER_SNAKE_CASE  
  
### 必选实践  
1. 所有实体类使用 Lombok  
2. Controller 层只做参数校验和响应封装  
3. Service 层包含业务逻辑和事务管理  
4. 使用 Optional 处理可能为空的返回值  
  
### API 设计规范
```

GET    /api/v1/users          # 获取用户列表
POST   /api/v1/users          # 创建用户
PUT    /api/v1/users/{id}     # 更新用户
DELETE /api/v1/users/{id}     # 删除用户

```
### 响应格式  
```json  
{  
  "code": 200,  
  "message": "success",  
  "data": { ... }  
}
```

```
### Rules 最佳实践  
  
✅ **推荐：**  
- 规则要具体明确  
- 包含正反示例  
- 定期更新维护  
  
❌ **避免：**  
- 规则过于笼统  
- 规则互相冲突  
  
---  
  
## 二、MCP：模型上下文协议  
  
MCP 允许 Cursor 与外部工具和服务安全交互。  
  
### 配置 MCP Servers  
  
创建 `~/.cursor/mcp.json`：  
  
```json  
{  
  "mcpServers": {  
    "filesystem": {  
      "command": "npx",  
      "args": [  
        "-y",  
        "@modelcontextprotocol/server-filesystem",  
        "/Users/username/projects"  
      ]  
    },  
    "github": {  
      "command": "npx",  
      "args": [  
        "-y",  
        "@modelcontextprotocol/server-github"  
      ],  
      "env": {  
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"  
      }  
    },  
    "postgres": {  
      "command": "npx",  
      "args": [  
        "-y",  
        "@modelcontextprotocol/server-postgres",  
        "postgresql://localhost:5432/mydb"  
      ]  
    }  
  }  
}
```

### 常用 MCP Servers

| Server | 用途 |
| --- | --- |
| filesystem | 文件操作 |
| github | GitHub API |
| postgres | PostgreSQL 数据库 |
| git | Git 操作 |
| fetch | 网页抓取 |

### 在 Cursor 中使用 MCP

```
使用 @filesystem 读取当前目录的 package.json
```

```
使用 @github 创建一个新的 issue
```

---

## 三、Commands：自定义命令

### 内置 Commands

| 命令 | 功能 |
| --- | --- |
| `/help` | 显示帮助 |
| `/clear` | 清除聊天历史 |
| `/search` | 搜索代码 |
| `/problems` | 查看问题 |

### 创建自定义 Commands

创建 `~/.cursor/commands.json`：

```
{  
  "commands": {  
    "review": {  
      "description": "代码审查",  
      "prompt": "请审查当前文件，检查：1. 代码规范 2. 潜在 bug 3. 性能问题",  
      "context": ["currentFile"]  
    },  
    "test": {  
      "description": "生成测试",  
      "prompt": "为当前选中的函数生成完整的单元测试",  
      "context": ["selection"]  
    },  
    "doc": {  
      "description": "生成文档",  
      "prompt": "为当前文件生成完整的 API 文档",  
      "context": ["currentFile"]  
    }  
  }  
}
```

### 使用自定义 Commands

```
/review
```

```
/test
```

---

## 四、Modes：工作模式切换

### 内置 Modes

| Mode | 用途 | 特点 |
| --- | --- | --- |
| Ask | 快速问答 | 不修改代码 |
| Edit | 代码编辑 | 直接修改代码 |
| Agent | 自主代理 | 执行多步任务 |

### 自定义 Modes

创建 `~/.cursor/modes.json`：

```
{  
  "modes": {  
    "review": {  
      "name": "Review",  
      "description": "代码审查",  
      "model": "claude-3.5-sonnet",  
      "systemPrompt": "你是一个严格的代码审查员。检查代码规范、潜在 bug、性能问题。",  
      "tools": ["search", "readFile"]  
    },  
    "learn": {  
      "name": "Learn",  
      "description": "学习模式",  
      "systemPrompt": "你是一个耐心的编程教师。解释概念时要详细，提供多个示例。",  
      "tools": ["search", "readFile"]  
    }  
  }  
}
```

### 模式切换

```
/mode ask  
/mode edit  
/mode agent  
/mode review
```

---

## 五、综合实战案例

### 案例 1：从零创建 Spring Boot 项目

**步骤 1：** 使用 Agent 模式初始化

```
/mode agent  
创建一个 Spring Boot 3.2 项目，包含完整的分层架构和 Swagger 文档。
```

**步骤 2：** 配置 Rules

创建 `.cursor/rules/spring-boot.md`

**步骤 3：** 使用 Composer 创建用户模块

```
/mode edit  
创建用户管理模块：Entity、Repository、Service、Controller、DTO
```

**步骤 4：** 生成测试

```
/test  
为 UserService 生成完整的单元测试。
```

### 案例 2：遗留代码重构

**步骤 1：** 使用 Review 模式分析

```
/mode review  
分析这个 500 行的方法，识别可提取的子功能和潜在 bug。
```

**步骤 2：** 使用 Agent 模式重构

```
/mode agent  
根据审查结果重构：提取子方法、添加参数校验和日志。
```

---

## 效率对比

| 任务 | 传统方式 | 使用 Cursor | 提升 |
| --- | --- | --- | --- |
| 项目初始化 | 60 分钟 | 5 分钟 | 12 倍 |
| CRUD 开发 | 120 分钟 | 20 分钟 | 6 倍 |
| 单元测试 | 90 分钟 | 15 分钟 | 6 倍 |
| 代码审查 | 60 分钟 | 10 分钟 | 6 倍 |

**平均效率提升：5-10 倍** 🚀

---

## 避坑指南

### ⚠️ 不要盲目信任 AI

* ✅ 审查生成的代码，特别是安全相关逻辑
* ✅ 理解 AI 的解释，不要直接复制
* ✅ 关键业务逻辑要人工复核

### ⚠️ 上下文限制

* Cursor 的上下文窗口有限（约 100K tokens）
* 大项目建议分模块处理
* 使用 `@` 引用精准定位

---

## 结语

Cursor 不仅仅是一个编辑器，它是一个 **AI 增强的开发平台**。通过合理配置 Rules、MCP、Commands 和 Modes，你可以打造属于自己的超级开发环境。

**开始行动：**

1. 从配置 Rules 开始，定义你的代码规范
2. 尝试使用 Composer 进行多文件编辑
3. 配置 MCP 连接外部工具
4. 建立适合你的工作流

---

*觉得有用？分享给你的团队，一起提升开发效率！*

*最后更新：2026 年 3 月*