# nanobot 接口文档

nanobot 提供多种接口：CLI 命令、OpenAI 兼容 HTTP API、Python SDK 和 WebSocket 协议。

---

## CLI 命令

CLI 入口：`nanobot`（由 `nanobot/cli/commands.py` 提供）

### `nanobot chat`（默认）

启动交互式 REPL 聊天界面。

```bash
nanobot chat
nanobot chat --workspace /path/to/project
nanobot chat --model claude-sonnet-4-20250514
```

选项：
- `--workspace, -w`：工作区路径
- `--config, -c`：配置文件路径（默认 `~/.nanobot/config.json`）
- `--model, -m`：模型名称覆盖
- `--model-preset, -p`：模型预设覆盖

### `nanobot webui`

启动 WebUI（自动构建前端、启动 gateway 并打开浏览器）。

```bash
nanobot webui
nanobot webui --port 5173 --gateway-port 8765
```

选项：
- `--port`：WebUI 开发服务器端口（默认 5173）
- `--gateway-port`：Gateway 端口（默认 8765）
- `--workspace`：工作区路径
- `--background`：后台运行
- `--no-open`：不自动打开浏览器

### `nanobot gateway`

管理 gateway 服务。

```bash
# 前台启动
nanobot gateway --foreground

# 后台启动
nanobot gateway --background

# 状态查看
nanobot gateway status

# 查看日志
nanobot gateway logs

# 停止
nanobot gateway stop

# 安装为系统服务
nanobot gateway install-service
```

选项：
- `--port, -p`：Gateway 端口
- `--foreground` / `--background`：运行模式（互斥）
- `--verbose, -v`：详细日志

### `nanobot serve`

启动 OpenAI 兼容 HTTP API 服务器。

```bash
nanobot serve --port 8900
```

### `nanobot models`

列出可用模型和预设。

```bash
nanobot models
```

### `nanobot onboard`

交互式入门配置向导。

```bash
nanobot onboard
```

### `nanobot compact`

手动压缩会话上下文。

```bash
nanobot compact <session_key>
```

---

## OpenAI 兼容 HTTP API

由 `nanobot/api/server.py` 提供，使用 aiohttp 实现。

### 基础 URL

```
http://localhost:8900/v1
```

### 端点

#### `POST /v1/chat/completions`

创建聊天完成。支持流式和非流式。

**请求体**（OpenAI 兼容格式）：

```json
{
  "model": "claude-sonnet-4-20250514",
  "messages": [
    {"role": "user", "content": "Hello, summarize this repo"}
  ],
  "stream": true,
  "max_tokens": 4096
}
```

**响应**（非流式）：

```json
{
  "id": "chatcmpl-xxx",
  "object": "chat.completion",
  "created": 1717200000,
  "model": "claude-sonnet-4-20250514",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "This repository is..."
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 100,
    "completion_tokens": 200,
    "total_tokens": 300
  }
}
```

**媒体文件支持**：可通过 base64 data URL 上传图片等媒体文件。

**认证**：支持 API Key 验证（如配置了 `api.api_key`）。

#### `GET /v1/models`

列出可用模型。

**响应**：

```json
{
  "object": "list",
  "data": [
    {
      "id": "claude-sonnet-4-20250514",
      "object": "model",
      "created": 1717200000,
      "owned_by": "anthropic"
    }
  ]
}
```

---

## Python SDK

由 `nanobot/nanobot.py` 提供 `Nanobot` 类作为高级编程入口。

### 基本用法

```python
from nanobot import Nanobot, RunResult, RunStream

# 从配置文件创建实例
bot = Nanobot.from_config()
# 或指定配置路径和工作区
bot = Nanobot.from_config(
    config_path="~/.nanobot/config.json",
    workspace="/path/to/project",
)

# 发送消息（非流式）
result: RunResult = await bot.run("Summarize this repo")
print(result.content)

# 流式发送
async with bot.stream("Write a README") as stream:
    async for event in stream:
        if event.type == "text_delta":
            print(event.text, end="")

# 带工具调用钩子
async def on_tool(events):
    for ev in events:
        print(f"Tool: {ev['name']} -> {ev.get('result')}")

result = await bot.run("List files", hooks=[tool_hook(on_tool)])
```

### 客户端

```python
# 会话管理
sessions = bot.sessions.list()
session = bot.sessions.get(key)

# 记忆管理
memories = bot.memory.read()
bot.memory.append("New knowledge")

# 运行时信息
info = bot.runtime.info()
```

### 流式事件类型

| 事件类型 | 描述 |
|----------|------|
| `run_started` | 运行开始 |
| `text_delta` | 文本增量 |
| `text_completed` | 文本完成 |
| `reasoning_delta` | 推理增量 |
| `reasoning_completed` | 推理完成 |
| `tool_started` | 工具调用开始 |
| `tool_completed` | 工具调用完成 |
| `tool_failed` | 工具调用失败 |
| `run_completed` | 运行完成 |
| `run_failed` | 运行失败 |

---

## WebSocket 协议

WebUI 通过 WebSocket 多路复用协议与 gateway 通信。网关地址：`ws://localhost:8765/ws`。

### 协议概述

WebSocket 协议使用 JSON 消息，每条消息包含以下字段：

**请求消息格式**：
```json
{
  "id": "unique-request-id",
  "type": "request_type",
  "payload": {}
}
```

**响应消息格式**：
```json
{
  "id": "unique-response-id",
  "type": "response_type",
  "payload": {},
  "error": null
}
```

### 消息类型

#### `chat.send`

发送聊天消息。

```json
{
  "id": "1",
  "type": "chat.send",
  "payload": {
    "session_key": "default",
    "message": {"role": "user", "content": "Hello"},
    "model": "claude-sonnet-4-20250514"
  }
}
```

#### `chat.cancel`

取消当前聊天请求。

```json
{
  "id": "2",
  "type": "chat.cancel",
  "payload": {"session_key": "default"}
}
```

#### `session.list`

获取会话列表。

```json
{
  "id": "3",
  "type": "session.list",
  "payload": {"limit": 50}
}
```

#### `session.delete`

删除会话。

```json
{
  "id": "4",
  "type": "session.delete",
  "payload": {"session_key": "xxx"}
}
```

#### `settings.get` / `settings.set`

获取/更新运行时设置。

#### `skills.get`

获取可用技能列表。

#### `workspace.*`

工作区文件操作相关消息。

### 服务端推送事件

服务端通过同一 WebSocket 通道推送流式事件：

```json
{
  "type": "stream.delta",
  "payload": {
    "session_key": "default",
    "delta": {"type": "text_delta", "text": "...", "index": 0}
  }
}
```

---

## Channel 接口

### BaseChannel 抽象

所有渠道实现 `nanobot/channels/base.py` 中的 `BaseChannel`（ABC）：

```python
class BaseChannel(ABC):
    name: str           # 渠道标识
    display_name: str   # 显示名称

    async def start(self) -> None: ...
    async def stop(self) -> None: ...
    async def send(self, recipient: str, text: str, ...) -> None: ...
    async def login(self) -> bool: ...
```

### Channel Manifest

每个渠道通过 `manifest.py` 声明元数据：

```python
# nanobot/channels/telegram/manifest.py
MANIFEST = {
    "name": "telegram",
    "display_name": "Telegram",
    "description": "Telegram Bot integration",
    "requires": ["python-telegram-bot"],
}
```

ChannelManager 通过 pkgutil 自动发现所有渠道，并根据配置启用/禁用。

---

## Provider 接口

### LLMProvider 抽象

所有 LLM 提供商实现 `nanobot/providers/base.py` 中的 `LLMProvider`（ABC）：

```python
class LLMProvider(ABC):
    async def send_message(self, messages, settings) -> LLMResponse: ...
    async def send_message_stream(self, messages, settings) -> AsyncIterator[str | ToolCallRequest]: ...
    def tool_definitions_to_schema(self, tools) -> Any: ...
```

### 关键数据结构

```python
@dataclass
class ToolCallRequest:
    id: str
    name: str
    arguments: Any

@dataclass
class LLMResponse:
    content: str
    tool_calls: list[ToolCallRequest]
    usage: TokenUsage

@dataclass
class GenerationSettings:
    model: str
    max_tokens: int
    temperature: float
    tools: list[dict]
```
