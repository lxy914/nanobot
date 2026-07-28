# Providers 模块

LLM 提供商层为 nanobot 提供统一的大模型调用接口，支持 7 种提供商，具有自动回退能力。

## 结构

```
providers/
├── base.py                        # LLMProvider ABC：核心抽象（994行）
├── anthropic_provider.py          # Anthropic Claude API
├── openai_compat_provider.py      # OpenAI 兼容 API（通用）
├── azure_openai_provider.py       # Azure OpenAI
├── bedrock_provider.py            # AWS Bedrock
├── github_copilot_provider.py     # GitHub Copilot API
├── openai_codex_provider.py       # OpenAI Codex CLI
├── xai_grok_provider.py           # xAI Grok
├── xai_oauth.py                   # xAI OAuth 认证
├── fallback_provider.py           # 自动回退提供商
├── unconfigured_provider.py       # 未配置占位符
├── factory.py                     # ProviderSnapshot：提供商工厂
├── registry.py                    # 提供商注册表和模型发现
├── image_generation.py            # 图像生成配置
├── transcription.py               # 音频转录（Whisper/Groq）
└── openai_responses/              # OpenAI Responses API 适配器
```

## 关键文件

| 文件 | 目的 |
|------|------|
| `base.py` | LLMProvider ABC、ToolCallRequest、LLMResponse、GenerationSettings 等核心数据类型 |
| `factory.py` | ProviderSnapshot：根据配置快照实例化提供商 |
| `registry.py` | 提供商注册表和模型列表发现 |
| `fallback_provider.py` | 当主提供商失败时自动切换到备用提供商 |
| `image_generation.py` | 图像生成工具的后端提供商配置 |
| `transcription.py` | 音频转录的提供商集成 |

## 核心抽象

```python
class LLMProvider(ABC):
    """LLM 提供商基类"""

    async def send_message(
        self,
        messages: list[dict],
        settings: GenerationSettings,
    ) -> LLMResponse:
        """发送消息并获取完整响应（非流式）"""
        ...

    async def send_message_stream(
        self,
        messages: list[dict],
        settings: GenerationSettings,
    ) -> AsyncIterator[str | ToolCallRequest]:
        """流式发送消息，yield 文本块或工具调用请求"""
        ...

    def tool_definitions_to_schema(self, tools: list[dict]) -> Any:
        """将工具定义转换为提供商特定格式"""
        ...
```

## 依赖

**本模块依赖**:
- `anthropic` - Anthropic Python SDK
- `openai` - OpenAI Python SDK
- `boto3` - AWS SDK（Bedrock）
- `azure-identity` - Azure 认证（Azure OpenAI）

**依赖本模块的**:
- `nanobot/agent/runner.py` - AgentRunner 使用提供商执行 LLM 调用
- `nanobot/cli/commands.py` - CLI 使用提供商快照
- `nanobot/nanobot.py` - Nanobot facade

## 添加新提供商

1. 继承 `LLMProvider`（ABC）
2. 实现 `send_message()` 和 `send_message_stream()`
3. 实现 `tool_definitions_to_schema()` 转换工具格式
4. 在 `registry.py` 中注册
5. 在 `config/schema.py` 中添加 ProviderConfig 子类型
6. 在 `tests/providers/` 中添加测试
