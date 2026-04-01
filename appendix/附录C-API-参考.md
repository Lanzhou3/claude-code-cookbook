# 附录 C：API 参考

> 本附录提供 Claude Code 内部 API 和配置的完整参考。

---

## C.1 配置文件

### C.1.1 settings.json 完整结构

```json
{
  "//": "基础配置",
  "apiKey": "sk-ant-xxxxx...",
  "provider": "anthropic",
  "model": "claude-sonnet-4-6",
  "baseUrl": "https://api.anthropic.com",
  "apiVersion": "2023-06-01",

  "//": "模型配置",
  "maxTokens": 8192,
  "temperature": 1.0,
  "topP": 0.9,
  "topK": 40,

  "//": "权限配置",
  "permissions": {
    "mode": "default",
    "allow": [],
    "deny": [],
    "writablePaths": ["src/**", "tests/**"],
    "protectedPaths": ["/etc/**", "~/.ssh/**"],
    "shell": {
      "allowedCommands": ["npm", "git", "docker"],
      "deniedCommands": ["rm -rf /*"]
    },
    "audit": {
      "enabled": true,
      "path": "~/.claude/audit.log"
    }
  },

  "//": "UI 配置",
  "appearance": {
    "theme": "dark",
    "emoji": true,
    "color": true,
    "compact": false,
    "fontSize": 14,
    "fontFamily": "Menlo"
  },
  "output": {
    "format": "rich",
    "syntaxHighlight": true,
    "showTimestamps": true,
    "showTokenCount": true,
    "maxOutputLines": 1000
  },

  "//": "MCP 配置",
  "mcpServers": [
    {
      "name": "filesystem",
      "command": "npx",
      "args": ["@modelcontextprotocol/server-filesystem", "/path"],
      "transport": "stdio"
    }
  ],

  "//": "钩子配置",
  "hooks": {
    "enabled": true,
    "paths": ["~/.claude/hooks"]
  },

  "//": "日志配置",
  "logging": {
    "level": "info",
    "file": "~/.claude/logs/claude.log",
    "rotate": {
      "enabled": true,
      "maxSize": "10MB",
      "maxFiles": 5
    }
  },

  "//": "网络配置",
  "network": {
    "proxy": {
      "http": "http://proxy:8080",
      "https": "http://proxy:8080",
      "noProxy": ["localhost"]
    },
    "timeout": {
      "request": 30000,
      "connect": 10000,
      "read": 60000
    }
  },

  "//": "上下文配置",
  "context": {
    "maxMessages": 50,
    "compressAfter": 30,
    "summaryAfter": 20,
    "optimization": {
      "autoCompress": true,
      "compressionThreshold": 0.75
    }
  },

  "//": "会话配置",
  "sessions": {
    "autoSave": true,
    "interval": "5m",
    "maxAge": "30d"
  },

  "//": "记忆配置",
  "memory": {
    "learning": {
      "enabled": true,
      "confidenceThreshold": 0.8
    },
    "encryption": {
      "enabled": false
    }
  }
}
```

---

## C.2 配置类型定义

### C.2.1 Config

```typescript
interface Config {
  // 基础
  apiKey?: string
  provider?: Provider
  model?: string
  baseUrl?: string
  apiVersion?: string

  // 模型
  maxTokens?: number
  temperature?: number
  topP?: number
  topK?: number

  // 权限
  permissions?: PermissionsConfig

  // UI
  appearance?: AppearanceConfig
  output?: OutputConfig

  // MCP
  mcpServers?: MCPServerConfig[]

  // 钩子
  hooks?: HooksConfig

  // 日志
  logging?: LoggingConfig

  // 网络
  network?: NetworkConfig

  // 上下文
  context?: ContextConfig

  // 会话
  sessions?: SessionsConfig

  // 记忆
  memory?: MemoryConfig
}
```

### C.2.2 Provider

```typescript
type Provider = 'anthropic' | 'aws-bedrock' | 'gcp-vertex' | 'azure-foundry'
```

### C.2.3 PermissionsConfig

```typescript
interface PermissionsConfig {
  mode?: PermissionMode
  allow?: string[]
  deny?: string[]
  writablePaths?: string[]
  protectedPaths?: string[]
  shell?: {
    allowedCommands?: string[]
    deniedCommands?: string[]
    commandRules?: CommandRule[]
  }
  audit?: {
    enabled?: boolean
    path?: string
  }
}

type PermissionMode = 'default' | 'plan' | 'acceptEdits' | 'auto'
```

### C.2.4 MCPServerConfig

```typescript
interface MCPServerConfig {
  name: string
  transport?: TransportType
  command?: string
  args?: string[]
  url?: string
  env?: Record<string, string>
  headers?: Record<string, string>
}

type TransportType = 'stdio' | 'sse' | 'http' | 'websocket'
```

---

## C.3 环境变量参考

### C.3.1 API 环境变量

| 变量 | 类型 | 说明 |
|------|------|------|
| `ANTHROPIC_API_KEY` | string | Anthropic API 密钥 |
| `ANTHROPIC_BASE_URL` | string | API 基础 URL |
| `ANTHROPIC_VERSION` | string | API 版本 |
| `ANTHROPIC_ACCOUNT_ID` | string | 账户 ID |

### C.3.2 AWS Bedrock 环境变量

| 变量 | 类型 | 说明 |
|------|------|------|
| `AWS_REGION` | string | AWS 区域 |
| `AWS_ACCESS_KEY_ID` | string | AWS 访问密钥 |
| `AWS_SECRET_ACCESS_KEY` | string | AWS 秘密密钥 |
| `AWS_SESSION_TOKEN` | string | AWS 会话令牌 |
| `AWS_PROFILE` | string | AWS Profile |
| `AWS_ROLE_ARN` | string | IAM Role ARN |

### C.3.3 GCP Vertex AI 环境变量

| 变量 | 类型 | 说明 |
|------|------|------|
| `ANTHROPIC_VERTEX_PROJECT_ID` | string | GCP 项目 ID |
| `ANTHROPIC_VERTEX_LOCATION` | string | GCP 位置 |
| `GCP_REGION` | string | GCP 区域 |
| `GOOGLE_APPLICATION_CREDENTIALS` | string | 凭证文件路径 |

### C.3.4 Azure Foundry 环境变量

| 变量 | 类型 | 说明 |
|------|------|------|
| `ANTHROPIC_FOUNDRY_RESOURCE` | string | Azure Foundry 资源 |
| `ANTHROPIC_FOUNDRY_TOKEN` | string | Azure Foundry 令牌 |
| `ANTHROPIC_FOUNDRY_API_KEY` | string | Azure Foundry API Key |

### C.3.5 Claude Code 环境变量

| 变量 | 类型 | 说明 |
|------|------|------|
| `CLAUDE_CONFIG_DIR` | string | 配置目录 |
| `CLAUDE_DATA_DIR` | string | 数据目录 |
| `CLAUDE_SESSION_DIR` | string | 会话目录 |
| `CLAUDE_LOG_LEVEL` | string | 日志级别 |
| `ENABLE_LSP_TOOL` | string | 启用 LSP 工具 |
| `CLAUDE_CODE_SIMPLE` | string | 简单模式 |
| `USER_TYPE` | string | 用户类型（'ant' 为内部构建） |

---

## C.4 工具 Schema

### C.4.1 工具定义 Schema

```typescript
interface ToolDefinition {
  name: string
  description: string
  inputSchema: JSONSchema
}

interface JSONSchema {
  type: 'object'
  properties: Record<string, JSONSchemaProperty>
  required?: string[]
  additionalProperties?: boolean
}

interface JSONSchemaProperty {
  type: string
  description?: string
  enum?: unknown[]
  default?: unknown
  minimum?: number
  maximum?: number
  minLength?: number
  maxLength?: number
  pattern?: string
}
```

### C.4.2 工具结果 Schema

```typescript
// 实际 ToolResult 定义 (Tool.ts)
export type ToolResult<T> = {
  data: T                   // 结果数据（注意：不是 content）
  newMessages?: (
    | UserMessage
    | AssistantMessage
    | AttachmentMessage
    | SystemMessage
  )[]
  contextModifier?: (context: ToolUseContext) => ToolUseContext
  mcpMeta?: {
    _meta?: Record<string, unknown>
    structuredContent?: Record<string, unknown>
  }
}
```

---

## C.5 MCP 协议类型

### C.5.1 MCP 消息类型

```typescript
type MCPMessage =
  | MCPRequest
  | MCPResponse
  | MCPError

interface MCPRequest {
  jsonrpc: '2.0'
  id: number | string
  method: string
  params?: unknown
}

interface MCPResponse {
  jsonrpc: '2.0'
  id: number | string
  result?: unknown
}

interface MCPError {
  jsonrpc: '2.0'
  id: number | string
  error: {
    code: number
    message: string
    data?: unknown
  }
}
```

### C.5.2 MCP 能力

```typescript
interface ServerCapabilities {
  tools?: {
    listChanged?: boolean
  }
  resources?: {
    subscribe?: boolean
    listChanged?: boolean
  }
  prompts?: {
    listChanged?: boolean
  }
  logging?: {}
}
```

---

## C.6 状态类型

### C.6.1 AppState

```typescript
interface AppState {
  messages: Message[]
  pendingMessages: Message[]
  currentTask: Task | null
  taskQueue: Task[]
  ui: UIState
  toolResults: Map<string, ToolResult>
  toolStates: Map<string, ToolState>
  sessionHistory: SessionSummary[]
  bookmarks: Bookmark[]
}

interface UIState {
  isLoading: boolean
  isProcessing: boolean
  scrollPosition: number
  focusedInput: string | null
  selectedIndex: number
}
```

### C.6.2 Message

```typescript
interface Message {
  id: string
  role: 'user' | 'assistant' | 'system'
  content: string | ContentBlock[]
  timestamp: number
  metadata?: {
    model?: string
    usage?: TokenUsage
  }
}

type ContentBlock =
  | { type: 'text'; text: string }
  | { type: 'tool_use'; id: string; name: string; input: unknown }
  | { type: 'tool_result'; tool_use_id: string; content: string }
```

---

## C.7 Feature Flags

Claude Code 使用 `bun:bundle` 的 `feature()` 函数控制功能开关：

### C.7.1 常用 Feature Flags

| Flag | 说明 |
|------|------|
| `KAIROS` | KAIROS 特性 |
| `DAEMON` | 守护进程模式 |
| `VOICE_MODE` | 语音模式 |
| `WEB_BROWSER_TOOL` | Web 浏览器工具 |
| `OVERFLOW_TEST_TOOL` | 溢出测试工具 |
| `CONTEXT_COLLAPSE` | 上下文折叠 |
| `TERMINAL_PANEL` | 终端面板 |
| `HISTORY_SNIP` | 历史摘要 |
| `UDS_INBOX` | UDS 收件箱 |
| `WORKFLOW_SCRIPTS` | 工作流脚本 |
| `AGENT_TRIGGERS` | 代理触发器 |
| `AGENT_TRIGGERS_REMOTE` | 远程触发器 |
| `MONITOR_TOOL` | 监控工具 |
| `KAIROS_PUSH_NOTIFICATION` | KAIROS 推送通知 |
| `KAIROS_GITHUB_WEBHOOKS` | KAIROS GitHub Webhooks |
| `COORDINATOR_MODE` | 协调器模式 |

### C.7.2 检查 Feature

```typescript
import { feature } from 'bun:bundle'

if (feature('KAIROS')) {
  // KAIROS 特性代码
}
```

---

*© 本书由 Claude Code 辅助撰写*
