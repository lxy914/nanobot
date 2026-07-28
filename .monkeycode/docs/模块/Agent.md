# Agent 核心模块

Agent 核心引擎是 nanobot 的核心，负责消息处理、LLM 对话执行、工具调度和记忆管理。

## 结构

```
agent/
├── loop.py              # AgentLoop：核心消息处理循环（2040行）
├── runner.py            # AgentRunner：多轮 LLM 对话执行器（1505行）
├── memory.py            # MemoryStore + Consolidator：长期记忆（1210行）
├── context.py           # ContextBuilder：LLM 上下文构建
├── context_governance.py# ContextGovernor：上下文治理策略
├── autocompact.py       # AutoCompact：自动上下文压缩
├── subagent.py          # SubagentManager：子 Agent 生成
├── skills.py            # 技能加载器
├── hook.py              # AgentHook/AgentTurnHookFactory：钩子接口
├── turn_hooks.py        # AgentTurnHookSpec：回合级钩子
├── turn_delivery.py     # TurnDelivery/TurnRoute：回合投递
├── cron_turns.py        # CronTurnCoordinator：定时回合协调
├── automation_turns.py  # 自动化回合调度
├── goal_permission.py   # 持续性目标权限
├── model_presets.py     # 模型预设
├── model_runtime.py     # ModelRuntimeResolver：运行时解析
├── progress_hook.py     # 进度钩子
├── hooks/               # 具体钩子实现
└── tools/               # Agent 工具集（独立子模块）
```

## 关键文件

| 文件 | 目的 |
|------|------|
| `loop.py` | AgentLoop：消费消息、调度回合、管理生命周期 |
| `runner.py` | AgentRunner：执行 LLM 对话、工具调用循环、流式输出 |
| `memory.py` | MemoryStore：文件 I/O 记忆持久化；Consolidator：Dream 记忆整合 |
| `context.py` | ContextBuilder：组装系统提示、工具定义、记忆和历史消息 |
| `subagent.py` | SubagentManager：创建和管理子 Agent 会话 |
| `hook.py` | 定义 AgentHook 抽象和 AgentTurnHookFactory |

## 依赖

**本模块依赖**:
- `nanobot/providers/` - LLM 提供商
- `nanobot/session/` - 会话管理
- `nanobot/bus/` - 消息总线
- `nanobot/config/` - 配置 schema
- `nanobot/command/` - 命令路由
- `nanobot/templates/` - 提示模板

**依赖本模块的**:
- `nanobot/cli/` - CLI 使用 AgentLoop 执行对话
- `nanobot/nanobot.py` - Nanobot facade
- `nanobot/api/` - HTTP API 使用 AgentRunner

## 规范

### 钩子模式

Agent 支持两个级别的钩子：Agent 级别（AgentHook）和 Turn 级别（AgentTurnHook）。钩子通过工厂模式创建：

```python
# Agent 级别钩子
class MyAgentHook(AgentHook):
    async def on_message_start(self, ctx): ...
    async def on_tool_call(self, ctx, call): ...

# Turn 级别钩子
class MyTurnHook(AgentTurnHookSpec):
    async def on_turn_start(self, session): ...
    async def on_turn_complete(self, session, result): ...
```

### 错误处理

- AgentRunner 遇到工具执行错误时，将错误信息返回给 LLM 使其自行修正
- 流式超时通过 `NANOBOT_STREAM_IDLE_TIMEOUT_S`（默认 90s）控制
- 上下文预算耗尽时触发 AutoCompact 或抛出 BudgetExhaustedError

### 测试

测试文件位于 `tests/agent/`，覆盖 AgentLoop、AgentRunner、Memory、Subagent、Context 等核心逻辑。
