# Tools 模块

Agent 工具集为 LLM 提供与系统交互的能力，包括文件系统操作、Shell 执行、Web 搜索等 20+ 种工具。

## 结构

```
agent/tools/
├── base.py              # Tool/ToolResult：工具基类
├── registry.py          # ToolRegistry：工具注册/执行/缓存（211行）
├── loader.py            # ToolLoader：pkgutil 自动发现 + entry-point 插件
├── context.py           # RequestContext：每个请求的上下文绑定
├── schema.py            # 工具模式定义
├── filesystem.py        # 文件系统操作（读/写/编辑/列表）
├── shell.py             # Shell 命令执行（含沙箱后端）
├── search.py            # Web 搜索（DuckDuckGo）
├── web.py               # Web 页面抓取
├── mcp.py               # MCP 服务器工具连接器
├── cron.py              # 定时任务管理
├── long_task.py         # 长时间运行任务/持续性目标
├── image_generation.py  # AI 图像生成
├── self.py              # MyTool：自我修改/配置
├── spawn.py             # 子 Agent 生成
├── message.py           # MessageTool：发送消息给用户
├── apply_patch.py       # 补丁应用
├── sandbox.py           # 沙箱后端接口
├── exec_session.py      # ExecSessionManager：Shell 会话管理
├── file_state.py        # FileStateStore：文件状态跟踪
├── runtime_state.py     # 运行时状态工具
├── cli_apps.py          # CLI 应用工具配置
└── path_utils.py        # 路径辅助函数
```

## 关键文件

| 文件 | 目的 |
|------|------|
| `registry.py` | ToolRegistry：字典式工具注册表，区分 built-in/MCP 工具，缓存模式定义 |
| `loader.py` | ToolLoader：通过 pkgutil 扫描 + entry_point 插件自动发现工具 |
| `filesystem.py` | 提供 read/write/edit/list 文件操作，受工作区安全策略限制 |
| `shell.py` | 执行 Shell 命令，支持沙箱隔离（bubblewrap） |
| `search.py` | DuckDuckGo 即时搜索集成 |
| `mcp.py` | 连接和调用 MCP 服务器的工具 |
| `long_task.py` | 管理需要多轮执行的长期任务和持续性目标 |

## 依赖

**本模块依赖**:
- `nanobot/security/` - 工作区访问控制和网络安全
- `nanobot/session/` - 执行会话状态
- `ddgs` - DuckDuckGo 搜索
- `mcp` - Model Context Protocol SDK

**依赖本模块的**:
- `nanobot/agent/runner.py` - AgentRunner 调用 ToolRegistry 执行工具
- `nanobot/agent/loop.py` - AgentLoop 初始化工具注册表

## 添加新工具

1. 创建文件 `nanobot/agent/tools/<name>.py`
2. 继承 `Tool` 基类，定义 `name`, `description`, `parameters`（JSON Schema）
3. 实现 `execute()` 方法
4. 文件放置在 `nanobot/agent/tools/` 下，由 ToolLoader 自动发现

或通过 entry-point 插件注册：

```toml
# pyproject.toml
[project.entry-points."nanobot.tools"]
my_plugin = "my_package.plugins:MyTool"
```
