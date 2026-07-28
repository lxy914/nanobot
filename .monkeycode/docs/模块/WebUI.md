# WebUI 模块

基于 React/TypeScript 的浏览器聊天界面，通过 WebSocket 与 Gateway 通信，提供 Markdown 渲染、文件上传、多语言等完整聊天体验。

## 结构

```
webui/
├── src/
│   ├── main.tsx                    # 应用入口：ReactDOM 渲染、polyfill、运行时初始化
│   ├── App.tsx                     # 主应用组件：会话管理、渠道切换、主题/设置（2177行）
│   ├── globals.css                 # 全局样式
│   ├── components/
│   │   ├── Sidebar.tsx             # 侧边栏
│   │   ├── ChatList.tsx            # 会话列表
│   │   ├── MessageBubble.tsx       # 消息气泡
│   │   ├── MarkdownText.tsx        # Markdown 文本渲染
│   │   ├── MarkdownTextRenderer.tsx# 流式 Markdown 渲染器
│   │   ├── CodeBlock.tsx           # 代码块语法高亮
│   │   ├── ChatInput.tsx           # 输入框
│   │   ├── FilePreview.tsx         # 文件预览
│   │   ├── thread/                 # 线程/消息面板
│   │   ├── settings/               # 设置视图组件
│   │   └── ui/                     # 基础 UI 组件库（基于 Radix UI）
│   ├── hooks/
│   │   ├── useNanobotStream.ts     # WebSocket 流式消息处理
│   │   ├── useSessions.ts          # 会话列表管理
│   │   ├── useSettings.ts          # 设置管理
│   │   ├── useTheme.ts             # 明暗主题切换
│   │   ├── useVoiceRecorder.ts     # 语音录制
│   │   ├── useSkills.ts            # 技能数据获取
│   │   └── useSidebarState.ts      # 侧边栏状态持久化
│   ├── lib/
│   │   ├── nanobot-client.ts       # WebSocket 客户端核心实现
│   │   ├── api.ts                  # REST API 调用
│   │   ├── bootstrap.ts            # 认证引导（secret/token）
│   │   ├── types.ts                # TypeScript 类型定义
│   │   ├── runtime.ts              # Loopback 运行时初始化
│   │   ├── tool-traces.ts          # 工具调用追踪
│   │   ├── workspace.ts            # 工作区文件浏览
│   │   └── provider-brand.ts       # 提供商品牌信息
│   ├── providers/
│   │   └── NanobotClient.tsx        # React Context：NanobotClientProvider
│   ├── channel-plugins/            # 渠道插件系统
│   │   ├── registry.ts             # 渠道 UI 注册
│   │   └── types.ts                # 渠道插件类型
│   ├── i18n/                       # 国际化
│   │   ├── config.ts               # i18next 配置
│   │   └── locales/                # 语言包
│   ├── workers/
│   │   └── imageEncode.ts          # 图片编码 Web Worker
│   └── tests/                      # 52 个测试文件
├── package.json                    # 依赖和脚本
├── vite.config.ts                  # Vite 构建配置
├── tsconfig.json                   # TypeScript 配置
└── tailwind.config.js              # Tailwind CSS 配置
```

## 关键文件

| 文件 | 目的 |
|------|------|
| `App.tsx` | 主应用：路由、会话选择、WebSocket 连接管理、Bootstrap 认证 |
| `lib/nanobot-client.ts` | WebSocket 客户端：多路复用协议、流式事件分发、重连逻辑 |
| `hooks/useNanobotStream.ts` | 消费 WebSocket 客户端流式事件，更新 React 状态 |
| `components/MessageBubble.tsx` | 渲染单条消息：用户/AI 气泡、Markdown 内容、工具调用状态 |
| `lib/bootstrap.ts` | 首次连接时的 Bootstrap 认证流程 |
| `vite.config.ts` | 构建输出到 `../nanobot/web/dist/`，开发代理到 gateway `:8765` |

## 依赖

**本模块依赖**:
- `react` / `react-dom` 18 - UI 框架
- `radix-ui` - 对话框、下拉菜单、提示框等无障碍组件
- `streamdown` - 流式 Markdown 渲染
- `react-syntax-highlighter` - 代码语法高亮
- `i18next` / `react-i18next` - 国际化
- `tailwindcss` - 样式框架
- `lucide-react` - 图标库

**依赖本模块的**:
- Gateway 将构建产物 `nanobot/web/dist/` 作为 WebUI 静态资源提供服务

## 构建配置

Vite 配置（`vite.config.ts`）：
- 开发服务器：`127.0.0.1:5173`（strictPort）
- 构建输出：`../nanobot/web/dist/`
- 开发代理：`/webui`, `/api`, `/auth` -> `http://127.0.0.1:8765`
- 代码分割：`syntax-highlight`, `markdown-vendor`, `katex`
- HMR 专用路径：`/__nanobot_vite_hmr`（避免与业务 WebSocket 冲突）
