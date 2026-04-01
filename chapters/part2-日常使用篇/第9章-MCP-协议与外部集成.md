# 第9章 MCP 协议与外部集成

> 本章详细介绍 Model Context Protocol (MCP) 协议，以及如何配置和使用 MCP 服务器来扩展 Claude Code 的能力。

---

## 9.1 MCP 协议概述

### 9.1.1 什么是 MCP

MCP (Model Context Protocol) 是一个开放协议，用于 AI 模型与外部工具和资源之间的标准化通信。它由 Anthropic 提出并广泛采用。

```
┌──────────────┐         ┌──────────────┐
│  Claude Code │ ←─────→ │  MCP Server  │
│   (Client)   │  JSON-RPC            │
└──────────────┘         └──────────────┘
        │                         │
        │                         ▼
        │                ┌──────────────────┐
        │                │   Local Tools    │
        │                │   Resources     │
        │                │   Prompts       │
        │                └──────────────────┘
        │
        ▼
┌──────────────┐
│  Claude API  │
└──────────────┘
```

### 9.1.2 MCP 核心概念

| 概念 | 说明 |
|------|------|
| **MCP Server** | 提供工具和资源的外部服务 |
| **MCP Client** | Claude Code 内置的 MCP 客户端 |
| **Tools** | MCP 服务器提供的可调用工具 |
| **Resources** | 可读取的数据源（文件、API 等） |
| **Prompts** | 预定义的提示词模板 |

### 9.1.3 MCP vs 传统集成

| 特性 | MCP | 传统集成 |
|------|-----|----------|
| **标准化** | 统一协议 | 各自实现 |
| **可发现性** | 自动发现工具 | 手动配置 |
| **类型安全** | JSON Schema | 依赖文档 |
| **安全性** | 权限隔离 | 直接访问 |
| **可扩展性** | 社区共享 | 各自为政 |

---

## 9.2 MCP 服务器配置

### 9.2.1 配置文件位置

MCP 服务器在以下位置配置：

```bash
# 主配置文件
~/.claude/settings.json

# 项目级配置
.claude/settings.json
```

### 9.2.2 基本配置

```json
{
  "mcpServers": [
    {
      "name": "filesystem",
      "command": "npx",
      "args": ["@modelcontextprotocol/server-filesystem", "/home/user/projects"]
    }
  ]
}
```

### 9.2.3 完整配置示例

```json
{
  "mcpServers": [
    {
      "name": "filesystem",
      "command": "npx",
      "args": ["@modelcontextprotocol/server-filesystem", "/home/user/projects"],
      "env": {}
    },
    {
      "name": "github",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_xxxxx"
      }
    },
    {
      "name": "slack",
      "command": "node",
      "args": ["/path/to/slack-server/dist/index.js"],
      "env": {
        "SLACK_BOT_TOKEN": "xoxb-xxxxx",
        "SLACK_TEAM_ID": "T0123456789"
      }
    }
  ]
}
```

---

## 9.3 传输类型

### 9.3.1 Stdio 传输

标准输入输出传输，适用于本地进程：

```json
{
  "name": "filesystem",
  "transport": "stdio",
  "command": "npx",
  "args": ["@modelcontextprotocol/server-filesystem", "/path"]
}
```

### 9.3.2 HTTP/SSE 传输

Server-Sent Events，适用于远程服务：

```json
{
  "name": "remote-api",
  "transport": "sse",
  "url": "https://api.example.com/mcp",
  "headers": {
    "Authorization": "Bearer token"
  }
}
```

### 9.3.3 WebSocket 传输

双向通信，适用于实时应用：

```json
{
  "name": "realtime",
  "transport": "ws",
  "url": "wss://api.example.com/mcp",
  "reconnect": true
}
```

---

## 9.4 常用 MCP 服务器

### 9.4.1 文件系统服务器

提供安全的文件系统访问：

```bash
# 安装
npm install -g @modelcontextprotocol/server-filesystem

# 配置
{
  "name": "filesystem",
  "command": "npx",
  "args": ["@modelcontextprotocol/server-filesystem", "/home/user/projects"]
}
```

**可用工具：**
- `read_file` - 读取文件
- `read_directory` - 列出目录
- `write_file` - 写入文件
- `create_directory` - 创建目录
- `move_file` - 移动文件
- `search_files` - 搜索文件

### 9.4.2 GitHub 服务器

集成 GitHub API：

```bash
# 安装
npm install -g @modelcontextprotocol/server-github

# 配置
{
  "name": "github",
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-github"],
  "env": {
    "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_xxxxx"
  }
}
```

**可用工具：**
- `get_file` - 获取文件内容
- `create_file` - 创建文件
- `update_file` - 更新文件
- `list_issues` - 列出 Issue
- `create_issue` - 创建 Issue
- `search_code` - 搜索代码

### 9.4.3 Slack 服务器

集成 Slack 消息：

```bash
# 安装
npm install -g @modelcontextprotocol/server-slack

# 配置
{
  "name": "slack",
  "command": "node",
  "args": ["/path/to/slack-server/dist/index.js"],
  "env": {
    "SLACK_BOT_TOKEN": "xoxb-xxxxx",
    "SLACK_TEAM_ID": "T0123456789"
  }
}
```

**可用工具：**
- `send_message` - 发送消息
- `list_channels` - 列出频道
- `search_messages` - 搜索消息

---

## 9.5 使用 MCP 工具

### 9.5.1 查看可用工具

```bash
> /mcp tools

已连接的 MCP 服务器：
┌─────────────────────────────────────────────────────────┐
│ 📦 filesystem                                          │
│   ├── read_file                                       │
│   ├── write_file                                      │
│   └── search_files                                    │
│                                                         │
│ 📦 github                                             │
│   ├── get_file                                        │
│   ├── create_issue                                    │
│   └── search_code                                     │
│                                                         │
│ 📦 slack                                              │
│   ├── send_message                                    │
│   └── list_channels                                   │
└─────────────────────────────────────────────────────────┘
```

### 9.5.2 调用 MCP 工具

```bash
# 直接调用
> 使用 github 工具搜索 zlib 仓库的代码
  MCP: github → search_code(query: "zlib", repo: "owner/repo")

# 组合使用
> 先用 filesystem 读取文件，然后用 github 创建 PR
```

### 9.5.3 MCP 工具输出

```
[MCP] filesystem → read_file
──────────────────────────────────────
文件: /home/user/project/src/main.ts
大小: 1.2 KB
修改: 2024-04-01 10:30:00
──────────────────────────────────────
1 | import { main } from './main'
2 | console.log('Hello')
3 | ...
──────────────────────────────────────
```

---

## 9.6 MCP 资源

### 9.6.1 什么是资源

资源是 MCP 服务器提供的只读数据，可以被 Claude Code 读取并在上下文中使用：

```bash
# 列出可用资源
> /mcp resources

filesystem:
  file:///home/user/project/package.json
  file:///home/user/project/README.md

github:
  github://owner/repo/README.md
  github://owner/repo/issues?state=open
```

### 9.6.2 读取资源

```bash
# 读取单个资源
> /mcp resource read github://anthropic/claude-code/README.md

# 加载资源到上下文
> 加载 package.json 到上下文
```

### 9.6.3 资源订阅

```bash
# 订阅实时资源
> /mcp subscribe github://owner/repo/issues

# 取消订阅
> /mcp unsubscribe github://owner/repo/issues
```

---

## 9.7 安全考虑

### 9.7.1 权限控制

```json
{
  "mcpServers": [
    {
      "name": "filesystem",
      "command": "npx",
      "args": ["@modelcontextprotocol/server-filesystem", "/path"],
      "permissions": {
        "allow": ["read_file", "write_file"],
        "deny": ["delete_file"]
      }
    }
  ]
}
```

### 9.7.2 令牌管理

```bash
# 使用环境变量存储令牌
export GITHUB_TOKEN=ghp_xxxxx

# 在配置中引用
{
  "mcpServers": [
    {
      "name": "github",
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  ]
}
```

### 9.7.3 网络安全

```json
{
  "name": "remote-server",
  "url": "https://secure-api.example.com/mcp",
  "verify": true,
  "cert": "/path/to/cert.pem"
}
```

---

## 9.8 故障排除

### 9.8.1 常见错误

| 错误 | 原因 | 解决方案 |
|------|------|----------|
| `Server not found` | 服务未安装 | 运行 npx install |
| `Connection refused` | 端口被占用 | 检查端口或重启 |
| `Auth failed` | 令牌无效 | 检查环境变量 |
| `Timeout` | 网络问题 | 检查连接或增加超时 |

### 9.8.2 调试模式

```bash
# 启用 MCP 调试
claude --debug

# 查看 MCP 日志
> /mcp debug

[MCP] Connection log:
  - 10:00:00 Connecting to filesystem...
  - 10:00:01 ✓ Connected
  - 10:05:00 → read_file(path: "/test.txt")
  - 10:05:01 ← Success
```

### 9.8.3 验证连接

```bash
# 测试 MCP 连接
> /mcp test

测试 filesystem... ✓
测试 github...     ✓
测试 slack...       ✗ Connection failed

详情请查看 /mcp debug
```

---

## 9.9 开发 MCP 服务器

### 9.9.1 基本结构

```typescript
// my-mcp-server/index.ts
import { McpServer } from "@modelcontextprotocol/sdk/server";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio";

const server = new McpServer({
  name: "my-server",
  version: "1.0.0"
});

// 注册工具
server.tool(
  "greet",
  "生成问候语",
  { name: z.string() },
  async ({ name }) => ({
    content: [{ type: "text", text: `Hello, ${name}!` }]
  })
);

// 启动服务器
const transport = new StdioServerTransport();
server.run(transport);
```

### 9.9.2 注册资源

```typescript
server.resource(
  "greeting",
  "greeting://template",
  async (uri) => ({
    contents: [{
      uri: uri.href,
      mimeType: "text/plain",
      text: "Hello, {name}!"
    }]
  })
);
```

### 9.9.3 测试 MCP 服务器

```bash
# 本地测试
node dist/index.js

# 使用 Claude Code 测试
{
  "mcpServers": [
    {
      "name": "my-server",
      "command": "node",
      "args": ["/path/to/dist/index.js"]
    }
  ]
}
```

---

## 9.10 本章小结

本章介绍了 MCP 协议与外部集成：

| 要点 | 内容 |
|------|------|
| **MCP 概述** | 协议原理、核心概念、与传统集成对比 |
| **配置** | 配置文件位置、基本配置、完整示例 |
| **传输类型** | Stdio、HTTP/SSE、WS |
| **服务器** | filesystem、GitHub、Slack 等 |
| **使用** | 查看工具、调用工具、工具输出 |
| **资源** | 资源概念、读取、订阅 |
| **安全** | 权限控制、令牌管理、网络安全 |
| **故障排除** | 常见错误、调试、验证连接 |
| **开发** | MCP 服务器开发结构、注册工具和资源 |

---

## 下章预告

下一章我们将进入**第三部分：进阶配置篇**，学习**权限系统深入**：

- 权限模型架构
- 规则配置
- 风险分类
- 自定义权限

---

*© 本书由 Claude Code 辅助撰写*
