# Channels 模块

聊天渠道集成层，将外部消息平台的用户消息转换为系统内部格式，管理 17 种聊天平台的生命周期。

## 结构

```
channels/
├── base.py              # BaseChannel ABC：渠道基类（298行）
├── manager.py           # ChannelManager：生命周期管理
├── registry.py          # 渠道注册表
├── connect.py           # 渠道连接器
├── contracts.py         # 渠道契约/接口定义
├── plugin.py            # 渠道插件接口
├── validation.py        # 配置验证
├── _manifest.py          # 清单辅助
├── _setup.py             # 安装辅助
├── telegram/            # Telegram Bot
├── discord/             # Discord Bot
├── slack/               # Slack App
├── weixin/              # 微信
├── feishu/              # 飞书
├── dingtalk/            # 钉钉
├── whatsapp/            # WhatsApp
├── qq/                  # QQ
├── wecom/               # 企业微信
├── matrix/              # Matrix 协议
├── signal/              # Signal
├── email/               # 电子邮件
├── msteams/             # Microsoft Teams
├── mattermost/          # Mattermost
├── mochat/              # MoChat
├── napcat/              # NapCat（QQ 兼容）
└── websocket/           # WebSocket 直连
```

## 关键文件

| 文件 | 目的 |
|------|------|
| `base.py` | BaseChannel ABC：定义所有渠道的公共接口和默认行为（配对审批、音频转录、重试） |
| `manager.py` | ChannelManager：通过 pkgutil 自动发现渠道，管理 start/stop 生命周期 |
| `registry.py` | 渠道注册表：维护所有可用渠道的元数据 |

## 渠道目录结构

每个渠道是自包含包：

```
channels/<name>/
├── __init__.py
├── manifest.py         # MANIFEST 字典：name, display_name, description
├── runtime.py          # BaseChannel 实现：start(), stop(), send()
├── validation.py       # 配置验证（可选）
├── connect.py          # 连接器（部分渠道如 feishu, weixin）
├── webui/              # 渠道专用 WebUI 设置组件
└── tests/              # 渠道单元测试
```

## 依赖

**本模块依赖**:
- `nanobot/bus/` - MessageBus 消息队列
- `nanobot/config/` - 配置 schema 中的 ChannelsConfig
- `nanobot/pairing/` - 配对审批存储
- 各平台 SDK（python-telegram-bot, discord.py, slack-sdk 等）

**依赖本模块的**:
- `nanobot/cli/commands.py` - Gateway 通过 ChannelManager 管理渠道

## 添加新渠道

1. 创建 `nanobot/channels/<name>/` 目录
2. 编写 `manifest.py` 声明渠道元数据
3. 编写 `runtime.py` 继承 `BaseChannel` 并实现 `start()`, `stop()`, `send()`
4. 可选：添加 `validation.py`, `webui/` 子目录
5. ChannelManager 通过 pkgutil 自动发现
