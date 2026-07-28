# nanobot 文档

nanobot 是一个轻量级开源 AI Agent 框架，使用 Python 3.11+ 和 React/TypeScript 构建。本文档涵盖系统架构、开发流程、API 接口和核心概念。

**快速链接**: [架构](./ARCHITECTURE.md) | [接口](./INTERFACES.md) | [开发者指南](./DEVELOPER_GUIDE.md)

---

## 核心文档

### [架构](./ARCHITECTURE.md)
系统设计、技术栈、组件结构和数据流程。描述 AgentLoop、MessageBus、Providers、Channels、Session 和 Memory 子系统的协作方式。从这里开始了解系统如何运作。

### [接口](./INTERFACES.md)
公开接口文档：CLI 命令（chat, webui, gateway, serve 等）、OpenAI 兼容 HTTP API（`/v1/chat/completions`, `/v1/models`）、Python SDK（Nanobot 类）、WebSocket 多路复用协议和 Channel/Provider 扩展接口。

### [开发者指南](./DEVELOPER_GUIDE.md)
环境搭建、开发工作流、编码规范和常见任务（添加 Provider、Channel、Tool、WebUI 功能）。贡献者必读。

---

## 模块

| 模块 | 描述 | 文档 |
|------|------|------|
| `nanobot/agent/` | Agent 核心引擎：消息处理、LLM 对话、工具调度、记忆管理 | [Agent](./模块/Agent.md) |
| `nanobot/providers/` | LLM 提供商层：Anthropic, OpenAI, Azure, Bedrock, Grok 等 7 种提供商 | [Providers](./模块/Providers.md) |
| `nanobot/agent/tools/` | Agent 工具集：文件操作、Shell 执行、Web 搜索、MCP 等 20+ 种工具 | [Tools](./模块/Tools.md) |
| `nanobot/channels/` | 聊天渠道集成：Telegram, Discord, 微信, 飞书等 17 种平台的桥接层 | [Channels](./模块/Channels.md) |
| `webui/` | React/TypeScript Web 聊天界面：流式 Markdown 渲染、文件上传、多语言 | [WebUI](./模块/WebUI.md) |

---

## 核心概念

理解这些领域概念有助于导航代码库：

| 概念 | 描述 |
|------|------|
| [AgentLoop](./专有概念/AgentLoop.md) | 核心引擎：消费消息、调度回合、协调 LLM 对话、管理生命周期 |
| [MessageBus](./专有概念/MessageBus.md) | 消息中枢：通过两个 asyncio.Queue 解耦 Channel 和 Agent，实现异步 pub/sub |
| [Session](./专有概念/Session.md) | 会话管理：持久化对话历史、LRU 缓存、上下文压缩、会话分叉 |
| [Memory](./专有概念/Memory.md) | 长期记忆：MemoryStore 文件 I/O 持久化 + Consolidator Dream 两阶段整合 |
| [Channel](./专有概念/Channel.md) | 聊天平台桥接：17 种平台的集成层，消息格式转换和生命周期管理 |

---

## 入门指南

### 项目新人

按此路径学习：
1. **[架构](./ARCHITECTURE.md)** - 了解系统整体设计
2. **[核心概念](#核心概念)** - 学习领域术语和关键抽象
3. **[开发者指南](./DEVELOPER_GUIDE.md)** - 搭建开发环境
4. **[接口](./INTERFACES.md)** - 探索公开 API

### 需要集成

1. **[接口](./INTERFACES.md)** - API 契约和认证方式
2. **[架构](./ARCHITECTURE.md)** - 系统边界和数据流

### 首次贡献

1. **[开发者指南](./DEVELOPER_GUIDE.md)** - 搭建环境和工作流
2. **[常见任务](./DEVELOPER_GUIDE.md#常见任务)** - 分步操作指南
3. **[架构约束](./DEVELOPER_GUIDE.md#项目架构约束)** - 关键设计约束

---

## 快速参考

### 命令

```bash
nanobot chat                 # 交互式 REPL
nanobot webui                # 启动 WebUI（含 gateway）
nanobot gateway --background # 后台 gateway
nanobot gateway status       # 查看状态
nanobot serve                # HTTP API 服务器
nanobot models               # 列出模型
nanobot onboard              # 配置向导
cd webui && bun run dev      # WebUI 开发服务器
pytest tests/ -v             # 运行测试
ruff check nanobot/          # 代码检查
```

### 重要文件

| 文件 | 目的 |
|------|------|
| `nanobot/cli/commands.py` | CLI 主程序（3014 行） |
| `nanobot/agent/loop.py` | Agent 核心引擎（2040 行） |
| `nanobot/agent/runner.py` | LLM 对话执行器（1505 行） |
| `nanobot/agent/memory.py` | 记忆系统（1210 行） |
| `nanobot/session/manager.py` | 会话管理（967 行） |
| `nanobot/providers/base.py` | LLM 提供商抽象（994 行） |
| `nanobot/channels/base.py` | 渠道基类（298 行） |
| `nanobot/bus/queue.py` | 消息总线（44 行） |
| `pyproject.toml` | 项目元数据和依赖 |
| `webui/src/App.tsx` | WebUI 主应用（2177 行） |
