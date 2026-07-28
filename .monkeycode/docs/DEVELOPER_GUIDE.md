# nanobot 开发者指南

## 项目目的

nanobot 是一个轻量级个人 AI 助手框架，允许开发者部署一个可通过多种聊天渠道（Telegram、Discord、Slack、微信、钉钉等）交互的 AI Agent，同时提供 WebUI 界面和 OpenAI 兼容 HTTP API。

**核心职责**:
- 接收和发送多渠道聊天消息
- 调用 LLM 提供商生成智能回复
- 执行工具（文件操作、Shell 命令、Web 搜索等）
- 持久化和管理对话会话
- 通过 Dream 机制实现长期记忆学习

## 环境搭建

### 前置条件

- Python >= 3.11
- Node.js >= 24（仅 WebUI 开发）
- bun（推荐用于 WebUI 构建，也可用 npm）
- git

### 安装

```bash
# 克隆仓库
git clone <repo-url>
cd nanobot

# 安装 Python 依赖
pip install --break-system-packages -e ".[dev,api]"

# 安装 WebUI 依赖
cd webui && bun install && cd -
```

### 配置

配置存放在 `~/.nanobot/config.json`。运行初始化向导：

```bash
nanobot onboard
```

或手动创建配置文件。关键环境变量：

| 变量 | 必需 | 描述 | 示例 |
|------|------|------|------|
| `ANTHROPIC_API_KEY` | 否 | Anthropic API 密钥 | `sk-ant-...` |
| `OPENAI_API_KEY` | 否 | OpenAI API 密钥 | `sk-...` |
| `NANOBOT_CHANNELS` | 否 | 启用的渠道 | `telegram,discord` |
| `NANOBOT_STREAM_IDLE_TIMEOUT_S` | 否 | 流式空闲超时（秒） | `90` |

### 运行

```bash
# 交互式 REPL
nanobot chat

# 启动 WebUI（包含 gateway）
nanobot webui

# 仅启动 gateway（后台模式）
nanobot gateway --background

# 查看 gateway 状态
nanobot gateway status

# 启动 OpenAI 兼容 API 服务器
nanobot serve

# 运行测试
pytest tests/ -v

# WebUI 开发服务器
cd webui && bun run dev
```

## 开发工作流

### 代码质量工具

| 工具 | 命令 | 目的 |
|------|------|------|
| ruff | `ruff check nanobot/` | Python 代码检查 |
| pytest | `pytest tests/ -v` | 单元/集成测试 |
| Vitest | `cd webui && bun run test` | WebUI 测试 |
| TypeScript | `cd webui && bun run build` | 类型检查 + 构建 |

### 代码风格规范

**Python**（pyproject.toml 中定义）:
- ruff 检查：E, F, I, N, W 规则
- 行宽：100 字符（E501 忽略）
- asyncio 异步模式
- pytest `asyncio_mode = "auto"`

**TypeScript/React**:
- ESLint 10 + typescript-eslint
- Tailwind CSS 3 样式
- Vitest + Testing Library 测试

### 提交规范

提交信息遵循 conventional commits 格式：
- `feat:` - 新功能
- `fix:` - Bug 修复
- `chore:` - 杂项
- `refactor:` - 重构
- `docs:` - 文档
- `test:` - 测试

## 常见任务

### 添加新的 LLM 提供商

**需修改的文件**:
1. `nanobot/providers/<name>_provider.py` - 新建提供商实现
2. `nanobot/providers/registry.py` - 注册新提供商
3. `nanobot/config/schema.py` - 添加提供商配置类型
4. `tests/providers/test_<name>.py` - 添加测试

**步骤**:
1. 继承 `nanobot/providers/base.py` 中的 `LLMProvider`（ABC）
2. 实现 `send_message()` 和 `send_message_stream()`
3. 实现 `tool_definitions_to_schema()` 转换工具定义
4. 在 `registry.py` 中注册
5. 在 `schema.py` 中添加配置 schema
6. 编写测试（优先 mock 方式）

### 添加新的聊天渠道

**需修改的文件**:
1. `nanobot/channels/<name>/` - 新建渠道目录
2. `nanobot/channels/<name>/manifest.py` - 渠道清单
3. `nanobot/channels/<name>/runtime.py` - 渠道运行时
4. `nanobot/channels/<name>/validation.py` - 配置验证（可选）
5. `tests/channels/test_<name>.py` - 测试

**步骤**:
1. 创建渠道目录，继承 `BaseChannel`（`nanobot/channels/base.py`）
2. 实现 `start()`, `stop()`, `send()` 方法
3. 编写 `manifest.py` 声明渠道元数据
4. 可选：添加 `webui/` 子目录提供渠道专用 UI
5. ChannelManager 通过 pkgutil 自动发现新渠道

### 添加新的 Agent 工具

**需修改的文件**:
1. `nanobot/agent/tools/<tool_name>.py` - 新建工具文件
2. 注册（pkgutil 自动扫描或 entry-point 插件）

**步骤**:
1. 继承 `nanobot/agent/tools/base.py` 中的 `Tool` 基类
2. 定义工具的 `name`, `description`, `parameters`（JSON Schema）
3. 实现 `execute()` 方法
4. 文件放置在 `nanobot/agent/tools/` 下，由 `ToolLoader` 自动发现

### 添加新的 WebUI 功能

**需修改的文件**:
1. `webui/src/components/<component>.tsx` - 新组件
2. `webui/src/lib/<module>.ts` - 核心逻辑
3. `webui/src/tests/<test>.test.tsx` - 测试
4. `nanobot/webui/<api>.py` - 后端 API（如需要）

**步骤**:
1. 在 `webui/src/components/` 添加 React 组件
2. 如需后端支持，在 `nanobot/webui/` 添加对应 API
3. WebUI 通过 WebSocket 多路复用协议与 gateway 通信
4. 运行 `bun run test` 验证

### 调试

- Gateway 日志：`~/.nanobot/logs/gateway.log`
- 跟踪日志：`nanobot gateway logs`（实时查看）
- 配置验证：`nanobot chat` 的 REPL 模式
- 交互式调试：在 REPL 中使用 `/debug` 命令（如配置了）

## 编码规范

### 文件组织
- 每个文件一个类/职责
- 测试文件与源码同目录结构（`tests/` 镜像 `nanobot/`）
- 渠道是自包含包（manifest + runtime + validation + webui）

### 命名

| 类型 | 约定 | 示例 |
|------|------|------|
| 文件 | snake_case | `agent_loop.py` |
| 类 | PascalCase | `AgentLoop` |
| 函数 | snake_case | `send_message` |
| 常量 | SCREAMING_SNAKE | `DEFAULT_STREAM_IDLE_TIMEOUT_S` |

### 异步编程

- 使用 `asyncio` 和 `async/await`
- 渠道和 Agent 使用 `asyncio.Task` 实现并发
- MessageBus 使用 `asyncio.Queue` 解耦

### 错误处理

```python
# 推荐：使用 loguru 日志，明确的异常类型
from loguru import logger
logger.warning("Failed to send message to {}", channel_name)
raise ChannelError("Connection lost")

# 流式处理中，捕获异常并转换为 StreamErrorEvent
```

### 日志

```python
from loguru import logger

logger.debug()   # 开发细节
logger.info()    # 正常操作
logger.warning() # 可恢复问题
logger.error()   # 需要关注的故障
```

### 测试

- 测试框架：pytest + pytest-asyncio
- 测试文件：`test_<module>.py` 放在 `tests/` 对应子目录
- mock 优先：对于 LLM 提供商和外部 API 调用
- 测试覆盖率目标：>75%（`pyproject.toml` 中 `fail_under = 75`）

## Docker 部署

```bash
# 构建镜像
docker compose build

# 启动 gateway
docker compose up -d nanobot-gateway

# 启动 API 服务器
docker compose up -d nanobot-api

# 启动 CLI（交互式）
docker compose --profile cli up nanobot-cli

# 使用 Bubblewrap 沙箱
docker compose -f docker-compose.yml -f docker-compose.bwrap.yml up -d
```

镜像使用多阶段构建：
1. Stage 1（webui-builder）：Node.js 构建 WebUI 产物
2. Stage 2（runtime）：Python 3.12 + 非 root 用户 `nanobot`（UID 1000）

`entrypoint.sh` 处理 Render 部署环境中的权限切换（root -> nanobot 用户）。

## 项目架构约束

详见 `.agent/design.md` 和 `.agent/security.md`。关键约束包括：

- Agent 必须无状态（通过 SessionManager 加载状态）
- 所有文件 I/O 使用原子写入 + fsync
- 渠道与 Agent 通过 MessageBus 完全解耦
- 工作区访问受 WorkspaceScopeResolver 限制
- 网络请求受 SSRF 白名单保护
