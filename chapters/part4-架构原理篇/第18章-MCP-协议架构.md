# 第18章 MCP 协议架构

> 本章深入介绍 Model Context Protocol (MCP) 协议在 Claude Code 中的实现，包括协议原理、传输层、消息格式和服务器管理。

---

## 18.1 MCP 协议概述

### 18.1.1 什么是 MCP

MCP (Model Context Protocol) 是一个标准化协议，用于 AI 模型与外部工具和资源之间的通信：

```
┌─────────────────────────────────────────────────────────────┐
│                      MCP 协议示意                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   Claude Code  ──── JSON-RPC ───→  MCP Server              │
│      │                                    │                 │
│      │                                    ▼                 │
│      │                           ┌─────────────────┐       │
│      │                           │    Tools        │       │
│      │                           │    Resources    │       │
│      │                           │    Prompts      │       │
│      │                           └─────────────────┘       │
│      │                                    │                 │
│      ◄────────────────────────────────────┘                 │
│                     返回结果                                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 18.1.2 MCP 优势

| 特性 | 传统集成 | MCP |
|------|----------|-----|
| **标准化** | 每个工具独立 SDK | 统一协议 |
| **可发现** | 手动配置 | 自动发现 |
| **类型安全** | 依赖文档 | JSON Schema |
| **安全** | 直接系统访问 | 沙箱隔离 |
| **可组合** | 难以组合 | 服务可叠加 |

---

## 18.2 MCP 架构

### 18.2.1 架构组件

```
┌─────────────────────────────────────────────────────────────┐
│                      MCP 客户端 (Claude Code)               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │ ListMcp     │    │ MCPClient   │    │ Transport   │     │
│  │ Resources   │ ←→ │ (连接管理)  │ ←→ │ (传输层)    │     │
│  │ ReadMcp     │    │             │    │             │     │
│  │ Resources   │    └─────────────┘    └─────────────┘     │
│  └─────────────┘           │                  │              │
│                            │                  │              │
│                            ▼                  ▼              │
│                   ┌─────────────────┐ ┌────────────────┐   │
│                   │  MCPConnection  │ │ InProcess     │   │
│                   │  Manager        │ │ SdkControl    │   │
│                   │                 │ │ Transport     │   │
│                   └─────────────────┘ └────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────┐
│                      MCP 服务器 (外部)                        │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │ ToolHandler │    │ResourceHandler│  │PromptHandler│     │
│  └─────────────┘    └─────────────┘    └─────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

### 18.2.2 核心文件（实际结构）

```typescript
// services/mcp/ 实际目录结构
services/mcp/
├── client.ts                    // MCP 客户端实现 (119KB - 最大文件)
├── config.ts                    // 配置解析 (51KB)
├── auth.ts                      // 认证处理 (88KB - 第二大文件)
├── types.ts                     // 类型定义
├── MCPConnectionManager.tsx      // React 连接管理器组件
├── useManageMCPConnections.ts   // React hook (45KB)
├── utils.ts                     // 工具函数 (18KB)
├── channelNotification.ts        // 通道通知
├── channelPermissions.ts         // 通道权限
├── channelAllowlist.ts          // 通道白名单
├── claudeai.ts                 // Claude AI MCP 实现
├── elicitationHandler.ts        // Elicitation 处理
├── headersHelper.ts             // 请求头帮助
├── mcpStringUtils.ts            // 字符串工具
├── normalization.ts             // 规范化
├── oauthPort.ts                // OAuth 端口
├── officialRegistry.ts          // 官方注册表
├── vscodeSdkMcp.ts             // VS Code SDK MCP
├── xaa.ts                      // XAA 协议
├── xaaIdpLogin.ts              // XAA IdP 登录
├── InProcessTransport.ts        // 进程内传输
└── SdkControlTransport.ts      // SDK 控制传输
```

**重要说明**：
- **不存在** `transport/` 子目录
- **不存在** `protocol/` 子目录
- 传输相关文件直接位于 `services/mcp/` 根目录

---

## 18.3 协议消息

### 18.3.1 消息类型

```typescript
// JSON-RPC 消息格式
type JSONRPCMessage =
  | {
      jsonrpc: '2.0'
      id: number | string
      method: string
      params?: unknown
    }
  | {
      jsonrpc: '2.0'
      id: number | string
      result: unknown
    }
  | {
      jsonrpc: '2.0'
      id: number | string
      error: {
        code: number
        message: string
        data?: unknown
      }
    }
```

### 18.3.2 MCP 方法

| 方法 | 方向 | 说明 |
|------|------|------|
| `initialize` | C→S | 初始化连接 |
| `initialized` | C←S | 初始化完成 |
| `tools/list` | C→S | 列出可用工具 |
| `tools/call` | C→S | 调用工具 |
| `resources/list` | C→S | 列出资源 |
| `resources/read` | C→S | 读取资源 |
| `prompts/list` | C→S | 列出提示模板 |
| `prompts/get` | C→S | 获取提示 |

### 18.3.3 握手流程

```
Claude Code                           MCP Server
    │                                     │
    │──── initialize ──────────────────→│
    │     {                               │
    │       protocolVersion: "1.0",       │
    │       capabilities: {...}           │
    │     }                               │
    │                                     │
    │←─── initialized ────────────────────│
    │     {                               │
    │       protocolVersion: "1.0",       │
    │       capabilities: {...}           │
    │     }                               │
    │                                     │
    │──── tools/list ────────────────────→│
    │                                     │
    │←─── tools/list (result) ────────────│
    │     { tools: [...] }                 │
    │                                     │
    │──── tools/call ────────────────────→│
    │                                     │
    │←─── tools/call (result) ────────────│
```

---

## 18.4 传输层实现

### 18.4.1 传输类型

Claude Code 支持以下传输类型：

| 传输类型 | 文件 | 说明 |
|----------|------|------|
| **InProcess** | `InProcessTransport.ts` | 进程内传输 |
| **SdkControl** | `SdkControlTransport.ts` | SDK 控制传输 |
| **Stdio** | 通过 client.ts 实现 | 标准输入输出 |
| **SSE** | 通过 client.ts 实现 | Server-Sent Events |

### 18.4.2 Stdio 传输

标准输入输出传输，用于本地进程：

```typescript
// services/mcp/client.ts 中的传输实现
class StdioTransport implements Transport {
  private process: ChildProcess
  private writer: Writable
  private reader: Readable

  async connect(command: string, args: string[]): Promise<void> {
    this.process = spawn(command, args, { stdio: ['pipe', 'pipe', 'pipe'] })

    this.writer = this.process.stdin
    this.reader = this.process.stdout

    // 监听输出
    this.reader.on('data', (data) => {
      this.handleMessage(data)
    })
  }

  async send(message: JSONRPCMessage): Promise<void> {
    const line = JSON.stringify(message) + '\n'
    this.writer.write(line)
  }

  async close(): Promise<void> {
    this.process.kill()
  }
}
```

### 18.4.3 传输类型对比

| 传输类型 | 适用场景 | 延迟 | 复杂度 |
|----------|----------|------|--------|
| **Stdio** | 本地进程 | 低 | 低 |
| **SSE** | HTTP 长连接 | 中 | 中 |
| **InProcess** | 进程内通信 | 最低 | 低 |
| **SdkControl** | VS Code 扩展 | 低 | 中 |

---

## 18.5 MCP 客户端

### 18.5.1 客户端架构

```typescript
// services/mcp/client.ts
class MCPClient {
  private connections: Map<string, ServerConnection> = new Map()
  private transportFactory: Map<TransportType, TransportFactory>

  // 连接管理
  async connect(serverConfig: MCPServerConfig): Promise<void> {
    const transport = this.createTransport(serverConfig)
    const connection = new ServerConnection(transport)

    await connection.initialize()
    this.connections.set(serverConfig.name, connection)
  }

  // 工具调用
  async callTool(
    serverName: string,
    toolName: string,
    args: Record<string, unknown>
  ): Promise<ToolResult> {
    const connection = this.connections.get(serverName)
    return connection.callTool(toolName, args)
  }

  // 断开连接
  async disconnect(serverName: string): Promise<void> {
    const connection = this.connections.get(serverName)
    await connection.close()
    this.connections.delete(serverName)
  }
}
```

### 18.5.2 服务器连接

```typescript
class ServerConnection {
  private transport: Transport
  private requestId: number = 0
  private pendingRequests: Map<string, DeferredResult> = new Map()

  async initialize(): Promise<void> {
    const response = await this.sendRequest('initialize', {
      protocolVersion: '1.0',
      capabilities: {
        tools: {},
        resources: {}
      },
      clientInfo: {
        name: 'claude-code',
        version: '1.0'
      }
    })

    this.serverCapabilities = response.capabilities
  }

  async callTool(name: string, args: unknown): Promise<ToolResult> {
    const result = await this.sendRequest('tools/call', {
      name,
      arguments: args
    })

    return this.formatToolResult(result)
  }
}
```

---

## 18.6 配置管理

### 18.6.1 服务器配置

```typescript
// services/mcp/types.ts
interface MCPServerConfig {
  name: string
  transport: TransportType
  command?: string
  args?: string[]
  url?: string
  env?: Record<string, string>
  headers?: Record<string, string>
}
```

### 18.6.2 配置文件

```json
// ~/.claude/settings.json
{
  "mcpServers": [
    {
      "name": "filesystem",
      "transport": "stdio",
      "command": "npx",
      "args": ["@modelcontextprotocol/server-filesystem", "/home/user/projects"]
    },
    {
      "name": "github",
      "transport": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    {
      "name": "remote-api",
      "transport": "sse",
      "url": "https://api.example.com/mcp",
      "headers": {
        "Authorization": "Bearer ${API_TOKEN}"
      }
    }
  ]
}
```

### 18.6.3 配置解析

```typescript
// services/mcp/config.ts
const parseMCPServers = (config: Config): MCPServerConfig[] => {
  return (config.mcpServers ?? []).map((server) => ({
    name: server.name,
    transport: server.transport ?? 'stdio',
    command: server.command,
    args: server.args,
    url: server.url,
    env: resolveEnvVars(server.env),
    headers: server.headers
  }))
}

const resolveEnvVars = (
  env: Record<string, string> | undefined
): Record<string, string> | undefined => {
  if (!env) return undefined

  const result: Record<string, string> = {}
  for (const [key, value] of Object.entries(env)) {
    // 替换 ${VAR} 格式的环境变量引用
    result[key] = value.replace(/\$\{(\w+)\}/g, (_, varName) => {
      return process.env[varName] ?? ''
    })
  }
  return result
}
```

---

## 18.7 MCP 工具集成

### 18.7.1 内置 MCP 工具

Claude Code 内置两个 MCP 相关工具：

| 工具名 | 文件 | 功能 |
|--------|------|------|
| **ListMcpResourcesTool** | `tools/ListMcpResourcesTool/` | 列出 MCP 资源 |
| **ReadMcpResourceTool** | `tools/ReadMcpResourceTool/` | 读取 MCP 资源 |

### 18.7.2 动态加载

```typescript
// 启动时加载所有 MCP 工具
const loadMCPTools = async (
  mcpClient: MCPClient
): Promise<Tool[]> => {
  const tools: Tool[] = []

  for (const [serverName, connection] of mcpClient.connections) {
    const toolDefs = await connection.listTools()

    for (const toolDef of toolDefs) {
      tools.push(new MCPTool(mcpClient, serverName, toolDef))
    }
  }

  return tools
}
```

---

## 18.8 认证与安全

### 18.8.1 认证类型

| 类型 | 说明 | 配置位置 |
|------|------|----------|
| **无认证** | 本地进程 | 默认 |
| **Bearer Token** | HTTP Header | `auth.ts` |
| **API Key** | 自定义 Header | `auth.ts` |
| **OAuth 2.0** | 外部服务 | `xaa.ts`, `oauthPort.ts` |

### 18.8.2 安全措施

```typescript
// services/mcp/auth.ts
// 环境变量中的敏感信息处理
const sanitizeConfig = (config: MCPServerConfig): MCPServerConfig => {
  return {
    ...config,
    env: Object.fromEntries(
      Object.entries(config.env ?? {}).filter(([key]) =>
        !key.toLowerCase().includes('secret') &&
        !key.toLowerCase().includes('password') &&
        !key.toLowerCase().includes('token')
      )
    )
  }
}
```

---

## 18.9 错误处理

### 18.9.1 错误类型

```typescript
type MCPError =
  | { code: 'CONNECT_FAILED', message: string }
  | { code: 'TIMEOUT', message: string }
  | { code: 'TOOL_NOT_FOUND', server: string, tool: string }
  | { code: 'TOOL_ERROR', server: string, tool: string, error: string }
  | { code: 'INVALID_RESPONSE', message: string }
```

### 18.9.2 重连机制

```typescript
class ServerConnection {
  private reconnectAttempts = 0
  private maxReconnectAttempts = 3

  async handleDisconnect(): Promise<void> {
    if (this.reconnectAttempts >= this.maxReconnectAttempts) {
      throw new MCPError({ code: 'CONNECT_FAILED', message: 'Max reconnection attempts reached' })
    }

    this.reconnectAttempts++
    await sleep(Math.pow(2, this.reconnectAttempts) * 1000)

    await this.transport.reconnect()
    await this.initialize()
  }
}
```

---

## 18.10 调试与监控

### 18.10.1 调试模式

```bash
# 启用 MCP 调试
claude --debug

# 查看 MCP 日志
> /mcp debug

[MCP] 2024-04-01T10:00:00Z - Connecting to filesystem...
[MCP] 2024-04-01T10:00:01Z - Connected
[MCP] 2024-04-01T10:00:05Z - → tools/call { name: "read_file" }
[MCP] 2024-04-01T10:00:05Z - ← tools/call result { success: true }
```

### 18.10.2 性能监控

```typescript
// 追踪 MCP 调用延迟
const withTracing = async <T>(
  serverName: string,
  toolName: string,
  fn: () => Promise<T>
): Promise<T> => {
  const start = Date.now()

  try {
    const result = await fn()
    const duration = Date.now() - start

    telemetry.track('mcp:tool:duration', {
      server: serverName,
      tool: toolName,
      duration,
      success: true
    })

    return result
  } catch (error) {
    const duration = Date.now() - start

    telemetry.track('mcp:tool:duration', {
      server: serverName,
      tool: toolName,
      duration,
      success: false
    })

    throw error
  }
}
```

---

## 18.11 本章小结

本章介绍了 MCP 协议架构：

| 主题 | 核心要点 |
|------|----------|
| **协议概述** | 标准化的工具/资源通信协议 |
| **架构组件** | 客户端、连接管理器、传输层 |
| **消息格式** | JSON-RPC、协议方法、握手流程 |
| **传输层** | InProcess、SdkControl、Stdio、SSE |
| **客户端实现** | 连接管理、工具调用、生命周期 |
| **配置管理** | 服务器配置、环境变量解析 |
| **工具集成** | ListMcpResources、ReadMcpResource |
| **认证安全** | 多种认证类型、安全措施 |
| **错误处理** | 错误类型、重连机制 |
| **调试监控** | 调试模式、性能追踪 |

---

## 下章预告

下一章我们将学习 **状态管理与渲染**，深入了解 Claude Code 的响应式状态管理和终端 UI 渲染机制。

---

*© 本书由 Claude Code 辅助撰写*
