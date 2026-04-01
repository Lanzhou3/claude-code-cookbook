# 第22章 构建 MCP 服务器

> 本章详细介绍如何构建自定义的 MCP 服务器，包括协议实现、工具定义、传输层配置和部署。

---

## 22.1 MCP 服务器概述

### 22.1.1 什么是 MCP 服务器

MCP 服务器是提供工具、资源和提示的外部服务，Claude Code 通过 MCP 协议与其通信：

```
┌─────────────────────────────────────────────────────────────┐
│                    MCP 服务器架构                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   Claude Code ──── JSON-RPC ───→ MCP Server                │
│      │                                    │                 │
│      │                              ┌─────┴─────┐          │
│      │                              │           │          │
│      │                         ┌────────┐ ┌────────┐      │
│      │                         │ Tools  │ │Resources│      │
│      │                         └────────┘ └────────┘      │
│      │                                    │                 │
│      ◄────────────────────────────────────┘                 │
│                     结果返回                                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 22.1.2 MCP 服务器 vs 插件

| 特性 | MCP 服务器 | Claude Code 插件 |
|------|------------|------------------|
| **运行位置** | 独立进程 | Claude Code 内部 |
| **技术栈** | 任意语言 | TypeScript |
| **通信** | JSON-RPC | 直接函数调用 |
| **部署** | 独立部署 | 随 Claude Code |
| **适用场景** | 外部服务、跨语言 | 内部扩展 |

---

## 22.2 项目初始化

### 22.2.1 创建项目

```bash
# 创建项目目录
mkdir my-mcp-server && cd my-mcp-server

# 初始化 npm 项目
npm init -y

# 安装 MCP SDK
npm install @modelcontextprotocol/sdk

# 安装 TypeScript
npm install -D typescript @types/node
```

### 22.2.2 TypeScript 配置

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "lib": ["ES2022"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

### 22.2.3 项目结构

```
my-mcp-server/
├── src/
│   ├── index.ts           # 入口
│   ├── server.ts          # 服务器实现
│   ├── tools/             # 工具定义
│   │   └── index.ts
│   └── resources/          # 资源定义
│       └── index.ts
├── package.json
├── tsconfig.json
└── README.md
```

---

## 22.3 服务器实现

### 22.3.1 基本服务器

```typescript
// src/index.ts
import { Server } from '@modelcontextprotocol/sdk/server/index.js'
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js'
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
  ListResourcesRequestSchema
} from '@modelcontextprotocol/sdk/types.js'
import { MyTools } from './tools/index.js'

class MyMCPServer {
  private server: Server

  constructor() {
    this.server = new Server(
      {
        name: 'my-mcp-server',
        version: '1.0.0'
      },
      {
        capabilities: {
          tools: {},
          resources: {}
        }
      }
    )

    this.setupHandlers()
  }

  private setupHandlers(): void {
    // 工具列表处理器
    this.server.setRequestHandler(ListToolsRequestSchema, async () => {
      return {
        tools: MyTools.list()
      }
    })

    // 工具调用处理器
    this.server.setRequestHandler(CallToolRequestSchema, async (request) => {
      const { name, arguments: args } = request.params

      const tool = MyTools.get(name)
      if (!tool) {
        throw new Error(`Tool ${name} not found`)
      }

      return await tool.execute(args)
    })

    // 资源列表处理器
    this.server.setRequestHandler(ListResourcesRequestSchema, async () => {
      return {
        resources: MyResources.list()
      }
    })
  }

  async run(): Promise<void> {
    const transport = new StdioServerTransport()
    await this.server.connect(transport)
    console.error('MCP Server running on stdio')
  }
}

const server = new MyMCPServer()
server.run().catch(console.error)
```

### 22.3.2 工具定义

```typescript
// src/tools/index.ts
import { z } from 'zod'

export const MyTools = {
  list(): Tool[] {
    return [
      {
        name: 'greet',
        description: '生成问候语',
        inputSchema: {
          type: 'object',
          properties: {
            name: {
              type: 'string',
              description: '要问候的人名'
            },
            language: {
              type: 'string',
              enum: ['zh', 'en', 'ja'],
              description: '语言',
              default: 'zh'
            }
          },
          required: ['name']
        }
      },
      {
        name: 'calculate',
        description: '执行简单计算',
        inputSchema: {
          type: 'object',
          properties: {
            expression: {
              type: 'string',
              description: '数学表达式'
            }
          },
          required: ['expression']
        }
      }
    ]
  },

  get(name: string): Tool | undefined {
    const tools = this.list()
    return tools.find(t => t.name === name)
  }
}
```

---

## 22.4 工具执行

### 22.4.1 工具执行器

```typescript
// src/tools/executor.ts
export async function executeTool(
  name: string,
  args: Record<string, unknown>
): Promise<ToolResult> {
  switch (name) {
    case 'greet':
      return executeGreet(args as GreetArgs)
    case 'calculate':
      return executeCalculate(args as CalculateArgs)
    default:
      throw new Error(`Unknown tool: ${name}`)
  }
}

interface GreetArgs {
  name: string
  language?: 'zh' | 'en' | 'ja'
}

async function executeGreet(args: GreetArgs): Promise<ToolResult> {
  const { name, language = 'zh' } = args

  const greetings: Record<string, string> = {
    zh: `你好，${name}！`,
    en: `Hello, ${name}!`,
    ja: `${name}さん、こんにちは！`
  }

  return {
    content: [
      {
        type: 'text',
        text: greetings[language]
      }
    ]
  }
}

interface CalculateArgs {
  expression: string
}

async function executeCalculate(args: CalculateArgs): Promise<ToolResult> {
  // 安全计算（不使用 eval）
  const result = safeEvaluate(args.expression)

  return {
    content: [
      {
        type: 'text',
        text: `结果: ${result}`
      }
    ]
  }
}
```

### 22.4.2 Zod 验证

```typescript
import { z } from 'zod'

const GreetArgsSchema = z.object({
  name: z.string().min(1),
  language: z.enum(['zh', 'en', 'ja']).optional().default('zh')
})

function executeGreet(args: unknown): ToolResult {
  const { name, language } = GreetArgsSchema.parse(args)

  // 执行...
}
```

---

## 22.5 资源管理

### 22.5.1 资源定义

```typescript
// src/resources/index.ts
import { z } from 'zod'

export const MyResources = {
  list(): Resource[] {
    return [
      {
        uri: 'config://app/version',
        name: 'App Version',
        description: '当前应用版本',
        mimeType: 'text/plain'
      },
      {
        uri: 'config://app/settings',
        name: 'App Settings',
        description: '应用配置',
        mimeType: 'application/json'
      }
    ]
  },

  async read(uri: string): Promise<ResourceContent> {
    if (uri === 'config://app/version') {
      return {
        uri,
        mimeType: 'text/plain',
        text: '1.0.0'
      }
    }

    if (uri === 'config://app/settings') {
      return {
        uri,
        mimeType: 'application/json',
        text: JSON.stringify({
          theme: 'dark',
          language: 'zh-CN',
          debug: false
        }, null, 2)
      }
    }

    throw new Error(`Resource not found: ${uri}`)
  }
}
```

---

## 22.6 传输层配置

### 22.6.1 Stdio 传输

默认使用 stdio 传输：

```typescript
// src/index.ts
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js'

const transport = new StdioServerTransport()
await server.connect(transport)
```

### 22.6.2 HTTP/SSE 传输

```typescript
// HTTP/SSE 传输（用于远程服务器）
import { SSEServerTransport } from '@modelcontextprotocol/sdk/server/sse.js'

const transport = new SSEServerTransport({
  endpoint: '/mcp'
})

await server.connect(transport)

app.use('/mcp', (req, res) => {
  transport.handleRequest(req, res, {
    sessionId: req.query.sessionId
  })
})
```

### 22.6.3 WebSocket 传输

```typescript
// WebSocket 传输
import { WebSocketServerTransport } from '@modelcontextprotocol/sdk/server/websocket.js'

const wss = new WebSocketServer({
  port: 8080
})

wss.on('connection', async (ws) => {
  const transport = new WebSocketServerTransport({
    sessionIdGenerator: () => randomUUID()
  })

  await server.connect(transport)
  transport.handleWebSocket(ws)
})
```

---

## 22.7 认证与安全

### 22.7.1 API Key 认证

```typescript
import crypto from 'crypto'

class SecureMCPServer extends MCPServer {
  private validApiKeys: Set<string>

  constructor(apiKeys: string[]) {
    super()
    this.validApiKeys = new Set(apiKeys)
  }

  async handleRequest(request: Request): Promise<Response> {
    const apiKey = request.headers.get('x-api-key')

    if (!this.validApiKeys.has(apiKey)) {
      return new Response('Unauthorized', { status: 401 })
    }

    return super.handleRequest(request)
  }
}
```

### 22.7.2 OAuth 2.0

```typescript
// OAuth 认证
class OAuthMCPServer extends MCPServer {
  private oauthConfig: OAuthConfig

  async handleRequest(request: Request): Promise<Response> {
    const token = request.headers.get('Authorization')?.split(' ')[1]

    try {
      const claims = await this.verifyToken(token)
      request.context.user = claims
    } catch {
      return new Response('Unauthorized', { status: 401 })
    }

    return super.handleRequest(request)
  }
}
```

---

## 22.8 测试

### 22.8.1 本地测试

```typescript
// test/server.test.ts
import { Server } from '@modelcontextprotocol/sdk/server/index.js'
import { Client } from '@modelcontextprotocol/sdk/client/index.js'
import { StdioClientTransport } from '@modelcontextprotocol/sdk/client/stdio.js'

async function testServer() {
  // 启动服务器
  const serverProcess = spawn('node', ['dist/index.js'])

  // 创建客户端
  const transport = new StdioClientTransport(serverProcess.stdin, serverProcess.stdout)
  const client = new Client({ name: 'test', version: '1.0.0' }, {})

  await client.connect(transport)

  // 测试工具调用
  const result = await client.callTool({
    name: 'greet',
    arguments: { name: 'World', language: 'en' }
  })

  console.log('Result:', result)

  // 清理
  await client.close()
  serverProcess.kill()
}
```

### 22.8.2 Claude Code 集成测试

```typescript
// 测试 MCP 服务器配置
{
  "mcpServers": [
    {
      "name": "test-server",
      "command": "node",
      "args": ["/path/to/dist/index.js"]
    }
  ]
}
```

---

## 22.9 部署

### 22.9.1 编译

```typescript
// build.js
import * as esbuild from 'esbuild'

await esbuild.build({
  entryPoints: ['src/index.ts'],
  bundle: true,
  platform: 'node',
  target: 'node18',
  outfile: 'dist/index.js',
  external: ['@modelcontextprotocol/sdk']
})
```

### 22.9.2 Docker 部署

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY dist ./dist/

CMD ["node", "dist/index.js"]
```

### 22.9.3 npm 发布

```json
// package.json
{
  "name": "@my-org/my-mcp-server",
  "version": "1.0.0",
  "main": "dist/index.js",
  "bin": {
    "my-mcp-server": "dist/index.js"
  },
  "scripts": {
    "build": "node build.js",
    "prepublish": "npm run build"
  }
}
```

---

## 22.10 配置到 Claude Code

### 22.10.1 本地配置

```json
// ~/.claude/settings.json
{
  "mcpServers": [
    {
      "name": "my-server",
      "command": "node",
      "args": ["/path/to/my-mcp-server/dist/index.js"]
    }
  ]
}
```

### 22.10.2 远程配置

```json
{
  "mcpServers": [
    {
      "name": "my-remote-server",
      "transport": "sse",
      "url": "https://api.example.com/mcp",
      "headers": {
        "Authorization": "Bearer ${API_TOKEN}"
      }
    }
  ]
}
```

---

## 22.11 本章小结

本章介绍了 MCP 服务器构建：

| 主题 | 核心要点 |
|------|----------|
| **概述** | MCP 服务器定义、与插件对比 |
| **项目初始化** | 项目创建、TypeScript 配置 |
| **服务器实现** | 基本服务器、处理器设置 |
| **工具定义** | 工具列表、执行器 |
| **资源管理** | 资源定义、读取 |
| **传输层** | Stdio、HTTP/SSE、WebSocket |
| **认证安全** | API Key、OAuth |
| **测试** | 本地测试、集成测试 |
| **部署** | 编译、Docker、npm |
| **配置** | 本地配置、远程配置 |

---

## 下章预告

下一章我们将学习 **贡献与二次开发**，包括如何为 Claude Code 贡献代码和进行二次开发。

---

*© 本书由 Claude Code 辅助撰写*
