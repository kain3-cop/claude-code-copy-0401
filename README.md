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

## Source-Based Architecture Analysis (`src/`)

The sections above summarize the repository at a high level. Looking directly at the `src/` tree, the architecture is more interesting than “a CLI with tools”: it is a layered **agent runtime** with clear separation between command entry points, capability registration, permissioning, state, UI, and feature-gated product shapes.

### 1. The main spine: `main.tsx` → `commands.ts` / `tools.ts` → `query.ts` / `QueryEngine.ts`

- **`src/main.tsx`** is the runtime composition root.
  - It does more than start a CLI parser.
  - At the very top, it fires side effects such as `startMdmRawRead()` and `startKeychainPrefetch()` so startup I/O overlaps with the rest of module evaluation.
  - Initialization, config, auth, telemetry, commands, tools, MCP, REPL rendering, and feature-gated subsystems all meet here.
- **`src/commands.ts`** is the user-intent registry.
  - It centralizes slash commands and uses conditional imports to shape the command surface per environment.
- **`src/tools.ts`** is the capability registry.
  - It effectively defines what the model can do.
  - It gathers the base tools, then filters them through feature flags, runtime checks, and lazy `require()` helpers to avoid unnecessary coupling.
- **`src/query.ts`** is the single-turn execution loop.
  - It handles streaming responses, tool use, compaction, token budgeting, continuation, and recovery logic.
- **`src/QueryEngine.ts`** is the conversation-level coordinator.
  - It wraps `query()` into a reusable multi-turn engine that manages message state, usage, replay, memory loading, and SDK/headless integration.

In other words, the codebase deliberately separates:

1. **process boot and runtime assembly** (`main.tsx`)
2. **user intent entry points** (`commands.ts`)
3. **model capabilities** (`tools.ts`)
4. **single-turn execution protocol** (`query.ts`)
5. **multi-turn conversation state** (`QueryEngine.ts`)

That is one of the most elegant aspects of the design: “what the user asked”, “what the model can do”, and “how a turn executes” remain distinct.

### 2. Execution flow: from input to tool loop to UI

The main flow visible in the source is roughly:

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

The key architectural win here is that the model loop is not entangled with rendering:

- `query.ts` and `QueryEngine.ts` focus on **messages, transitions, budgets, and tool results**
- Ink/React components focus on **presentation**
- permission hooks focus on **human-in-the-loop governance**
- tool modules focus on **capability-specific logic**

That makes the system extensible without forcing every new feature into the CLI entrypoint.

### 3. Why the tool layer is more than a registry

At first glance, `src/tools.ts` looks like a long import list. In practice, it reveals several deliberate choices:

- it is the **single capability catalog**
- it uses **feature flags and runtime checks** to shrink the active tool surface
- it uses **lazy imports** to avoid circular dependencies
- it separates “all possible tools” from “currently enabled tools”

That matters. In many agent systems, tool discovery, enablement, and exposure drift apart. Here they are intentionally centralized, which makes it easier to reason about:

- what the model can theoretically see
- what the runtime can actually execute
- what the current environment should expose

That is a practical and mature architectural choice for a large CLI agent.

### 4. `query.ts` is fundamentally a state machine, not just an API wrapper

`src/query.ts` is one of the most revealing files in the repository.

It does not simply call the model and print a response. It manages:

- streaming events
- tool-use / tool-result stitching and recovery
- token budgets and continuation
- compaction when context grows
- stop hooks and post-sampling hooks
- transition reasons between iterations

Its architectural strength is that it treats an LLM turn as a **stateful protocol**, not a single request.

That is what allows the CLI to support:

- long tool-call chains
- automatic continuation
- compaction and recovery paths
- structured interruption handling

This is the difference between “chat frontend code” and “agent runtime code”.

### 5. Tool execution is optimized around concurrency safety

`src/services/tools/toolOrchestration.ts` contains one of the cleanest ideas in the codebase: tool calls are partitioned into batches based on whether they are safe to run concurrently.

- read-only / concurrency-safe tools run in parallel
- stateful or uncertain tools run serially
- context modifiers are queued and replayed deterministically

That balances three competing concerns:

- **speed** from parallel execution
- **correctness** from serialized mutation
- **predictability** from explicit context-update ordering

Many systems either run everything serially and waste time, or run too much in parallel and create race conditions. This implementation takes the more mature middle path.

### 6. Permissioning is a first-class control plane

`src/hooks/useCanUseTool.tsx` and the surrounding permission modules show another strong design choice: permission checks are not buried inside individual tools.

Instead, permission handling is a dedicated layer that can:

- allow immediately from config or policy
- deny immediately
- ask the user interactively
- integrate classifier / auto-mode decisions
- incorporate special handling for swarm/coordinator modes

That means tool authors do not need to reinvent permission behavior. The runtime provides a shared decision pipeline, which keeps policy enforcement consistent and extensible.

### 7. Feature flags actively shape the architecture

The codebase makes heavy use of Bun feature flags in places such as:

- `src/main.tsx`
- `src/commands.ts`
- `src/tools.ts`
- `src/query.ts`
- `src/state/AppState.tsx`

The subtlety is that these are not just ordinary `if` toggles. They are often used together with:

- conditional `require()` calls
- dead-code elimination
- environment-specific product shaping from one source tree

That allows the same codebase to serve multiple distributions without forcing every environment to pay the full runtime and import cost.

### 8. State management uses React without making the core runtime React-centric

`src/state/AppState.tsx` is not just a basic provider. It exposes a central store through `useSyncExternalStore`, which means:

- there is a shared app store
- components subscribe to slices
- non-React code can still receive `getState` / `setState`
- rendering remains decoupled from the core runtime

This is a strong compromise:

- React/Ink is used where it shines: terminal UI
- the agent runtime itself is not forced to become React-driven

That matters in a codebase that supports REPL, headless, bridge, and SDK-style execution paths.

### 9. The most impressive trait: deliberate boundaries at scale

The most ingenious part of the architecture is not one isolated module. It is the repeated discipline of keeping boundaries intact:

- command registration ≠ tool registration
- tool registration ≠ permission resolution
- permission resolution ≠ tool execution
- tool execution ≠ query orchestration
- query orchestration ≠ rendering

For a large TypeScript CLI application, that kind of boundary discipline is rare. It is what lets the repository grow into:

- bridge / IDE mode
- swarm / coordinator mode
- MCP integrations
- plugin / skill systems
- multiple permission models

without collapsing into one giant main loop.

### 10. In one sentence

If you read the source directly, Claude Code is best understood as:

> **a layered agent runtime where commands express user intent, tools express model capabilities, the query loop executes a stateful model/tool protocol, and permissions, state, and UI remain separate control planes around that loop.**

That separation of concerns is what gives the architecture both precision and scalability.

### 11. Detailed responsibility split: `main.tsx`, `QueryEngine.ts`, and `query.ts`

If you treat these three files as a group, they map cleanly to three different responsibilities:

| File | Primary responsibility | Why this split is elegant |
|---|---|---|
| `src/main.tsx` | Process startup, runtime assembly, initial configuration, acquiring tools / commands / AppState | Separates “how the program starts” from “how a turn executes” |
| `src/QueryEngine.ts` | Conversation lifecycle, message accumulation, permission-denial tracking, system-prompt assembly, SDK/headless coordination | Pulls multi-turn session state out of the single-turn algorithm |
| `src/query.ts` | Single-turn state machine, streaming events, tool loop, compaction, budget, recovery logic | Keeps the execution protocol as a relatively pure async generator |

In simpler terms:

- `main.tsx` decides **how the system is assembled**
- `QueryEngine.ts` decides **how a conversation persists**
- `query.ts` decides **how one turn advances to completion**

That split is valuable because “what startup needs” and “what a single query needs” are not the same problem. If they were forced together, the main loop would become much harder to reason about.

### 12. How `tools`, `permissions`, and `AppState` are decoupled but coordinated

One of the most sophisticated traits in the source is that these three layers are not hard-wired together.

#### `tools`

`src/Tool.ts` and `src/tools.ts` define and register capabilities. Tools mainly express:

- name
- input schema
- description
- execution logic
- concurrency safety

In other words, tools are treated as capability modules, not global state containers.

#### `permissions`

`src/hooks/useCanUseTool.tsx` extracts permission checking into a dedicated `CanUseToolFn`.
It:

- calls `hasPermissionsToUseTool(...)`
- decides allow / deny / ask
- enters interactive confirmation or classifier flows when needed

So permissions are not implemented ad hoc inside every tool. They run through one decision pipeline.

#### `AppState`

`src/state/AppStateStore.ts` stores runtime state, and `toolPermissionContext` is one of the key inputs read by the permission layer.

The important part is that `QueryEngine.ts` does not bind itself directly to a concrete React store implementation. It only receives:

- `getAppState: () => AppState`
- `setAppState: (f) => void`

That creates a clean relationship:

```text
Tools define capabilities
The permission layer decides whether they may run
AppState stores permission and runtime state
QueryEngine reads and updates state only through callbacks
```

This keeps the layers coordinated without tangling them together.

### 13. A call / data-flow diagram that is closer to the source

The following diagram is a closer representation of the actual boundaries in the source tree:

```text
main.tsx
  ├─ initialize environment / config / auth / telemetry
  ├─ getTools() / getCommands()
  ├─ prepare getAppState / setAppState / canUseTool
  └─ call QueryEngine.submitMessage(...)
             │
             ▼
     QueryEngine.ts
       ├─ maintain mutableMessages / totalUsage / permissionDenials
       ├─ fetchSystemPromptParts(...)
       ├─ wrap wrappedCanUseTool(...)
       └─ call query(...)
                │
                ▼
            query.ts
              ├─ queryLoop()
              ├─ call the Claude API
              ├─ parse assistant messages / tool_use blocks
              ├─ runTools(...)
              ├─ compact / budget / recovery
              └─ yield Message / StreamEvent
                       │
                       ▼
      services/tools/toolOrchestration.ts
        ├─ partitionToolCalls(...)
        ├─ execute read-only tools concurrently
        └─ execute stateful tools serially
                       │
                       ▼
         hooks/useCanUseTool.tsx
           ├─ hasPermissionsToUseTool(...)
           ├─ allow / deny / ask
           └─ update AppState-backed permission context when needed
```

The most important point in this diagram is that **`query.ts` can do its work without depending on the UI component tree, while permission and state layers can still intervene at the right boundaries.**

### 14. A few additional details that become more impressive on close reading

#### Detail A: `QueryEngine` extends permission behavior by wrapping, not rewriting

In `src/QueryEngine.ts`, `submitMessage()` wraps the injected `canUseTool` into `wrappedCanUseTool`, adding `permissionDenials` tracking for SDK reporting without re-implementing the permission system itself.

That is elegant because:

- permission decisions still belong to the permission layer
- the conversation layer only adds conversation-scoped responsibilities such as result tracking

It is a very disciplined extension point.

#### Detail B: `query()` is an async generator, which naturally matches a streaming protocol

`src/query.ts` uses `AsyncGenerator` for `query()` / `queryLoop()`. That is not just a stylistic choice; it is extremely well suited to this runtime:

- it can stream messages/events incrementally upstream
- it can pause naturally between tool loops, recovery, compaction, and interruption handling
- it can model “the request” as a true flow instead of a pile of callbacks

That makes it easier for REPL, SDK, and headless modes to share the same execution core.

#### Detail C: `AppState` is treated as a state container, not the center of business logic

`src/state/AppStateStore.ts` holds a lot of state, but the core logic is not pushed into the store itself.

That suggests deliberate restraint:

- the state layer stores facts
- QueryEngine, the permission layer, and tool orchestration advance the workflow

The boundaries remain understandable even as the state surface grows.

#### Detail D: tool concurrency is not a global switch, but a property-driven batching strategy

`src/services/tools/toolOrchestration.ts` does not simply “allow concurrency” or “disable concurrency”. It first runs `partitionToolCalls(...)`, then decides which blocks can run in parallel and which must run serially.

That reflects a mature runtime mindset:
**concurrency should serve correctness first, not performance alone.**

---

## Key Files in Detail

### `QueryEngine.ts`

The core engine for LLM API calls. Handles streaming responses, tool-call loops, thinking mode, retry logic, and token counting.

### `Tool.ts`

Defines base types and interfaces for all tools — input schemas, permission models, and progress state types.

### `commands.ts`

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
