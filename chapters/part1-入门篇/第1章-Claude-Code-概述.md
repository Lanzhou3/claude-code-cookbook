# 第1章 Claude Code 概述

> 本章介绍 Claude Code 的定位、核心能力和系统架构概览。

---

## 1.1 Claude Code 是什么

Claude Code 是 Anthropic 官方推出的**终端 AI 编程助手**，它是一个运行在本地终端的 CLI 工具，能够帮助开发者完成各种编程任务。

### 与其他工具的对比

| 特性 | Claude Code | GitHub Copilot |Cursor |
|------|-------------|----------------|--------|
| **运行环境** | 本地终端 | IDE 插件 | IDE 插件 |
| **交互方式** | REPL 对话 | 内联补全 | 聊天 + 补全 |
| **工具集** | 20-40+ 动态工具 | 基础补全 | 中等 |
| **MCP 支持** | 原生支持 | 有限 | 有限 |
| **可扩展性** | 插件 + 技能 | 受限 | 受限 |

### 核心定位

Claude Code 的设计理念是：

> **让 AI 成为你的终端搭档**，不是简单的补全工具，而是能够理解项目、执行任务、操作文件的智能助手。

---

## 1.2 核心能力矩阵

Claude Code 提供七大核心能力：

### 2.1 文件操作

```bash
# 读取文件
Claude: 请读取 src/index.ts 的内容

# 编辑文件
Claude: 在这个函数中添加参数验证

# 创建文件
Claude: 帮我创建一个新的工具模块
```

### 2.2 Shell 命令执行

```bash
# 运行测试
Claude: 运行 npm test 看看有没有失败

# Git 操作
Claude: 帮我提交这些更改，commit message 写 "fix: 修复登录bug"

# 项目构建
Claude: 执行 docker build 构建镜像
```

### 2.3 代码搜索与理解

```bash
# 搜索代码
Claude: 帮我找到所有使用 async/await 的地方

# 项目理解
Claude: 这个项目的架构是怎样的？

# 依赖分析
Claude: 看看 package.json 里的依赖是否有安全漏洞
```

### 2.4 自动化任务

```bash
# 重构
Claude: 把所有 class 组件改成 function 组件

# 代码审查
Claude: /review 帮我审查这个 PR

# 测试生成
Claude: 为这个模块写单元测试
```

### 2.5 上下文感知

```bash
# 理解项目结构
Claude: 这是什么框架搭建的项目？

# Git 上下文
Claude: 当前在哪个分支？有哪些未提交的更改？

# 项目配置
Claude: 帮我查看项目的 TypeScript 配置
```

### 2.6 多工具协作

```bash
# 搜索 + 编辑
Claude: 找到所有 console.log 然后删掉它们

# 读取 + 执行
Claude: 先看看这个脚本干什么，然后运行它
```

### 2.7 持久化与记忆

```bash
# 会话恢复
Claude: /resume 恢复上次的会话

# 项目记忆
Claude: 把这个项目的技术栈记录到 MEMORY.md
```

---

## 1.3 系统架构概览

Claude Code 采用**分层架构**，每层职责明确：

```
┌─────────────────────────────────────────────────────────────┐
│                     CLI 入口层                               │
│            entrypoints/cli.tsx → main.tsx                    │
├─────────────────────────────────────────────────────────────┤
│                     REPL 界面层                              │
│                 screens/REPL.tsx + ink/                     │
├─────────────────────────────────────────────────────────────┤
│                     查询引擎层                                │
│               QueryEngine.ts + query.ts                      │
├─────────────────────────────────────────────────────────────┤
│                     工具编排层                               │
│         services/tools/ + Tool.ts + tools/                    │
├─────────────────────────────────────────────────────────────┤
│                     API 通信层                               │
│          services/api/claude.ts + client.ts                  │
├─────────────────────────────────────────────────────────────┤
│                     MCP 协议层                               │
│               services/mcp/ (完整实现)                      │
├─────────────────────────────────────────────────────────────┤
│                     状态管理层                               │
│           bootstrap/state.ts + state/AppState.ts             │
├─────────────────────────────────────────────────────────────┤
│                     扩展机制层                               │
│              plugins/ + skills/ + hooks/                    │
└─────────────────────────────────────────────────────────────┘
```

### 各层职责

| 层次 | 核心文件 | 职责 |
|------|----------|------|
| **CLI 入口** | `entrypoints/cli.tsx` | 参数解析、版本处理、初始化 |
| **REPL 界面** | `screens/REPL.tsx` | 用户交互、消息渲染、输入处理 |
| **查询引擎** | `QueryEngine.ts` | 对话循环、API 调用、工具调度 |
| **工具系统** | `Tool.ts`, `tools/*` | 动态工具集的实现与编排 |
| **API 通信** | `services/api/*` | Anthropic API 调用、流式处理 |
| **MCP 协议** | `services/mcp/*` | 外部工具服务器连接 |
| **状态管理** | `bootstrap/state.ts` | 全局状态、会话管理 |
| **扩展机制** | `plugins/`, `skills/` | 插件、技能、钩子 |

---

## 1.4 技术栈选型

Claude Code 在技术选型上有独特的考量：

### 4.1 为什么用 Bun

| 考量 | Bun 的优势 |
|------|-----------|
| **启动速度** | 比 Node.js 快 3-4 倍 |
| **TypeScript** | 原生支持，无需编译 |
| **包管理** | 内置 bundler、package manager |
| **API 兼容** | 兼容 Node.js 模块 |

### 4.2 为什么自研 Ink

现有的终端 UI 库无法满足需求：

| 需求 | Ink 的解决方案 |
|------|---------------|
| React 组件模型 | 自研 Fiber 架构 |
| 精确光标控制 | 自研屏幕渲染 |
| 焦点管理 | 自研 FocusManager |
| Vim 模式 | 自研 Vim Input Handler |

### 4.3 为什么用 Zod

```typescript
// 类型安全 + 运行时验证
import { z } from 'zod/v4'

const AgentInputSchema = z.object({
  description: z.string(),
  prompt: z.string(),
  model: z.enum(['sonnet', 'opus', 'haiku']).optional()
})

// 解析用户输入，同时验证类型
const input = AgentInputSchema.parse(rawInput)
```

### 4.4 为什么用 MCP

MCP (Model Context Protocol) 提供了：

- **标准化**：统一的工具/资源访问协议
- **可扩展**：任何人都可以开发 MCP 服务器
- **安全性**：OAuth 认证、权限隔离
- **互操作**：Claude Code 可以调用任何 MCP 服务器的工具

---

## 1.5 目录结构

Claude Code 的源码目录组织：

```
claude-code/
├── entrypoints/              # CLI 入口
│   ├── cli.tsx             # 引导入口
│   └── init.tsx            # 初始化
├── bootstrap/               # 引导状态
│   └── state.ts            # 全局单例状态
├── state/                   # 响应式状态
│   ├── AppState.ts         # AppState 类型
│   └── AppStateStore.ts    # Observable Store
├── QueryEngine.ts           # 核心查询引擎
├── query.ts                 # 查询管道
├── Tool.ts                  # 工具接口
├── tools.ts                 # 工具注册表
├── tools/                   # 动态工具实现
│   ├── BashTool/           # Shell 执行
│   ├── AgentTool/          # 子代理
│   ├── FileReadTool/       # 文件读取
│   ├── FileEditTool/       # 文件编辑
│   ├── FileWriteTool/      # 文件写入
│   ├── GlobTool/           # 模式匹配
│   ├── GrepTool/           # 内容搜索
│   ├── MCPTool/            # MCP 工具
│   └── ...                 # 更多工具
├── services/
│   ├── api/               # API 客户端
│   │   ├── client.ts       # 多提供商
│   │   └── claude.ts       # Claude API
│   ├── mcp/                # MCP 协议
│   │   ├── client.ts        # MCP 客户端
│   │   ├── types.ts         # 类型定义
│   │   └── config.ts        # 配置解析
│   ├── compact/             # 上下文压缩
│   └── tools/              # 工具编排
│       ├── toolExecution.ts # 执行
│       └── toolOrchestration.ts # 并发
├── screens/                # REPL 界面
│   └── REPL.tsx
├── components/              # React 组件
├── ink/                    # 终端渲染框架
│   ├── ink.tsx            # Ink 主类
│   ├── reconciler.ts       # Fiber 协调器
│   ├── renderer.ts         # 渲染器
│   ├── screen.ts           # 屏幕管理
│   └── ...                 # 更多组件
├── utils/                  # 工具函数
├── types/                  # 类型定义
├── constants/               # 常量
├── hooks/                   # React Hooks
├── plugins/                # 插件系统
├── skills/                 # 技能系统
└── memdir/                 # 记忆系统
```

---

## 1.6 快速体验

安装完成后，你可以：

### 6.1 查看帮助

```bash
claude --help
```

### 6.2 基本对话

```bash
$ claude
Claude Code CLI

你好！我可以帮你编程。请问有什么需要帮助的？

> 帮我创建一个 hello world 程序

我来帮你创建一个 Hello World 程序。首先告诉我你使用什么语言？
```

### 6.3 执行命令

```bash
$ claude
> 运行 ls 看看当前目录有什么

当前目录内容：
README.md
src/
package.json
node_modules/
```

### 6.4 文件操作

```bash
$ claude
> 帮我看看 src/index.ts 的内容

// src/index.ts
import { main } from './main'

console.log('Hello, World!')

export { main }
```

---

## 1.7 本章小结

本章介绍了 Claude Code 的基本概念：

| 要点 | 内容 |
|------|------|
| **定位** | 终端 AI 编程助手，本地运行 |
| **能力** | 文件操作、Shell 执行、代码搜索、自动化 |
| **架构** | 分层设计，模块职责明确 |
| **技术** | Bun + React/Ink + TypeScript + Zod + MCP |
| **特色** | 动态工具集、MCP 协议、扩展机制 |

---

## 下章预告

下一章我们将学习**安装与配置**，包括：

- 不同操作系统的安装方式
- API Key 配置
- 多提供商设置（AWS Bedrock、GCP Vertex AI）
- 首次启动流程

---

*© 本书由 Claude Code 辅助撰写*
