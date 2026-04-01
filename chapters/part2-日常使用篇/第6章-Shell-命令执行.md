# 第6章 Shell 命令执行

> 本章介绍 Claude Code 中执行 Shell 命令的各种方式，包括基本命令、管道重定向、环境变量管理和后台任务等。

---

## 6.1 基本命令执行

### 6.1.1 运行简单命令

```bash
> 运行 ls 查看当前目录

> 执行 pwd 显示当前路径

> 运行 git status 查看 Git 状态
```

### 6.1.2 命令输出格式

```
┌────────────────────────────────────────┐
  $ ls -la
────────────────────────────────────────┤
drwxr-xr-x  12 user  staff   384 Apr  1 10:00 .
drwxr-xr-x   8 user  staff   256 Mar 31 09:30 ..
-rw-r--r--   1 user  staff   220 Apr  1 10:00 README.md
drwxr-xr-x   3 user  staff    96 Apr  1 09:45 src
drwxr-xr-x   2 user  staff    64 Apr  1 09:45 tests
└────────────────────────────────────────┘

✨ 命令执行成功 (0.23s)
```

### 6.1.3 执行多个命令

```bash
# 顺序执行
> 先运行 npm install，然后运行 npm test

# 组合命令
> 创建目录并进入：mkdir new-project && cd new-project
```

---

## 6.2 权限管理

### 6.2.1 权限提示

执行 Shell 命令时，Claude Code 会根据命令风险级别进行权限检查：

```
> 运行 rm -rf node_modules

⚠️  危险操作确认：
  命令: rm -rf node_modules
  影响: 将删除 node_modules 目录及所有依赖

  [y] 确认执行  [n] 拒绝  [a] 允许所有类似操作
```

### 6.2.2 风险级别

| 级别 | 命令示例 | 默认行为 |
|------|----------|----------|
| **low** | `ls`, `pwd`, `git status` | 自动执行 |
| **medium** | `npm install`, `git add` | 确认后执行 |
| **high** | `rm -rf`, `chmod` | 明确确认 |
| **critical** | `sudo`, `dd` | 拒绝执行 |

### 6.2.3 权限配置

```json
{
  "permissions": {
    "mode": "acceptEdits",
    "shell": {
      "allow": ["npm", "git", "docker"],
      "deny": ["rm -rf /*", "dd"]
    }
  }
}
```

---

## 6.3 管道和重定向

### 6.3.1 使用管道

```bash
> 查找包含 "error" 的日志并显示前 10 行
  cat app.log | grep error | head -10

> 统计文件数量
  find src -name "*.ts" | wc -l

> 排序并去重
  cat names.txt | sort | uniq
```

### 6.3.2 输出重定向

```bash
> 将输出保存到文件
  echo "hello" > output.txt

> 追加到文件
  echo "world" >> output.txt

> 重定向错误输出
  command 2> error.log
```

### 6.3.3 输入重定向

```bash> 使用文件作为输入
  sort < unsorted.txt

# 结合使用
  sort < input.txt > output.txt
```

---

## 6.4 环境变量

### 6.4.1 查看环境变量

```bash
> 查看当前 PATH
  echo $PATH

> 查看所有环境变量
  env

> 查看特定变量
  echo $HOME && echo $USER
```

### 6.4.2 设置环境变量

```bash
# 临时设置（单次命令）
> API_KEY=xxx npm run build

# 设置并执行
> MY_VAR=hello && echo $MY_VAR

# 多变量
> PORT=3000 NODE_ENV=production npm start
```

### 6.4.3 常用变量

| 变量 | 用途 | Claude Code 使用 |
|------|------|-----------------|
| `PATH` | 命令搜索路径 | 执行系统命令 |
| `HOME` | 用户主目录 | 解析 `~` 路径 |
| `SHELL` | 默认 shell | 执行脚本 |
| `LANG` | 语言设置 | 输出编码 |
| `ANTHROPIC_API_KEY` | API 密钥 | Claude 认证 |

---

## 6.5 后台任务

### 6.5.1 后台执行

```bash
> 在后台启动开发服务器
  npm run dev &

> 后台执行长时间任务
  npm run build &
```

### 6.5.2 任务控制

```bash
# 查看后台任务
> jobs

# 将后台任务调到前台
> fg

# 切换到指定任务
> fg %1

# 挂起当前任务
> Ctrl+Z

# 继续后台执行
> bg
```

### 6.5.3 nohup 使用

```bash
# 持久化后台任务
> nohup npm run server > server.log 2>&1 &

# 查看 nohup 进程
> ps aux | grep npm
```

---

## 6.6 进程管理

### 6.6.1 查看进程

```bash> 查看 Claude Code 进程
  ps aux | grep claude

> 查看所有 Node 进程
  ps aux | grep node

> 实时进程监控
  top
```

### 6.6.2 终止进程

```bash
# 按名称终止
> pkill node

# 按 PID 终止
> kill 1234

# 强制终止
> kill -9 1234
```

### 6.6.3 超时控制

```
> 运行 sleep 100

⏱️  命令超时设置：
  默认超时: 30 秒
  最大超时: 5 分钟

  [c] 继续等待  [s] 设置更长超时  [x] 终止
```

---

## 6.7 脚本执行

### 6.7.1 执行脚本文件

```bash
# 执行 Shell 脚本
> 运行 ./scripts/setup.sh

# 执行 Python 脚本
> python script.py

# 执行 Node 脚本
> node script.js
```

### 6.7.2 Shebang 支持

Claude Code 自动识别脚本类型：

```bash
# 如果脚本有 shebang，会自动使用正确解释器
#!/bin/bash        → bash
#!/usr/bin/python → python
#!/usr/bin/node   → node
```

### 6.7.3 脚本权限

```
> 运行 ./script.sh

⚠️  脚本无执行权限：
  文件: ./script.sh
  权限: -rw-r--r--

  [y] 添加执行权限并运行  [n] 取消
```

---

## 6.8 开发工具集成

### 6.8.1 npm/yarn/pnpm

```bash
# 安装依赖
> npm install
> yarn install
> pnpm install

# 运行脚本
> npm run dev
> yarn build
> pnpm test

# 包管理
> npm list
> npm outdated
```

### 6.8.2 Git 命令

```bash
# 基本操作
> git status
> git add .
> git commit -m "message"
> git push

# 分支操作
> git branch
> git checkout -b new-branch
> git merge feature

# 查看历史
> git log --oneline
> git diff
```

### 6.8.3 Docker

```bash
# 构建和运行
> docker build -t myapp .
> docker run -p 3000:3000 myapp

# 容器管理
> docker ps
> docker stop container_id
> docker logs container_id
```

---

## 6.9 错误处理

### 6.9.1 常见错误

```bash
# 命令不存在
$ ls: command not found
→ PATH 配置问题

# 权限拒绝
$ Permission denied
→ 需要 sudo 或修改权限

# 文件不存在
$ No such file or directory
→ 检查路径拼写
```

### 6.9.2 调试技巧

```bash
# 打印命令但不执行
> echo ls -la  # 预览命令

# 显示详细输出
> ls -la -x    # 调试模式

# 检查返回码
> command; echo "Exit code: $?"
```

### 6.9.3 超时处理

```
⏱️  命令执行超时

  命令: npm install
  已运行: 30s
  超时限制: 30s

  [c] 继续等待 60s
  [x] 终止并重试
  [i] 增加超时限制
```

---

## 6.10 安全最佳实践

### 6.10.1 危险命令避免

Claude Code 会阻止以下命令：

| 命令 | 风险 | 替代方案 |
|------|------|----------|
| `rm -rf /` | 删除系统 | `rm -rf ./target` |
| `dd if=/dev/zero` | 磁盘写入 | 指定目标文件 |
| `:(){ :\|:& };:` | Fork 炸弹 | 使用超时 |
| `curl \| sh` | 任意执行 | 下载后检查 |

### 6.10.2 安全建议

```bash
# 使用相对路径
> cd ./project && npm build

# 指定完整命令
> /usr/bin/git commit -m "msg"

# 避免环境变量泄露
> 命令中使用 $(cat secret.txt) 而非 $SECRET
```

---

## 6.11 本章小结

本章介绍了 Shell 命令执行的相关内容：

| 要点 | 内容 |
|------|------|
| **基本执行** | 单命令、组合命令、多命令顺序 |
| **权限管理** | 风险级别、权限提示、配置 |
| **管道重定向** | 管道符、输出/输入重定向 |
| **环境变量** | 查看、设置、常用变量 |
| **后台任务** | 后台执行、任务控制、nohup |
| **进程管理** | 查看、终止、超时 |
| **脚本执行** | 文件执行、权限、shebang |
| **开发集成** | npm、Git、Docker |
| **错误处理** | 常见错误、调试、超时 |
| **安全** | 危险命令、安全建议 |

---

## 下章预告

下一章我们将学习 **搜索与导航**，包括：

- 项目结构分析
- 代码导航
- 符号查找
- 引用追踪

---

*© 本书由 Claude Code 辅助撰写*
