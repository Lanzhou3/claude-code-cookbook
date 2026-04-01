# 《从入门到精通 Claude Code》

> 终端 AI 编程助手的全面指南

---

## 快速导航

### [第一部分：入门篇](./chapters/part1-入门篇/)
- [第1章 Claude Code 概述](./chapters/part1-入门篇/第1章-Claude-Code-概述.md)
- [第2章 安装与配置](./chapters/part1-入门篇/第2章-安装与配置.md)
- [第3章 初次使用](./chapters/part1-入门篇/第3章-初次使用.md)
- [第4章 核心概念速览](./chapters/part1-入门篇/第4章-核心概念速览.md)

### [第二部分：日常使用篇](./chapters/part2-日常使用篇/)
- [第5章 文件操作全指南](./chapters/part2-日常使用篇/第5章-文件操作全指南.md)
- [第6章 Shell 命令执行](./chapters/part2-日常使用篇/第6章-Shell-命令执行.md)
- [第7章 搜索与导航](./chapters/part2-日常使用篇/第7章-搜索与导航.md)
- [第8章 技能系统详解](./chapters/part2-日常使用篇/第8章-技能系统详解.md)
- [第9章 MCP 协议与外部集成](./chapters/part2-日常使用篇/第9章-MCP-协议与外部集成.md)

### [第三部分：进阶配置篇](./chapters/part3-进阶配置篇/)
- [第10章 权限系统深入](./chapters/part3-进阶配置篇/第10章-权限系统深入.md)
- [第11章 上下文管理策略](./chapters/part3-进阶配置篇/第11章-上下文管理策略.md)
- [第12章 会话与记忆系统](./chapters/part3-进阶配置篇/第12章-会话与记忆系统.md)
- [第13章 钩子与自动化](./chapters/part3-进阶配置篇/第13章-钩子与自动化.md)
- [第14章 配置与个性化](./chapters/part3-进阶配置篇/第14章-配置与个性化.md)

### [第四部分：架构原理篇](./chapters/part4-架构原理篇/)
- [第15章 系统架构与设计思想](./chapters/part4-架构原理篇/第15章-系统架构与设计思想.md) ⭐
- [第16章 查询引擎与消息管道](./chapters/part4-架构原理篇/第16章-查询引擎与消息管道.md)
- [第17章 工具系统设计与实现](./chapters/part4-架构原理篇/第17章-工具系统设计与实现.md)
- [第18章 MCP 协议架构](./chapters/part4-架构原理篇/第18章-MCP-协议架构.md)
- [第19章 状态管理与渲染](./chapters/part4-架构原理篇/第19章-状态管理与渲染.md)

### [第五部分：扩展开发篇](./chapters/part5-扩展开发篇/)
- [第20章 插件系统](./chapters/part5-扩展开发篇/第20章-插件系统.md)
- [第21章 创建自定义技能](./chapters/part5-扩展开发篇/第21章-创建自定义技能.md)
- [第22章 构建 MCP 服务器](./chapters/part5-扩展开发篇/第22章-构建-MCP-服务器.md)
- [第23章 贡献与二次开发](./chapters/part5-扩展开发篇/第23章-贡献与二次开发.md)
- [第24章 隐藏功能与高级彩蛋](./chapters/part5-扩展开发篇/第24章-隐藏功能与高级彩蛋.md)

### [附录](./appendix/)
- [附录 A 命令参考](./appendix/附录A-命令参考.md)
- [附录 B 工具参考](./appendix/附录B-工具参考.md)
- [附录 C API 参考](./appendix/附录C-API-参考.md)
- [附录 D 故障排除](./appendix/附录D-故障排除.md)
- [附录 E 架构决策记录](./appendix/附录E-架构决策记录.md)

---

## 书籍概述

本书系统介绍 Claude Code CLI，从基础使用到高级定制，涵盖架构原理、扩展开发、工程实践。

### 目标读者

- **入门读者**：从未使用过 Claude Code 的开发者
- **进阶读者**：已有使用经验，想深入了解内部原理
- **高级读者**：想基于 Claude Code 进行二次开发或贡献

### 技术栈

| 层级 | 技术 |
|------|------|
| 运行时 | Bun |
| UI 框架 | React + 自定义 Ink |
| 类型系统 | TypeScript + Zod v4 |
| AI SDK | @anthropic-ai/sdk |
| MCP 协议 | @modelcontextprotocol/sdk |

---

## 目录结构

```
cookbook/
├── README.md                 # 本文件
├── chapters/                # 正文章节
│   ├── part1-入门篇/        # 第一部分：入门
│   ├── part2-日常使用篇/    # 第二部分：日常使用
│   ├── part3-进阶配置篇/    # 第三部分：进阶配置
│   ├── part4-架构原理篇/    # 第四部分：架构原理
│   └── part5-扩展开发篇/    # 第五部分：扩展开发
├── appendix/                # 附录
└── assets/                  # 资源文件
```

---

*本书由 Claude Code 辅助撰写，全书共 29 章，约 285,000 字。*
