# 附录 B：工具参考

> 本附录列出 Claude Code 所有内置工具及其用法。
>
> **重要说明**：Claude Code 的工具数量是**动态变化**的，取决于 feature flags、环境变量和构建类型。以下列出的是**始终可用的工具**和**常见条件工具**。

---

## B.1 重要说明

### B.1.1 工具动态性

Claude Code 使用**动态工具加载**机制：

```typescript
// 工具是否可用取决于：
// 1. feature flags (通过 bun:bundle 的 feature() 函数)
 // 2. 环境变量 (如 ENABLE_LSP_TOOL, NODE_ENV)
// 3. 构建类型 (USER_TYPE === 'ant')
// 4. 配置开关 (如 isTodoV2Enabled(), isWorktreeModeEnabled())
```

### B.1.2 工具命名规范

- **始终可用**：基础工具集，约 20 个
- **条件可用**：根据配置和环境，10-20+ 个不等
- 工具通过 `getAllBaseTools()` 动态返回

---

## B.2 文件操作工具

### B.2.1 Read (FileReadTool)

读取文件内容。

```bash
# 基本用法
Read file_path="src/index.ts"

# 显示行号
Read file_path="src/index.ts" show_line_numbers=true

# 限制行数
Read file_path="src/index.ts" offset=0 limit=50
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `file_path` | string | 是 | 文件路径 |
| `show_line_numbers` | boolean | 否 | 显示行号 |
| `offset` | number | 否 | 起始行 |
| `limit` | number | 否 | 最大行数 |

### B.2.2 Write (FileWriteTool)

写入或创建文件。

```bash
# 创建文件
Write file_path="src/new.ts" content="// new file"

# 覆盖文件
Write file_path="src/existing.ts" content="// new content"
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `file_path` | string | 是 | 文件路径 |
| `content` | string | 是 | 文件内容 |

### B.2.3 Edit (FileEditTool)

编辑文件中的特定内容。

```bash
# 替换文本
Edit file_path="src/index.ts" old_string="old" new_string="new"

# 精确匹配
Edit file_path="src/index.ts" old_string="exact match" new_string="replacement" exact_match=true
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `file_path` | string | 是 | 文件路径 |
| `old_string` | string | 是 | 要替换的文本 |
| `new_string` | string | 是 | 替换后的文本 |
| `exact_match` | boolean | 否 | 是否精确匹配 |
| `create_if_missing` | boolean | 否 | 文件不存在时创建 |

### B.2.4 NotebookEdit (NotebookEditTool)

编辑 Jupyter Notebook。

```bash
# 编辑 notebook
NotebookEdit notebook_path="analysis.ipynb" cell_id="abc" new_source="# Updated content"
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `notebook_path` | string | 是 | Notebook 路径 |
| `cell_id` | string | 是 | 单元格 ID |
| `new_source` | string | 是 | 新内容 |

---

## B.3 搜索工具

### B.3.1 Glob (GlobTool)

**注意**：当 Claude Code 内置 ripgrep 时，此工具可能被跳过（使用内置的更快版本）。

```bash
# 查找所有 TS 文件
Glob pattern="**/*.ts"

# 排除目录
Glob pattern="src/**/*.ts" exclude="**/*.test.ts"
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `pattern` | string | 是 | 匹配模式 |
| `exclude` | string | 否 | 排除模式 |

### B.3.2 Grep (GrepTool)

**注意**：当 Claude Code 内置 ripgrep 时，此工具可能被跳过。

```bash
# 基本搜索
Grep pattern="async function" path="src"

# 忽略大小写
Grep pattern="error" path="src" ignore_case=true

# 递归搜索
Grep pattern="TODO" path="src" recursive=true
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `pattern` | string | 是 | 搜索模式 |
| `path` | string | 是 | 搜索路径 |
| `ignore_case` | boolean | 否 | 忽略大小写 |
| `recursive` | boolean | 否 | 递归搜索 |
| `file_pattern` | string | 否 | 文件匹配模式 |

---

## B.4 Shell 工具

### B.4.1 Bash (BashTool)

执行 Shell 命令。

```bash
# 基本执行
Bash command="ls -la"

# 指定超时
Bash command="npm install" timeout=60000

# 指定工作目录
Bash command="npm test" cwd="/path/to/project"
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `command` | string | 是 | Shell 命令 |
| `timeout` | number | 否 | 超时（毫秒） |
| `cwd` | string | 否 | 工作目录 |

---

## B.5 智能代理工具

### B.5.1 Agent (AgentTool)

启动子代理。

```bash
Agent prompt="分析代码库架构"
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `prompt` | string | 是 | 任务描述 |
| `agent` | string | 否 | 代理类型 |
| `model` | string | 否 | 使用模型 |

### B.5.2 TaskOutput (TaskOutputTool)

获取任务输出。

```bash
TaskOutput task_id="task-123" block=false timeout=5000
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `task_id` | string | 是 | 任务 ID |
| `block` | boolean | 否 | 是否阻塞等待 |
| `timeout` | number | 否 | 超时时间 |

### B.5.3 TaskStop (TaskStopTool)

停止运行中的任务。

```bash
TaskStop task_id="task-123"
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `task_id` | string | 是 | 任务 ID |

---

## B.6 任务管理工具

### B.6.1 TodoWrite (TodoWriteTool)

写入任务列表。

```bash
TodoWrite content="完成第一版实现" status="in_progress"
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `content` | string | 是 | 任务内容 |
| `status` | string | 否 | 状态 |

### B.6.2 TaskCreate (TaskCreateTool) ⚡ 条件可用

创建任务（需要 TodoV2 启用）。

```bash
TaskCreate subject="新任务" description="任务描述"
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `subject` | string | 是 | 任务主题 |
| `description` | string | 否 | 任务描述 |

### B.6.3 TaskGet (TaskGetTool) ⚡ 条件可用

获取任务详情。

```bash
TaskGet task_id="task-123"
```

### B.6.4 TaskUpdate (TaskUpdateTool) ⚡ 条件可用

更新任务。

```bash
TaskUpdate task_id="task-123" status="completed"
```

### B.6.5 TaskList (TaskListTool) ⚡ 条件可用

列出任务。

```bash
TaskList
```

---

## B.7 用户交互工具

### B.7.1 AskUserQuestion (AskUserQuestionTool)

向用户提问。

```bash
AskUserQuestion question="选择语言" options=[{"label":"TypeScript","description":"推荐使用"}] multi_select=false
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `question` | string | 是 | 问题 |
| `options` | array | 是 | 选项 |
| `multi_select` | boolean | 否 | 是否多选 |

---

## B.8 计划与模式工具

### B.8.1 EnterPlanMode (EnterPlanModeTool)

进入计划模式。

```bash
EnterPlanMode
```

### B.8.2 ExitPlanMode (ExitPlanModeV2Tool)

退出计划模式。

```bash
ExitPlanMode
```

---

## B.9 Web 工具

### B.9.1 WebFetch (WebFetchTool)

获取网页内容。

```bash
WebFetch url="https://example.com"
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `url` | string | 是 | 网址 |

### B.9.2 WebSearch (WebSearchTool)

网络搜索。

```bash
WebSearch query="最新技术新闻" num_results=5
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `query` | string | 是 | 搜索查询 |
| `num_results` | number | 否 | 结果数量 |

### B.9.3 WebBrowser (WebBrowserTool) ⚡ 条件可用

浏览器操作（需要 `feature('WEB_BROWSER_TOOL')`）。

```bash
WebBrowser url="https://example.com" action="click" selector=".button"
```

---

## B.10 技能工具

### B.10.1 Skill (SkillTool)

执行技能。

```bash
Skill skill="skill-name" args={"key":"value"}
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `skill` | string | 是 | 技能名称 |
| `args` | object | 否 | 技能参数 |

---

## B.11 MCP 工具

### B.11.1 ListMcpResources (ListMcpResourcesTool)

列出 MCP 资源。

```bash
ListMcpResources
```

### B.11.2 ReadMcpResource (ReadMcpResourceTool)

读取 MCP 资源。

```bash
ReadMcpResource uri="mcp://github/repos"
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `uri` | string | 是 | 资源 URI |

---

## B.12 团队协作工具 ⚡ 条件可用

### B.12.1 TeamCreate (TeamCreateTool)

创建团队（需要 `isAgentSwarmsEnabled()`）。

```bash
TeamCreate team_name="my-team"
```

### B.12.2 TeamDelete (TeamDeleteTool)

删除团队。

```bash
TeamDelete team_id="team-123"
```

### B.12.3 SendMessage (SendMessageTool)

发送消息。

```bash
SendMessage team_id="team-123" message="开始任务"
```

---

## B.13 定时任务工具 ⚡ 条件可用

### B.13.1 CronCreate (CronCreateTool)

创建定时任务（需要 `feature('AGENT_TRIGGERS')`）。

```bash
CronCreate cron="0 9 * * *" prompt="检查代码"
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `cron` | string | 是 | Cron 表达式 |
| `prompt` | string | 是 | 任务描述 |
| `recurring` | boolean | 否 | 是否重复 |

### B.13.2 CronDelete (CronDeleteTool)

删除定时任务。

```bash
CronDelete id="cron-123"
```

### B.13.3 CronList (CronListTool)

列出定时任务。

```bash
CronList
```

---

## B.14 工作流工具 ⚡ 条件可用

### B.14.1 Workflow (WorkflowTool)

执行工作流（需要 `feature('WORKFLOW_SCRIPTS')`）。

```bash
Workflow workflow_id="deploy" params={"env":"prod"}
```

---

## B.15 其他工具

### B.15.1 Brief (BriefTool)

生成简短摘要。

```bash
Brief content="很长的内容..."
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `content` | string | 是 | 内容 |

### B.15.2 Config (ConfigTool) ⚡ Ant 构建专用

查看/修改配置。

```bash
Config action="get" key="model"
```

---

## B.16 工具返回值

### B.16.1 成功返回

```typescript
// 实际 ToolResult 定义 (Tool.ts)
type ToolResult<T> = {
  data: T                   // 结果数据
  newMessages?: (           // 可选的新消息
    | UserMessage
    | AssistantMessage
    | SystemMessage
  )[]
  contextModifier?: (context: ToolUseContext) => ToolUseContext
  mcpMeta?: {              // MCP 协议元数据
    _meta?: Record<string, unknown>
    structuredContent?: Record<string, unknown>
  }
}
```

### B.16.2 错误返回

```typescript
// 实际 ValidationResult 类型 (Tool.ts)
type ValidationResult =
  | { result: true }
  | {
      result: false
      message: string    // 错误消息
      errorCode: number // 错误代码
    }

// 工具执行失败时，ToolResult 中 data 字段可能包含错误信息
// 具体格式取决于各个工具的实现
```

---

## B.17 工具状态标记说明

| 标记 | 含义 |
|------|------|
| ⚡ **条件可用** | 需要特定 feature flag、环境变量或配置才能使用 |
| 无标记 | **始终可用**的基础工具 |

---

*© 本书由 Claude Code 辅助撰写*
