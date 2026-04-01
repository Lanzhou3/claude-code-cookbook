# 附录 D：故障排除

> 本附录提供 Claude Code 常见问题的诊断和解决方法。

---

## D.1 安装问题

### D.1.1 命令未找到

**问题**：`claude: command not found`

**解决方法**：

```bash
# 检查安装位置
which claude
ls -la ~/.local/bin/claude

# 添加到 PATH
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# 或者使用完整路径
~/.local/bin/claude --version
```

### D.1.2 权限被拒绝

**问题**：`Permission denied` 或 `EACCES`

**解决方法**：

```bash
# 修复权限
chmod +x /usr/local/bin/claude

# 如果安装在用户目录
mkdir -p ~/.local/bin
mv claude ~/.local/bin/
chmod +x ~/.local/bin/claude
```

### D.1.3 Bun 版本问题

**问题**：`Bun version mismatch`

**解决方法**：

```bash
# 更新 Bun
curl -fsSL https://bun.sh/install | bash

# 或使用 npm
npm install -g bun

# 验证版本
bun --version
```

---

## D.2 API 配置问题

### D.2.1 API Key 无效

**问题**：`Invalid API key` 或 `Authentication failed`

**解决方法**：

```bash
# 检查 API Key 格式
echo $ANTHROPIC_API_KEY | head -c 20

# 测试 API 连接
curl -s -H "x-api-key: $ANTHROPIC_API_KEY" \
     -H "anthropic-version: 2023-06-01" \
     https://api.anthropic.com/v1/messages \
     -d '{"model":"claude-3-5-sonnet-4","max_tokens":1,"messages":[{"role":"user","content":"hi"}]}'

# 重新设置 API Key
export ANTHROPIC_API_KEY="sk-ant-xxxxx..."
```

### D.2.2 区域限制

**问题**：`Region not supported` 或 `Access denied`

**解决方法**：

```bash
# 使用支持的区域
export AWS_REGION="us-east-1"

# 或使用 Anthropic API
export ANTHROPIC_API_KEY="sk-ant-..."
```

### D.2.3 认证过期

**问题**：`Token expired` 或 `Authentication expired`

**解决方法**：

```bash
# GCP: 刷新凭证
gcloud auth application-default login

# AWS: 更新令牌
aws sso login --profile <profile>

# Azure: 重新登录
az login
```

---

## D.3 连接问题

### D.3.1 网络超时

**问题**：`Request timeout` 或 `Connection timed out`

**解决方法**：

```bash
# 设置代理
export HTTP_PROXY="http://proxy:8080"
export HTTPS_PROXY="http://proxy:8080"

# 增加超时
# 在 settings.json 中
{
  "network": {
    "timeout": {
      "request": 60000,
      "connect": 20000,
      "read": 120000
    }
  }
}

# 检查网络
curl -v https://api.anthropic.com
```

### D.3.2 SSL 证书错误

**问题**：`SSL certificate error` 或 `TLS handshake failed`

**解决方法**：

```bash
# 更新系统证书
# macOS
brew install curl-ca-bundle
export SSL_CERT_FILE=/usr/local/opt/curl-ca-bundle/share/cacert.pem

# Linux
sudo apt-get install ca-certificates
sudo update-ca-certificates

# 或禁用 SSL 验证（仅测试用）
export NODE_TLS_REJECT_UNAUTHORIZED=0
```

### D.3.3 代理问题

**问题**：`Proxy authentication failed` 或 `ECONNREFUSED`

**解决方法**：

```bash
# 配置代理认证
export HTTP_PROXY="http://user:password@proxy:8080"
export HTTPS_PROXY="http://user:password@proxy:8080"

# 或使用跳过代理的地址
export NO_PROXY="localhost,127.0.0.1,api.anthropic.com"
```

---

## D.4 权限问题

### D.4.1 权限被拒绝

**问题**：`Permission denied` 执行操作时

**解决方法**：

```bash
# 检查文件权限
ls -la /path/to/file

# 修复权限
chmod 644 /path/to/file

# 当前会话提升权限
claude --permissions acceptEdits

# 配置文件权限
{
  "permissions": {
    "mode": "acceptEdits"
  }
}
```

### D.4.2 敏感操作被阻止

**问题**：`Blocked: dangerous operation`

**解决方法**：

```bash
# 查看操作原因
claude --debug

# 临时提升权限
claude --permissions auto

# 或在配置中白名单
{
  "permissions": {
    "shell": {
      "allowedCommands": ["npm", "git"]
    }
  }
}
```

---

## D.5 性能问题

### D.5.1 启动缓慢

**问题**：`claude` 命令启动很慢

**解决方法**：

```bash
# 检查版本
claude --version

# 清理缓存
rm -rf ~/.claude/cache/*

# 禁用遥测
{
  "telemetry": {
    "enabled": false
  }
}

# 检查网络延迟
ping api.anthropic.com
```

### D.5.2 响应缓慢

**问题**：AI 响应很慢

**解决方法**：

```bash
# 使用更快的模型
claude --model haiku

# 检查 Token 使用
/token

# 清理上下文
/context optimize

# 压缩会话
/sessions compact
```

### D.5.3 内存占用高

**问题**：`Out of memory` 或内存持续增长

**解决方法**：

```bash
# 检查内存使用
ps aux | grep claude

# 清理会话
/sessions clear-old

# 减少上下文大小
{
  "context": {
    "maxMessages": 30
  }
}

# 重启 Claude Code
/exit
claude
```

---

## D.6 工具问题

### D.6.1 工具执行失败

**问题**：`Tool execution failed`

**解决方法**：

```bash
# 查看详细错误
claude --debug

# 检查工具状态
/mcp tools

# 测试 MCP 连接
/mcp test <server-name>

# 查看工具日志
tail -f ~/.claude/logs/claude.log
```

### D.6.2 工具无响应

**问题**：工具调用后无响应

**解决方法**：

```bash
# 中断当前操作
Ctrl+C

# 检查工具是否超时
/mcp test filesystem

# 重启 MCP 服务器
/mcp restart <server>
```

---

## D.7 配置问题

### D.7.1 配置文件格式错误

**问题**：`Invalid configuration` 或 `JSON parse error`

**解决方法**：

```bash
# 验证 JSON 格式
cat ~/.claude/settings.json | python3 -m json.tool

# 使用在线 JSON 验证
# https://jsonformatter.curiousconcept.com/

# 重置配置
mv ~/.claude/settings.json ~/.claude/settings.json.bak
# 重新创建默认配置
```

### D.7.2 配置不生效

**问题**：修改配置后没有效果

**解决方法**：

```bash
# 检查配置位置优先级
# 1. 命令行参数
# 2. 环境变量
# 3. 项目配置 .claude/
# 4. 用户配置 ~/.claude/

# 重启 Claude Code
/exit
claude

# 检查配置路径
echo $CLAUDE_CONFIG_DIR
```

---

## D.8 会话问题

### D.8.1 无法恢复会话

**问题**：`Session not found` 或 `Failed to resume`

**解决方法**：

```bash
# 列出可用会话
/sessions list

# 检查会话文件
ls -la ~/.claude/sessions/

# 手动恢复
claude --session <session-id>

# 修复会话文件
# 如果会话文件损坏，删除后重建
rm ~/.claude/sessions/<broken-session>
```

### D.8.2 会话数据丢失

**问题**：历史记录丢失或会话损坏

**解决方法**：

```bash
# 查找备份
ls -la ~/.claude/backups/

# 恢复备份
cp ~/.claude/backups/<backup> ~/.claude/sessions/<session-id>

# 导出会话
/sessions export <session-id> --output session.json
```

---

## D.9 诊断命令

### D.9.1 诊断清单

```bash
# 1. 检查版本
claude --version

# 2. 检查配置
claude --config validate

# 3. 检查 API 连接
curl -s https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{"model":"claude-3-5-sonnet-4-6","max_tokens":10,"messages":[{"role":"user","content":"hi"}]}'

# 4. 检查日志
tail -100 ~/.claude/logs/claude.log

# 5. 检查网络
ping api.anthropic.com
```

### D.9.2 调试模式

```bash
# 启用完整调试
claude --debug --verbose

# 查看 MCP 调试信息
/mcp debug

# 查看状态
/status

# Token 使用
/token
```

---

## D.10 获取帮助

### D.10.1 资源

| 资源 | 地址 |
|------|------|
| 官方文档 | https://docs.anthropic.com/claude-code |
| GitHub Issues | https://github.com/anthropics/claude-code/issues |
| Discord | https://discord.gg/anthropic |
| 支持邮箱 | support@anthropic.com |

### D.10.2 提交问题

```markdown
## 问题报告模板

### 环境
- Claude Code 版本: x.x.x
- 操作系统: macOS/Linux/Windows
- Bun 版本: x.x.x

### 问题描述
[详细描述问题]

### 重现步骤
1. ...
2. ...
3. ...

### 错误信息
```
[错误日志]
```

### 预期行为
[期望的结果]

### 实际行为
[实际的结果]
```

---

*© 本书由 Claude Code 辅助撰写*
