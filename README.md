# Claude Code — Leaked Source (2026-03-31)

> **On March 31, 2026, the full source code of Anthropic's Claude Code CLI was leaked** via a `.map` file exposed in their npm registry.

---

## How It Leaked

[Chaofan Shou (@Fried_rice)](https://x.com/Fried_rice) discovered the leak and posted it publicly:

> **"Claude code source code has been leaked via a map file in their npm registry!"**
>
> — [@Fried_rice, March 31, 2026](https://x.com/Fried_rice/status/2038894956459290963)

The source map file in the published npm package contained a reference to the full, unobfuscated TypeScript source, which was downloadable as a zip archive from Anthropic's R2 storage bucket.

---

## Overview

Claude Code is Anthropic's official CLI tool that lets you interact with Claude directly from the terminal to perform software engineering tasks — editing files, running commands, searching codebases, managing git workflows, and more.

This repository contains the leaked `src/` directory.

- **Leaked on**: 2026-03-31
- **Language**: TypeScript
- **Runtime**: Bun
- **Terminal UI**: React + [Ink](https://github.com/vadimdemedes/ink) (React for CLI)
- **Scale**: ~1,900 files, 512,000+ lines of code

---

## Directory Structure

```
src/
├── main.tsx                 # Entrypoint (Commander.js-based CLI parser)
├── commands.ts              # Command registry
├── tools.ts                 # Tool registry
├── Tool.ts                  # Tool type definitions
├── QueryEngine.ts           # LLM query engine (core Anthropic API caller)
├── context.ts               # System/user context collection
├── cost-tracker.ts          # Token cost tracking
│
├── commands/                # Slash command implementations (~50)
├── tools/                   # Agent tool implementations (~40)
├── components/              # Ink UI components (~140)
├── hooks/                   # React hooks
├── services/                # External service integrations
├── screens/                 # Full-screen UIs (Doctor, REPL, Resume)
├── types/                   # TypeScript type definitions
├── utils/                   # Utility functions
│
├── bridge/                  # IDE integration bridge (VS Code, JetBrains)
├── coordinator/             # Multi-agent coordinator
├── plugins/                 # Plugin system
├── skills/                  # Skill system
├── keybindings/             # Keybinding configuration
├── vim/                     # Vim mode
├── voice/                   # Voice input
├── remote/                  # Remote sessions
├── server/                  # Server mode
├── memdir/                  # Memory directory (persistent memory)
├── tasks/                   # Task management
├── state/                   # State management
├── migrations/              # Config migrations
├── schemas/                 # Config schemas (Zod)
├── entrypoints/             # Initialization logic
├── ink/                     # Ink renderer wrapper
├── buddy/                   # Companion sprite (Easter egg)
├── native-ts/               # Native TypeScript utils
├── outputStyles/            # Output styling
├── query/                   # Query pipeline
└── upstreamproxy/           # Proxy configuration
```

---

## Core Architecture

### 1. Tool System (`src/tools/`)

Every tool Claude Code can invoke is implemented as a self-contained module. Each tool defines its input schema, permission model, and execution logic.

| Tool | Description |
|---|---|
| `BashTool` | Shell command execution |
| `FileReadTool` | File reading (images, PDFs, notebooks) |
| `FileWriteTool` | File creation / overwrite |
| `FileEditTool` | Partial file modification (string replacement) |
| `GlobTool` | File pattern matching search |
| `GrepTool` | ripgrep-based content search |
| `WebFetchTool` | Fetch URL content |
| `WebSearchTool` | Web search |
| `AgentTool` | Sub-agent spawning |
| `SkillTool` | Skill execution |
| `MCPTool` | MCP server tool invocation |
| `LSPTool` | Language Server Protocol integration |
| `NotebookEditTool` | Jupyter notebook editing |
| `TaskCreateTool` / `TaskUpdateTool` | Task creation and management |
| `SendMessageTool` | Inter-agent messaging |
| `TeamCreateTool` / `TeamDeleteTool` | Team agent management |
| `EnterPlanModeTool` / `ExitPlanModeTool` | Plan mode toggle |
| `EnterWorktreeTool` / `ExitWorktreeTool` | Git worktree isolation |
| `ToolSearchTool` | Deferred tool discovery |
| `CronCreateTool` | Scheduled trigger creation |
| `RemoteTriggerTool` | Remote trigger |
| `SleepTool` | Proactive mode wait |
| `SyntheticOutputTool` | Structured output generation |

### 2. Command System (`src/commands/`)

User-facing slash commands invoked with `/` prefix.

| Command | Description |
|---|---|
| `/commit` | Create a git commit |
| `/review` | Code review |
| `/compact` | Context compression |
| `/mcp` | MCP server management |
| `/config` | Settings management |
| `/doctor` | Environment diagnostics |
| `/login` / `/logout` | Authentication |
| `/memory` | Persistent memory management |
| `/skills` | Skill management |
| `/tasks` | Task management |
| `/vim` | Vim mode toggle |
| `/diff` | View changes |
| `/cost` | Check usage cost |
| `/theme` | Change theme |
| `/context` | Context visualization |
| `/pr_comments` | View PR comments |
| `/resume` | Restore previous session |
| `/share` | Share session |
| `/desktop` | Desktop app handoff |
| `/mobile` | Mobile app handoff |

### 3. Service Layer (`src/services/`)

| Service | Description |
|---|---|
| `api/` | Anthropic API client, file API, bootstrap |
| `mcp/` | Model Context Protocol server connection and management |
| `oauth/` | OAuth 2.0 authentication flow |
| `lsp/` | Language Server Protocol manager |
| `analytics/` | GrowthBook-based feature flags and analytics |
| `plugins/` | Plugin loader |
| `compact/` | Conversation context compression |
| `policyLimits/` | Organization policy limits |
| `remoteManagedSettings/` | Remote managed settings |
| `extractMemories/` | Automatic memory extraction |
| `tokenEstimation.ts` | Token count estimation |
| `teamMemorySync/` | Team memory synchronization |

### 4. Bridge System (`src/bridge/`)

A bidirectional communication layer connecting IDE extensions (VS Code, JetBrains) with the Claude Code CLI.

- `bridgeMain.ts` — Bridge main loop
- `bridgeMessaging.ts` — Message protocol
- `bridgePermissionCallbacks.ts` — Permission callbacks
- `replBridge.ts` — REPL session bridge
- `jwtUtils.ts` — JWT-based authentication
- `sessionRunner.ts` — Session execution management

### 5. Permission System (`src/hooks/toolPermission/`)

Checks permissions on every tool invocation. Either prompts the user for approval/denial or automatically resolves based on the configured permission mode (`default`, `plan`, `bypassPermissions`, `auto`, etc.).

### 6. Feature Flags

Dead code elimination via Bun's `bun:bundle` feature flags:

```typescript
import { feature } from 'bun:bundle'

// Inactive code is completely stripped at build time
const voiceCommand = feature('VOICE_MODE')
  ? require('./commands/voice/index.js').default
  : null
```

Notable flags: `PROACTIVE`, `KAIROS`, `BRIDGE_MODE`, `DAEMON`, `VOICE_MODE`, `AGENT_TRIGGERS`, `MONITOR_TOOL`

---

## 基于 `src/` 源码的架构分析

上面的章节偏向仓库概览；如果直接从 `src/` 源码往下看，这个项目真正有意思的地方在于：它并不只是“一个带工具的 CLI”，而是一个分层很清晰的 **agent runtime（代理运行时）**。命令入口、能力注册、权限决策、状态管理、UI 渲染、特性开关都被拆成了相对独立的控制面。

### 1. 主脉络：`main.tsx` → `commands.ts` / `tools.ts` → `query.ts` / `QueryEngine.ts`

- **`src/main.tsx`** 是运行时装配中心。
  - 它不只是启动 CLI parser。
  - 文件最前面会先触发 `startMdmRawRead()`、`startKeychainPrefetch()` 这样的副作用，让启动 I/O 和后续模块加载并行发生。
  - 初始化、配置、认证、遥测、命令、工具、MCP、REPL 渲染、特性开关模块都在这里汇合。
- **`src/commands.ts`** 是用户意图入口的注册中心。
  - 所有 slash command 在这里集中组织，并通过条件导入区分不同运行环境。
- **`src/tools.ts`** 是模型能力注册中心。
  - 它本质上决定了“模型能调用哪些能力”。
  - 它把基础工具汇总起来，再结合 feature flag、运行环境和 lazy `require()` 做裁剪与解耦。
- **`src/query.ts`** 是单轮执行循环。
  - 它负责流式响应、tool use、上下文压缩、token 预算、继续执行与恢复策略。
- **`src/QueryEngine.ts`** 是会话级协调器。
  - 它把 `query()` 包装成一个可复用的多轮会话引擎，管理消息状态、usage、回放、memory 加载，以及 SDK / headless 场景。

换句话说，整个系统把下面几件事刻意拆开了：

1. **进程启动与运行时装配**（`main.tsx`）
2. **用户意图入口**（`commands.ts`）
3. **模型可用能力**（`tools.ts`）
4. **单轮执行协议**（`query.ts`）
5. **多轮会话状态**（`QueryEngine.ts`）

这正是它架构精妙的第一层：**“用户想做什么”、“模型能做什么”、“一次执行如何推进”** 被清楚地区分开了。

### 2. 执行链路：从输入到工具循环再到 UI

从源码能看出的主链路大致是：

```text
user input
  → src/main.tsx
  → command resolution in src/commands.ts
  → tool availability from src/tools.ts
  → turn execution in src/query.ts
  → multi-turn/session management in src/QueryEngine.ts
  → tool orchestration in src/services/tools/toolOrchestration.ts
  → permission checks in src/hooks/useCanUseTool.tsx
  → state/UI updates through src/state/AppState.tsx and Ink components
```

这条链路的精妙之处在于：模型循环并没有和 UI 渲染耦死在一起。

- `query.ts` / `QueryEngine.ts` 关注的是 **消息、状态推进、预算、工具结果**
- Ink / React 组件关注的是 **展示**
- permission hook 关注的是 **human-in-the-loop 的治理**
- tool 模块关注的是 **单个能力的执行逻辑**

因此它可以继续扩展，而不会把所有复杂度都堆到 CLI 入口里。

### 3. 工具体系为什么不只是“一个 registry”

`src/tools.ts` 乍看只是很长的 import 列表，但它其实暴露了几个很关键的设计：

- 它是**统一的能力目录**。
- 它通过 **feature flag + 运行时条件** 缩小实际暴露的工具面。
- 它通过 **lazy import** 避免循环依赖。
- 它把“全部可能工具”和“当前可用工具”这两个概念分开了。

这点很重要。很多 agent 系统里，工具发现、工具开关、工具权限会慢慢散落到各处；而这里是有意识地收拢在同一层里，因此更容易回答：

- 模型理论上能看到什么
- 当前运行时实际上能执行什么
- 当前环境应当暴露什么

这对于一个大型 CLI agent 来说，是非常实用而成熟的架构取舍。

### 4. `query.ts` 的本质是“状态机”，而不只是 API 包装

`src/query.ts` 是整个仓库里最值得看的文件之一。

它不是“请求模型然后把答案打印出来”这么简单，而是在管理：

- streaming event
- tool use / tool result 的拼接与兜底恢复
- token budget 与 continuation
- 上下文变长后的 compact
- stop hook / post-sampling hook
- 多次迭代之间的 transition 原因

它的高级之处在于：它把一次 LLM turn 当成了一个 **带状态推进规则的协议**，而不是单次请求。

正因为这样，这个 CLI 才能稳定支持：

- 长链路工具调用
- 自动继续执行
- 上下文压缩与恢复
- 结构化中断处理

这也是“聊天前端代码”和“代理运行时代码”的真正区别。

### 5. 工具执行层最漂亮的一点：并发安全分批

`src/services/tools/toolOrchestration.ts` 里有一个非常漂亮的设计：先判断工具调用是否可以并发，再按批次执行。

- 只读 / 并发安全工具并行跑
- 会修改状态的工具串行跑
- context modifier 会被排队并按确定顺序回放

这背后平衡了三件事：

- **速度**：能并行的尽量并行
- **正确性**：涉及状态变化时保持串行
- **可预测性**：上下文变更顺序明确

很多系统不是“一律串行，浪费时间”，就是“过度并发，最后打架”。这里走的是中间但成熟的路线。

### 6. 权限系统是第一层控制面，而不是工具里的附属逻辑

`src/hooks/useCanUseTool.tsx` 以及周边 permission 模块说明了另一个很强的点：权限检查不是散落在各个工具内部。

它被提炼成了独立的一层，可以统一处理：

- 配置或策略直接放行
- 配置或策略直接拒绝
- 进入交互式确认
- 接入 classifier / auto mode 这类自动化判定
- 接入 swarm / coordinator 这样的特殊模式

这样一来，工具作者不需要重复发明一套 permission 逻辑，而 runtime 提供统一决策管线。这个抽象层次非常对。

### 7. Feature flag 不是点缀，而是真正在塑造架构

源码里大量使用了 Bun 的 feature flag，例如：

- `src/main.tsx`
- `src/commands.ts`
- `src/tools.ts`
- `src/query.ts`
- `src/state/AppState.tsx`

微妙的地方在于，它们不只是普通的 `if` 开关，而经常配合：

- 条件 `require()`
- 死代码消除（dead-code elimination）
- 同一套源码生成不同产品形态

这意味着：一个源码树可以服务多个分发形态，同时又不必让所有环境都承担完整的运行时和导入成本。这是相当“工程化”的设计。

### 8. 状态管理的精妙之处：既用 React，又不让核心逻辑 React 化

`src/state/AppState.tsx` 不只是一个普通 provider。它通过 `useSyncExternalStore` 暴露中心 store，因此：

- 有统一的 app store
- 组件按 slice 订阅
- 非 React 代码也能拿到 `getState` / `setState`
- UI 渲染和核心运行时逻辑保持解耦

这是一种非常好的折中：

- React / Ink 被用在最擅长的地方：终端 UI
- agent runtime 本身没有被迫“React 化”

这一点对同时支持 REPL、headless、bridge、SDK 等多种执行路径的项目尤其重要。

### 9. 这套架构最厉害的地方：在大体量代码里还守住了边界

真正让人觉得“精妙”的，不是某一个孤立模块，而是整个项目反复坚持边界分离：

- command registration ≠ tool registration
- tool registration ≠ permission resolution
- permission resolution ≠ tool execution
- tool execution ≠ query orchestration
- query orchestration ≠ rendering

对于一个大型 TypeScript CLI 项目来说，这种边界感是很难得的。它使得仓库可以继续生长出：

- bridge / IDE 模式
- swarm / coordinator 模式
- MCP 集成
- plugin / skill 系统
- 多种 permission model

而不是最后退化成一个什么都往里塞的巨型 main loop。

### 10. 用一句话概括

如果直接从源码来理解 Claude Code，它更像是：

> **一个分层的代理运行时：commands 表达用户意图，tools 表达模型能力，query loop 执行一个有状态的 model/tool 协议，而 permissions、state、UI 则作为独立控制面包裹在外层。**

这就是它架构的精确性，也是它能持续扩展的根本原因。

---

## Key Files in Detail

### `QueryEngine.ts` (~46K lines)

The core engine for LLM API calls. Handles streaming responses, tool-call loops, thinking mode, retry logic, and token counting.

### `Tool.ts` (~29K lines)

Defines base types and interfaces for all tools — input schemas, permission models, and progress state types.

### `commands.ts` (~25K lines)

Manages registration and execution of all slash commands. Uses conditional imports to load different command sets per environment.

### `main.tsx`

Commander.js-based CLI parser + React/Ink renderer initialization. At startup, parallelizes MDM settings, keychain prefetch, and GrowthBook initialization for faster boot.

---

## Tech Stack

| Category | Technology |
|---|---|
| Runtime | [Bun](https://bun.sh) |
| Language | TypeScript (strict) |
| Terminal UI | [React](https://react.dev) + [Ink](https://github.com/vadimdemedes/ink) |
| CLI Parsing | [Commander.js](https://github.com/tj/commander.js) (extra-typings) |
| Schema Validation | [Zod v4](https://zod.dev) |
| Code Search | [ripgrep](https://github.com/BurntSushi/ripgrep) (via GrepTool) |
| Protocols | [MCP SDK](https://modelcontextprotocol.io), LSP |
| API | [Anthropic SDK](https://docs.anthropic.com) |
| Telemetry | OpenTelemetry + gRPC |
| Feature Flags | GrowthBook |
| Auth | OAuth 2.0, JWT, macOS Keychain |

---

## Notable Design Patterns

### Parallel Prefetch

Startup time is optimized by prefetching MDM settings, keychain reads, and API preconnect in parallel — before heavy module evaluation begins.

```typescript
// main.tsx — fired as side-effects before other imports
startMdmRawRead()
startKeychainPrefetch()
```

### Lazy Loading

Heavy modules (OpenTelemetry ~400KB, gRPC ~700KB) are deferred via dynamic `import()` until actually needed.

### Agent Swarms

Sub-agents are spawned via `AgentTool`, with `coordinator/` handling multi-agent orchestration. `TeamCreateTool` enables team-level parallel work.

### Skill System

Reusable workflows defined in `skills/` and executed through `SkillTool`. Users can add custom skills.

### Plugin Architecture

Built-in and third-party plugins are loaded through the `plugins/` subsystem.

---

## Disclaimer

This repository archives source code that was leaked from Anthropic's npm registry on **2026-03-31**. All original source code is the property of [Anthropic](https://www.anthropic.com). Contact [nichxbt](https://www.x.com/nichxbt) for any comments.
