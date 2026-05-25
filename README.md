# Hermes Agent Dashboard — UI 重设计参考文档

## 1. 整体布局结构

Dashboard 是一个**单页应用 (SPA)**，布局为：

```
┌─────────────────────────────────────────────┐
│  顶栏 (Header)                               │
│  [汉堡菜单] [页面标题]         [语言][主题][?] │
├──────────┬──────────────────────────────────┤
│ 侧边栏    │  主内容区                         │
│ (固定宽)  │  (可滚动)                         │
│          │                                   │
│ 导航项1  │                                   │
│ 导航项2  │                                   │
│ ...      │                                   │
│          │                                   │
│──────────│                                   │
│ 底部状态  │                                   │
└──────────┴──────────────────────────────────┘
```

- **侧边栏宽度**: 约 240px，固定不动
- **主内容区**: 自适应宽度，内容可滚动
- **响应式**: 手机端侧边栏变为抽屉式弹出

---

## 2. 导航菜单项（侧边栏）

| 路径 | 中文名 | 英文名 | 图标 (Lucide) | 功能 |
|------|--------|--------|---------------|------|
| `/chat` | 聊天 | Chat | Terminal | 嵌入式终端聊天（xterm.js + WebSocket） |
| `/sessions` | 会话 | Sessions | MessageSquare | 浏览/搜索/恢复历史会话 |
| `/analytics` | 分析 | Analytics | BarChart3 | Token 用量统计、趋势图表 |
| `/models` | 模型 | Models | Cpu | 模型选择、Provider 切换、API Key 管理 |
| `/logs` | 日志 | Logs | FileText | 实时日志查看（agent.log, errors.log） |
| `/cron` | 定时任务 | Cron | Clock | Cron 任务管理（创建/编辑/暂停/删除） |
| `/skills` | 技能 | Skills | Package | 技能浏览/安装/管理 |
| `/plugins` | 插件 | Plugins | Puzzle | 插件管理 |
| `/profiles` | 配置档 | Profiles | Users | 多配置档管理 |
| `/config` | 设置 | Config | Settings | 全局配置编辑（YAML） |
| `/env` | 密钥 | Keys | KeyRound | API Key / 环境变量管理 |
| `/docs` | 文档 | Documentation | BookOpen | 内嵌 Hermes 文档站 |

### 底部区域
- **活跃会话数**：显示当前活跃的会话数量
- **网关状态**：显示各平台连接状态（QQ、Telegram 等）
- **侧边栏底部**：`SidebarFooter` + `SidebarStatusStrip`

---

## 3. 各页面详细结构

### 3.1 Chat 聊天页 (`/chat`)
**核心功能**: 嵌入式终端聊天

- **主体**: xterm.js 终端模拟器（WebGL 渲染）
  - 字体: JetBrains Mono
  - 配色: 深青色背景 `#0d2626` + 奶油色文字 `#f0e6d2`
  - 支持 Unicode 11 宽度、Web 链接点击
- **通信**: WebSocket `/api/pty?token=<session>&channel=<id>`
- **右侧边栏** (`ChatSidebar`):
  - 模型信息卡片（当前模型名、Provider）
  - 工具调用列表（实时显示工具调用状态）
  - Slash 命令面板（`/` 触发）
- **工具栏按钮**:
  - 复制按钮 (Copy)
  - 侧边栏开关 (PanelRight)
  - 关闭 (X)

### 3.2 Sessions 会话页 (`/sessions`)
- **会话列表**: 表格/卡片形式展示历史会话
  - 标题、时间、消息数、来源平台
- **搜索**: FTS5 全文搜索
- **操作**: 点击恢复会话、删除会话
- **批量操作**: 批量删除、导出

### 3.3 Analytics 分析页 (`/analytics`)
- **概览卡片**: 总 Token 数、总花费、总会话数
- **趋势图表**: 日/周/月 Token 使用趋势（Chart.js）
- **模型分布**: 各模型使用占比饼图
- **详细列表**: 按日期/模型的使用明细

### 3.4 Models 模型页 (`/models`)
- **当前模型**: 显示当前使用的模型和 Provider
- **Provider 列表**: 所有配置的 Provider
  - 每个 Provider 显示: 名称、Base URL、模型列表、状态
- **模型选择器** (`ModelPickerDialog`):
  - 按 Provider 分组的模型列表
  - 搜索过滤
  - 一键切换
- **OAuth 登录**: 支持 Nous Portal、OpenAI Codex 等 OAuth 认证
  - `OAuthLoginModal` 弹窗
  - `OAuthProvidersCard` 展示
- **API Key 管理**: 添加/删除/轮换 API Key

### 3.5 Logs 日志页 (`/logs`)
- **日志源选择**: agent.log / errors.log / gateway.log
- **实时流**: 自动滚动更新
- **级别过滤**: INFO / WARNING / ERROR / DEBUG
- **行着色**: error=红色, warning=黄色, info=默认, debug=灰色
- **搜索**: 关键词过滤

### 3.6 Cron 定时任务页 (`/cron`)
- **任务列表**: 所有 Cron 任务
  - 名称、调度表达式、上次运行、状态
- **创建任务**: 表单（调度、Prompt、投递目标）
- **操作**: 编辑、暂停/恢复、立即运行、删除
- **状态指示**: 运行中/暂停/错误

### 3.7 Skills 技能页 (`/skills`)
- **已安装技能**: 列表展示
  - 名称、描述、版本、来源
- **技能市场**: 浏览/搜索可安装技能
- **操作**: 安装、卸载、更新、查看详情
- **技能详情**: Markdown 渲染的 SKILL.md 内容

### 3.8 Plugins 插件页 (`/plugins`)
- **已安装插件**: 列表
- **插件市场**: 浏览社区插件
- **操作**: 安装、卸载、启用/禁用

### 3.9 Profiles 配置档页 (`/profiles`)
- **配置档列表**: 所有 Hermes 配置档
- **操作**: 创建、切换、删除、导出/导入
- **详情**: 各配置档的隔离配置、会话、技能

### 3.10 Config 设置页 (`/config`)
- **YAML 编辑器**: 直接编辑 `config.yaml`
- **分类浏览**: 按 section 分组（model, agent, display 等）
- **实时验证**: 语法检查
- **保存/重置**: 原子写入，自动备份

### 3.11 Keys 密钥页 (`/env`)
- **环境变量列表**: 所有 `.env` 中的 Key
  - 名称、值（遮蔽显示）、状态
- **操作**: 添加、编辑、删除、复制
- **Provider 关联**: 显示每个 Key 对应的 Provider

### 3.12 Docs 文档页 (`/docs`)
- **内嵌 iframe**: 加载 `hermes-agent.nousresearch.com/docs/`
- **外部链接**: 新标签页打开

---

## 4. 通用组件

| 组件 | 功能 |
|------|------|
| `LanguageSwitcher` | 语言切换（en/zh/zh-Hant/ja/ko 等） |
| `ThemeSwitcher` | 主题切换（深色/浅色/跟随系统） |
| `ModelInfoCard` | 当前模型信息展示卡片 |
| `ModelPickerDialog` | 模型选择弹窗 |
| `OAuthLoginModal` | OAuth 认证弹窗 |
| `ChatSidebar` | 聊天页右侧信息栏 |
| `SlashPopover` | Slash 命令面板 |
| `ToolCall` | 工具调用状态展示 |
| `Markdown` | Markdown 渲染组件 |
| `Toast` | 消息提示 |
| `Backdrop` | 背景装饰（暖色光晕 + 噪点纹理） |
| `SidebarFooter` | 侧边栏底部 |
| `SidebarStatusStrip` | 侧边栏状态条 |
| `DeleteConfirmDialog` | 删除确认弹窗 |
| `BottomPickSheet` | 底部选择面板（移动端） |

---

## 5. 主题系统

### 默认配色（Hermes Teal）
```css
--background: #041c1c;        /* 深青色背景 */
--foreground: #ffffff;         /* 白色前景 */
--midground: #ffe6cb;          /* 奶油色强调 */
--warm-glow: rgba(255, 189, 56, 0.35);  /* 暖色光晕 */
```

### 主题切换
- **默认 (Hermes Teal)**: 深青色 + 奶油色
- **深色**: 纯黑/深灰
- **浅色**: 白色/浅灰
- **跟随系统**: 自动检测 OS 设置

### 背景装饰 (`Backdrop`)
- CSS 噪点纹理（可调透明度）
- 暖色径向渐变光晕
- 可通过 `--noise-opacity-mul` 控制噪点强度

---

## 6. 国际化 (i18n)

### 切换中文
语言切换器在顶栏右侧，支持：
- `en` — English（默认）
- `zh` — 简体中文 ✅
- `zh-Hant` — 繁體中文
- `ja` — 日本語
- `ko` — 한국어
- `es`, `fr`, `de`, `it`, `pt`, `ru` 等

**切换方式**: 点击顶栏的语言图标，选择 `zh` 即可。

### 中文翻译文件
路径: `web/src/i18n/zh.ts`
包含所有 UI 文本的中文翻译。

---

## 7. 技术栈

| 技术 | 用途 |
|------|------|
| React 18 | UI 框架 |
| React Router | 路由 |
| Tailwind CSS | 样式 |
| @nous-research/ui | Nous 设计系统组件 |
| Lucide React | 图标库 |
| xterm.js | 终端模拟（聊天页） |
| Chart.js | 图表（分析页） |
| Zustand / Context | 状态管理 |
| Vite | 构建工具 |

---

## 8. WebSocket 端点

| 端点 | 用途 |
|------|------|
| `/api/pty?token=<t>&channel=<c>` | PTY 终端（聊天页用） |
| `/api/ws?token=<t>` | JSON-RPC WebSocket（侧边栏元数据） |
| `/api/pub?token=<t>&channel=<c>` | 事件发布（PTY → 侧边栏） |
| `/api/events?channel=<c>` | 事件订阅（侧边栏接收） |

### 认证
- 每次 Dashboard 启动生成一个 `session_token`
- HTML 中注入: `window.__HERMES_SESSION_TOKEN__="<token>"`
- 所有 WebSocket 连接必须带上 `?token=<session_token>`

---

## 9. 设计要求

主人想要的：
1. **功能全保留** — 所有上述页面和功能都要有
2. **界面干净** — 去掉噪点纹理、减少视觉噪音
3. **风格参考** — 类似 WebUI 的简洁风格
4. **中文界面** — 默认使用中文
5. **跨平台** — 手机/平板/桌面都能用
6. **深色主题** — 以深色为主，保护眼睛

### 建议改进
- 去掉 `Backdrop` 的噪点纹理
- 简化侧边栏，用更清晰的分组
- 卡片式布局替代表格
- 更大的触摸目标（移动端友好）
- 统一的圆角和间距
- 柔和的渐变替代硬边框
