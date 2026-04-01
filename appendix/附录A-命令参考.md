# 附录 A：命令参考

> 本附录列出 Claude Code 所有可用的命令，包括 slash 命令和快捷键。

---

## A.1 Slash 命令

### A.1.1 基础命令

| 命令 | 描述 | 用法 |
|------|------|------|
| `/help` | 显示帮助 | `/help [topic]` |
| `/exit` | 退出 Claude Code | `/exit` |
| `/clear` | 清屏 | `/clear` |
| `/resume` | 恢复会话 | `/resume [session-id]` |
| `/history` | 查看历史 | `/history` |

### A.1.2 会话命令

| 命令 | 描述 | 用法 |
|------|------|------|
| `/sessions` | 会话管理 | `/sessions list` |
| `/sessions save` | 保存会话 | `/sessions save [name]` |
| `/sessions switch` | 切换会话 | `/sessions switch <id>` |
| `/bookmark` | 会话书签 | `/bookmark [list/add]` |

### A.1.3 权限命令

| 命令 | 描述 | 用法 |
|------|------|------|
| `/permissions` | 权限管理 | `/permissions status` |
| `/permissions mode` | 设置模式 | `/permissions mode <mode>` |
| `/allow` | 允许操作 | `/allow <operation>` |
| `/deny` | 拒绝操作 | `/deny <operation>` |

### A.1.4 上下文命令

| 命令 | 描述 | 用法 |
|------|------|------|
| `/context` | 上下文管理 | `/context status` |
| `/context optimize` | 优化上下文 | `/context optimize` |
| `/context dump` | 导出上下文 | `/context dump` |
| `/token` | Token 统计 | `/token` |

### A.1.5 记忆命令

| 命令 | 描述 | 用法 |
|------|------|------|
| `/remember` | 保存记忆 | `/remember <content>` |
| `/memory` | 记忆管理 | `/memory list` |
| `/memory load` | 加载记忆 | `/memory load <id>` |
| `/memory search` | 搜索记忆 | `/memory search <query>` |

### A.1.6 项目命令

| 命令 | 描述 | 用法 |
|------|------|------|
| `/project` | 项目管理 | `/project init` |
| `/project sync` | 同步项目 | `/project sync` |
| `/project info` | 项目信息 | `/project info` |

### A.1.7 Git 命令

| 命令 | 描述 | 用法 |
|------|------|------|
| `/git` | Git 操作 | `/git status` |
| `/git status` | 状态 | `/git status` |
| `/git commit` | 提交 | `/git commit -m <msg>` |
| `/git push` | 推送 | `/git push` |

### A.1.8 MCP 命令

| 命令 | 描述 | 用法 |
|------|------|------|
| `/mcp` | MCP 管理 | `/mcp status` |
| `/mcp tools` | 列出工具 | `/mcp tools` |
| `/mcp resources` | 列出资源 | `/mcp resources` |
| `/mcp test` | 测试连接 | `/mcp test <server>` |

### A.1.9 钩子命令

| 命令 | 描述 | 用法 |
|------|------|------|
| `/hooks` | 钩子管理 | `/hooks list` |
| `/hooks enable` | 启用钩子 | `/hooks enable <hook>` |
| `/hooks disable` | 禁用钩子 | `/hooks disable <hook>` |
| `/hooks test` | 测试钩子 | `/hooks test <hook>` |

### A.1.10 技能命令

| 命令 | 描述 | 用法 |
|------|------|------|
| `/skill` | 使用技能 | `/skill <name>` |
| `/skills list` | 列出技能 | `/skills list` |
| `/skills install` | 安装技能 | `/skills install <name>` |

### A.1.11 配置命令

| 命令 | 描述 | 用法 |
|------|------|------|
| `/config` | 配置管理 | `/config show` |
| `/config set` | 设置配置 | `/config set <key> <value>` |
| `/config export` | 导出配置 | `/config export` |
| `/config import` | 导入配置 | `/config import <file>` |

### A.1.12 主题命令

| 命令 | 描述 | 用法 |
|------|------|------|
| `/theme` | 主题管理 | `/theme list` |
| `/theme set` | 设置主题 | `/theme set <name>` |
| `/theme preview` | 预览主题 | `/theme preview <name>` |

### A.1.13 模型命令

| 命令 | 描述 | 用法 |
|------|------|------|
| `/model` | 模型管理 | `/model list` |
| `/model set` | 设置模型 | `/model set <name>` |

---

## A.2 快捷键

### A.2.1 REPL 快捷键

| 快捷键 | 功能 | 说明 |
|--------|------|------|
| `Enter` | 发送消息 | 发送当前输入 |
| `Shift+Enter` | 换行 | 输入多行 |
| `Ctrl+C` | 取消 | 取消当前操作 |
| `Ctrl+L` | 清屏 | 清除屏幕 |
| `Ctrl+D` | 退出 | 退出 Claude Code |
| `Tab` | 补全 | 自动补全 |
| `↑` | 上一个 | 浏览历史上一条 |
| `↓` | 下一个 | 浏览历史下一条 |
| `Ctrl+R` | 搜索 | 搜索历史 |
| `Ctrl+C` | 中断 | 中断长时间操作 |

### A.2.2 Vim 模式

| 快捷键 | 功能 | 说明 |
|--------|------|------|
| `i` | 插入模式 | 进入插入模式 |
| `Esc` | 命令模式 | 退出插入模式 |
| `h/j/k/l` | 移动 | 左/下/上/右 |
| `w` | 单词跳转 | 下一个单词 |
| `b` | 后跳 | 上一个单词 |
| `0` | 行首 | 跳到行首 |
| `$` | 行尾 | 跳到行尾 |
| `/` | 搜索 | 正向搜索 |
| `?` | 搜索 | 反向搜索 |
| `n` | 下一个 | 下一匹配 |
| `N` | 上一个 | 上一匹配 |

---

## A.3 命令行参数

### A.3.1 全局参数

| 参数 | 描述 | 用法 |
|------|------|------|
| `--version` | 显示版本 | `claude --version` |
| `--help` | 显示帮助 | `claude --help` |
| `--model` | 指定模型 | `claude --model opus` |
| `--permissions` | 设置权限 | `claude --permissions acceptEdits` |
| `--debug` | 调试模式 | `claude --debug` |

### A.3.2 会话参数

| 参数 | 描述 | 用法 |
|------|------|------|
| `--resume` | 恢复会话 | `claude --resume` |
| `--session` | 指定会话 | `claude --session <id>` |
| `--history` | 查看历史 | `claude --history` |

### A.3.3 输出参数

| 参数 | 描述 | 用法 |
|------|------|------|
| `--output` | 输出格式 | `claude --output json` |
| `--quiet` | 安静模式 | `claude --quiet` |
| `--verbose` | 详细输出 | `claude --verbose` |

---

## A.4 环境变量

### A.4.1 API 配置

| 变量 | 描述 | 示例 |
|------|------|------|
| `ANTHROPIC_API_KEY` | API 密钥 | `sk-ant-...` |
| `ANTHROPIC_BASE_URL` | API 地址 | `https://api.anthropic.com` |
| `ANTHROPIC_VERSION` | API 版本 | `2023-06-01` |

### A.4.2 AWS Bedrock

| 变量 | 描述 | 示例 |
|------|------|------|
| `AWS_REGION` | AWS 区域 | `us-east-1` |
| `AWS_ACCESS_KEY_ID` | 访问密钥 | `AKIA...` |
| `AWS_SECRET_ACCESS_KEY` | 秘密密钥 | `...` |
| `AWS_PROFILE` | AWS Profile | `claude-user` |

### A.4.3 GCP Vertex AI

| 变量 | 描述 | 示例 |
|------|------|------|
| `ANTHROPIC_VERTEX_PROJECT_ID` | GCP 项目 | `my-project` |
| `GCP_REGION` | GCP 区域 | `us-central1` |
| `GOOGLE_APPLICATION_CREDENTIALS` | 凭证文件 | `/path/to/key.json` |

### A.4.4 Azure Foundry

| 变量 | 描述 | 示例 |
|------|------|------|
| `ANTHROPIC_FOUNDRY_RESOURCE` | 资源 URL | `https://...azurefoundry.ai` |
| `ANTHROPIC_FOUNDRY_TOKEN` | 访问令牌 | `...` |

### A.4.5 网络代理

| 变量 | 描述 | 示例 |
|------|------|------|
| `HTTP_PROXY` | HTTP 代理 | `http://proxy:8080` |
| `HTTPS_PROXY` | HTTPS 代理 | `http://proxy:8080` |
| `NO_PROXY` | 跳过代理 | `localhost,127.0.0.1` |

---

*© 本书由 Claude Code 辅助撰写*
