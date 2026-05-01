# Learning Roadmap — deepagents Codebase

This roadmap guides you through the codebase in dependency order: foundational concepts first, complex integrations last. Each stage builds on the previous one.

---

## How to Use This Roadmap

- Each **Stage** is a logical learning unit
- **Read** links point to the generated docs (which have all classes/functions explained)
- **Source** links point to the actual code when you're ready to go deeper
- Dependencies are listed so you know what you must understand before each stage

---

## Stage 0 — Orientation (15 min)

**Goal:** Understand what the project is and how the packages relate.

| Read | What you'll learn |
|---|---|
| [Root README](../README.md) | Project goals, use cases, quick install |
| [AGENTS.md](../AGENTS.md) | Development conventions and contribution rules |
| [docs/README.md](README.md) | How this documentation is organized |
| [docs/libs/README.md](libs/README.md) | Package dependency graph (SDK → CLI → ACP) |

**Key insight:** deepagents is a modular agent framework. The core SDK (`deepagents`) is fully standalone. The CLI (`deepagents-cli`) wraps it with a terminal UI. The ACP adapter (`deepagents-acp`) makes it network-accessible. Everything else (evals, partners) is optional.

---

## Stage 1 — Python and LangGraph Foundations (prerequisite)

**Goal:** Ensure you understand the frameworks deepagents builds on.

**External resources to read first:**
- [LangGraph Quickstart](https://langchain-ai.github.io/langgraph/tutorials/introduction/) — understand `StateGraph`, `CompiledStateGraph`, `checkpointer`
- [LangChain Tools](https://python.langchain.com/docs/how_to/custom_tools/) — understand `BaseTool`, `@tool`, tool schemas
- [LangChain Chat Models](https://python.langchain.com/docs/integrations/chat/) — understand `BaseChatModel`, streaming, tool binding

**Key concepts to understand before proceeding:**
- LangGraph nodes + edges
- Agent state (TypedDict)
- Checkpointers (state persistence between turns)
- LangChain tool schemas (JSON Schema)
- Streaming agent responses

---

## Stage 2 — Core SDK: Data Types and Model Resolution (30 min)

**Goal:** Understand the SDK's type system and how models are resolved.

| Read | Source | What you'll learn |
|---|---|---|
| [_version.md](libs/deepagents/deepagents/_version.md) | `libs/deepagents/deepagents/_version.py` | Package version constant |
| [_models.md](libs/deepagents/deepagents/_models.md) | `libs/deepagents/deepagents/_models.py` | How `"anthropic:claude-opus-4-6"` strings get resolved to `BaseChatModel` instances |
| [__init__.md](libs/deepagents/deepagents/__init__.md) | `libs/deepagents/deepagents/__init__.py` | Public API surface — what users import |

**Key functions:**
- `resolve_model(model_spec)` — converts string or object to `BaseChatModel`; applies registered `ProviderProfile` behaviors automatically
- `get_model_identifier(model)` — extracts provider:model string from a live model instance
- `get_model_provider(model)` — extracts the provider name (e.g., `"anthropic"`, `"openai"`)
- `model_matches_spec(model, spec)` — checks if a model matches a given spec
- `register_provider_profile(provider, profile)` — extend framework with custom per-provider behaviors

**Key concept:** The **Profiles API** lets you register provider-specific transformations (e.g., enabling OpenAI Responses API, injecting custom HTTP headers) that apply automatically whenever a model from that provider is resolved.

**Dependencies:** None. These are pure utility functions.

---

## Stage 3 — Core SDK: Backend Protocol (1–2 hours)

**Goal:** Understand how deepagents abstracts file and shell access.

This is the most foundational architectural concept in the SDK. All other layers (middleware, tools, CLI) build on it.

### 3a — The Protocol (interfaces)

| Read | Source | What you'll learn |
|---|---|---|
| [backends/README.md](libs/deepagents/deepagents/backends/README.md) | `libs/deepagents/deepagents/backends/` | Overview of backend types and selection guide |
| [backends/protocol.md](libs/deepagents/deepagents/backends/protocol.md) | `libs/deepagents/deepagents/backends/protocol.py` | `BackendProtocol` interface, all result types (`ReadResult`, `WriteResult`, etc.) |

**Key insight:** Every operation (read, write, edit, ls, grep, glob, execute) has a well-defined TypedDict result. The `files_update` field in write results is how in-memory backends sync state back to LangGraph checkpoints.

### 3b — Backend Implementations (read in order)

| Read | Source | What you'll learn |
|---|---|---|
| [backends/utils.md](libs/deepagents/deepagents/backends/utils.md) | `backends/utils.py` | Shared helpers: file formatting, path validation, ripgrep/Python fallback search |
| [backends/state.md](libs/deepagents/deepagents/backends/state.md) | `backends/state.py` | `StateBackend` — ephemeral in-memory storage in LangGraph state (the default) |
| [backends/filesystem.md](libs/deepagents/deepagents/backends/filesystem.md) | `backends/filesystem.py` | `FilesystemBackend` — direct disk access, virtual path mode |
| [backends/sandbox.md](libs/deepagents/deepagents/backends/sandbox.md) | `backends/sandbox.py` | `BaseSandbox` — abstract base that implements ALL file ops via `execute()` shell commands |
| [backends/langsmith.md](libs/deepagents/deepagents/backends/langsmith.md) | `backends/langsmith.py` | `LangSmithSandbox` — extends `BaseSandbox` with LangSmith-native write/upload |
| [backends/local_shell.md](libs/deepagents/deepagents/backends/local_shell.md) | `backends/local_shell.py` | `LocalShellBackend` — `FilesystemBackend` + unrestricted local shell |
| [backends/store.md](libs/deepagents/deepagents/backends/store.md) | `backends/store.py` | `StoreBackend` — persistent cross-thread storage via LangGraph `BaseStore` |
| [backends/composite.md](libs/deepagents/deepagents/backends/composite.md) | `backends/composite.py` | `CompositeBackend` — routes ops to different backends by path prefix |

**Key design patterns:**
- `StateBackend` returns `files_update={path: data}` → middleware merges into LangGraph state via `Command`
- External backends (`FilesystemBackend`, `StoreBackend`) return `files_update=None` → already persisted
- `BaseSandbox.execute(cmd)` is the single abstract method; all file ops are implemented as shell templates

**Dependencies:** LangGraph concepts (State, Store, Checkpointer)

---

## Stage 4 — Core SDK: Middleware System (2–3 hours)

**Goal:** Understand how capabilities are injected into agents at call time.

The middleware system is the primary extension point. Every feature (file tools, subagents, memory, skills) is a middleware.

### 4a — The Pattern

| Read | Source | What you'll learn |
|---|---|---|
| [middleware/README.md](libs/deepagents/deepagents/middleware/README.md) | `libs/deepagents/deepagents/middleware/` | Middleware interface, dependency graph, default stack ordering |
| [middleware/__init__.md](libs/deepagents/deepagents/middleware/__init__.md) | `middleware/__init__.py` | Exports, middleware vs. plain tools (why middleware exists) |
| [middleware/_utils.md](libs/deepagents/deepagents/middleware/_utils.md) | `middleware/_utils.py` | `append_to_system_message` — the core utility every middleware uses |

### 4b — Individual Middleware (read in dependency order)

| Read | Source | What you'll learn |
|---|---|---|
| [middleware/filesystem.md](libs/deepagents/deepagents/middleware/filesystem.md) | `middleware/filesystem.py` | All 7 file tools, large result eviction, dynamic tool injection based on backend capabilities |
| [middleware/memory.md](libs/deepagents/deepagents/middleware/memory.md) | `middleware/memory.py` | How `AGENTS.md` files become agent context; memory update instructions |
| [middleware/skills.md](libs/deepagents/deepagents/middleware/skills.md) | `middleware/skills.py` | `SKILL.md` discovery, progressive disclosure pattern, layered skill sources |
| [middleware/subagents.md](libs/deepagents/deepagents/middleware/subagents.md) | `middleware/subagents.py` | The `task` tool — how synchronous subagents are compiled and invoked |
| [middleware/async_subagents.md](libs/deepagents/deepagents/middleware/async_subagents.md) | `middleware/async_subagents.py` | Background tasks on remote LangGraph deployments via SDK |
| [middleware/summarization.md](libs/deepagents/deepagents/middleware/summarization.md) | `middleware/summarization.py` | Context compaction — how long conversations are summarized to stay within token limits |
| [middleware/patch_tool_calls.md](libs/deepagents/deepagents/middleware/patch_tool_calls.md) | `middleware/patch_tool_calls.py` | Why dangling tool calls break agents; how synthetic ToolMessages fix them |

**Key insight:** `wrap_model_call(input)` is called before every LLM request. Middleware can add tools, inject system prompts, or transform the input. `before_agent()` runs once before the first turn — used for loading files from the backend.

**Dependencies:** Stage 3 (Backend Protocol)

---

## Stage 5 — Core SDK: The Agent Factory (1 hour)

**Goal:** Understand how all pieces are wired together.

| Read | Source | What you'll learn |
|---|---|---|
| [deepagents/graph.md](libs/deepagents/deepagents/graph.md) | `libs/deepagents/deepagents/graph.py` | `create_deep_agent()` — the full assembly sequence (8 steps) |
| [pyproject.toml.md](libs/deepagents/pyproject.toml.md) | `libs/deepagents/pyproject.toml` | All SDK dependencies, optional extras, dev tooling |

**The assembly in `create_deep_agent()`:**
1. Model resolution (`resolve_model` + `apply_provider_profile`)
2. Backend defaults (`StateBackend`)
3. Harness profile lookup (`_harness_profile_for_model`) — per-model prompt/tool/middleware overrides
4. `general-purpose` subagent construction (auto-added unless already provided or profile disables it)
5. Subagent type separation (sync vs. async)
6. Per-subagent middleware stacks
7. Main agent middleware stack assembly (base → user → tail)
8. System prompt merging (user prompt → base/profile prompt → profile suffix)
9. LangGraph `create_agent()` call + config wrapping

**After this stage:** You can read any example agent and understand exactly what it's building.

---

## Stage 6 — Examples: Applying the SDK (1–2 hours)

**Goal:** See `create_deep_agent()` in action across different use cases.

Read these examples in order of complexity:

| Read | Source | What you'll learn |
|---|---|---|
| [text-to-sql-agent/agent.md](examples/text-to-sql-agent/agent.md) | `examples/text-to-sql-agent/agent.py` | Simplest usage: custom tools + FilesystemBackend |
| [deep_research/agent.md](examples/deep_research/agent.md) | `examples/deep_research/agent.py` | Multi-agent pattern: orchestrator + research subagents |
| [deep_research/research_agent/README.md](examples/deep_research/research_agent/README.md) | `examples/deep_research/research_agent/` | Custom prompts and tools for a subagent |
| [content-builder-agent/content_writer.md](examples/content-builder-agent/content_writer.md) | `examples/content-builder-agent/content_writer.py` | Skills + YAML-defined subagents |
| [nvidia_deep_agent/src/agent.md](examples/nvidia_deep_agent/src/agent.md) | `examples/nvidia_deep_agent/src/agent.py` | Multi-model agents, Modal GPU sandbox |
| [nvidia_deep_agent/src/backend.md](examples/nvidia_deep_agent/src/backend.md) | `examples/nvidia_deep_agent/src/backend.py` | How to build a custom backend (Modal) |

**Dependencies:** Stages 2–5

---

## Stage 7 — Partner Sandbox Packages (1 hour)

**Goal:** Understand how cloud sandbox integrations are built.

| Read | Source | What you'll learn |
|---|---|---|
| [partners/README.md](libs/partners/README.md) | `libs/partners/` | Overview of all partner packages |
| [partners/modal/sandbox.md](libs/partners/modal/sandbox.md) | `libs/partners/modal/` | `ModalSandbox` — serverless GPU sandbox via Modal |
| [partners/daytona/sandbox.md](libs/partners/daytona/sandbox.md) | `libs/partners/daytona/` | `DaytonaSandbox` — persistent cloud workspaces |
| [partners/runloop/sandbox.md](libs/partners/runloop/sandbox.md) | `libs/partners/runloop/` | `RunloopSandbox` — managed sandbox environments |
| [partners/quickjs/middleware.md](libs/partners/quickjs/middleware.md) | `libs/partners/quickjs/` | `REPLMiddleware` — persistent JS REPL backed by quickjs-rs, with PTC and skill module imports (unique: not a sandbox, but middleware) |

**Key insight:** Modal, Daytona, and Runloop all subclass `BaseSandbox` and only need to implement `execute(cmd)`. QuickJS is different — it's a middleware that provides a JS REPL tool.

**Dependencies:** Stage 3b (`BaseSandbox`)

---

## Stage 8 — CLI: Architecture Overview (1 hour)

**Goal:** Understand how the TUI wraps the SDK.

| Read | Source | What you'll learn |
|---|---|---|
| [cli/README.md](libs/cli/README.md) | `libs/cli/` | Full feature list, all slash commands, configuration |
| [cli/deepagents_cli/__main__.md](libs/cli/deepagents_cli/__main__.md) | `deepagents_cli/__main__.py` | Entry point |
| [cli/deepagents_cli/main.md](libs/cli/deepagents_cli/main.md) | `deepagents_cli/main.py` | CLI argument parsing, startup sequence |
| [cli/deepagents_cli/config.md](libs/cli/deepagents_cli/config.md) | `deepagents_cli/config.py` | `Settings` dataclass, config file layout (`~/.deepagents/config.toml`) |
| [cli/deepagents_cli/model_config.md](libs/cli/deepagents_cli/model_config.md) | `deepagents_cli/model_config.py` | `ModelSpec`, provider discovery, profile loading |

**Dependencies:** Stage 5 (Agent Factory)

---

## Stage 9 — CLI: Session and Server Management (1 hour)

**Goal:** Understand how the CLI manages long-running sessions and the LangGraph server.

| Read | Source | What you'll learn |
|---|---|---|
| [cli/deepagents_cli/sessions.md](libs/cli/deepagents_cli/sessions.md) | `deepagents_cli/sessions.py` | Thread persistence via SQLite, `ThreadInfo`, resumable sessions |
| [cli/deepagents_cli/server.md](libs/cli/deepagents_cli/server.md) | `deepagents_cli/server.py` | How the CLI spawns and manages a `langgraph dev` subprocess |
| [cli/deepagents_cli/_server_config.md](libs/cli/deepagents_cli/_server_config.md) | `deepagents_cli/_server_config.py` | `ServerConfig` — the CLI↔server env var protocol |
| [cli/deepagents_cli/agent.md](libs/cli/deepagents_cli/agent.md) | `deepagents_cli/agent.py` | How the CLI assembles the full middleware stack for a session |

**Key insight:** The CLI does NOT embed the agent directly — it spawns a `langgraph dev` subprocess (LangGraph server) and communicates with it via the LangGraph SDK client. This enables persistence, hot-reload, and ACP mode.

---

## Stage 10 — CLI: TUI Widgets (2 hours)

**Goal:** Understand the interactive terminal interface components.

This stage is only needed if you plan to work on the UI.

### 10a — Core application

| Read | Source | What you'll learn |
|---|---|---|
| [cli/deepagents_cli/app.md](libs/cli/deepagents_cli/app.md) | `deepagents_cli/app.py` | `DeepAgentsApp` — the top-level Textual app, screen layout, message routing |

### 10b — Input and Output widgets

| Read | Source | What you'll learn |
|---|---|---|
| [widgets/chat_input.md](libs/cli/deepagents_cli/widgets/chat_input.md) | `widgets/chat_input.py` | `ChatInput` — multi-line input with history and autocomplete |
| [widgets/messages.md](libs/cli/deepagents_cli/widgets/messages.md) | `widgets/messages.py` | All message display widgets (user, assistant, tool calls, errors) |
| [widgets/message_store.md](libs/cli/deepagents_cli/widgets/message_store.md) | `widgets/message_store.py` | `MessageStore` — virtualized chat history backing store |
| [widgets/tool_widgets.md](libs/cli/deepagents_cli/widgets/tool_widgets.md) | `widgets/tool_widgets.py` | Tool-specific HITL preview widgets (file writes, diffs) |
| [widgets/approval.md](libs/cli/deepagents_cli/widgets/approval.md) | `widgets/approval.py` | `ApprovalMenu` — the HITL approve/reject/always-approve interaction |

### 10c — Navigation and configuration modals

| Read | Source | What you'll learn |
|---|---|---|
| [widgets/model_selector.md](libs/cli/deepagents_cli/widgets/model_selector.md) | `widgets/model_selector.py` | `/model` command — interactive model picker |
| [widgets/thread_selector.md](libs/cli/deepagents_cli/widgets/thread_selector.md) | `widgets/thread_selector.py` | `/threads` command — session browser |
| [widgets/theme_selector.md](libs/cli/deepagents_cli/widgets/theme_selector.md) | `widgets/theme_selector.py` | `/theme` command — live theme preview |
| [widgets/mcp_viewer.md](libs/cli/deepagents_cli/widgets/mcp_viewer.md) | `widgets/mcp_viewer.py` | `/mcp` command — MCP server/tool inspector |

### 10d — Supporting widgets

| Read | Source | What you'll learn |
|---|---|---|
| [widgets/autocomplete.md](libs/cli/deepagents_cli/widgets/autocomplete.md) | `widgets/autocomplete.py` | Slash command and `@file` mention autocomplete |
| [widgets/status.md](libs/cli/deepagents_cli/widgets/status.md) | `widgets/status.py` | Status bar — model name, token counts |
| [widgets/loading.md](libs/cli/deepagents_cli/widgets/loading.md) | `widgets/loading.py` | Animated thinking/loading indicator |
| [widgets/diff.md](libs/cli/deepagents_cli/widgets/diff.md) | `widgets/diff.py` | Syntax-highlighted unified diff renderer |

**External prerequisite:** [Textual documentation](https://textual.textualize.io/) — understand `App`, `Widget`, `Screen`, reactive attributes, and message posting.

---

## Stage 11 — CLI: Integrations and Skills (30 min)

**Goal:** Understand how MCP tools and skills work in the CLI.

| Read | Source | What you'll learn |
|---|---|---|
| [cli/deepagents_cli/mcp_tools.md](libs/cli/deepagents_cli/mcp_tools.md) | `deepagents_cli/mcp_tools.py` | MCP server loading, `MCPServerInfo`, OAuth login (`auth: "oauth"`), per-server `allowedTools`/`disabledTools` filtering |
| [cli/deepagents_cli/hooks.md](libs/cli/deepagents_cli/hooks.md) | `deepagents_cli/hooks.py` | External hook dispatch (pre/post message, tool execution events) |
| [cli/deepagents_cli/subagents.md](libs/cli/deepagents_cli/subagents.md) | `deepagents_cli/subagents.py` | AGENTS.md-based subagent loading for the CLI |
| [cli/deepagents_cli/skills/README.md](libs/cli/deepagents_cli/skills/README.md) | `deepagents_cli/skills/` | Skill discovery and the `/skills` CLI commands |
| [cli/deepagents_cli/integrations/README.md](libs/cli/deepagents_cli/integrations/README.md) | `deepagents_cli/integrations/` | `SandboxFactory` — how `--sandbox modal` selects a backend |

---

## Stage 12 — ACP Protocol Adapter (1 hour)

**Goal:** Understand how deepagents becomes a network service.

| Read | Source | What you'll learn |
|---|---|---|
| [acp/README.md](libs/acp/README.md) | `libs/acp/` | What ACP is, session lifecycle, HITL flow, plan display |
| [acp/deepagents_acp/utils.md](libs/acp/deepagents_acp/utils.md) | `deepagents_acp/utils.py` | Content block converters, shell command signature extraction |
| [acp/deepagents_acp/server.md](libs/acp/deepagents_acp/server.md) | `deepagents_acp/server.py` | `AgentServerACP` — the full ACP adapter implementation |
| [acp/deepagents_acp/__main__.md](libs/acp/deepagents_acp/__main__.md) | `deepagents_acp/__main__.py` | Entry point for `python -m deepagents_acp` |
| [acp/examples/demo_agent.md](libs/acp/examples/demo_agent.md) | `libs/acp/examples/demo_agent.py` | Full production-ready ACP agent with multi-mode and model switching |

**Key concepts:**
- ACP session lifecycle: `new_session` → `prompt` → `cancel`
- HITL via `__interrupt__` → `client.request_permission()` → approve/reject/always
- `approve_always` adds command to allowlist via `extract_command_types()` normalization

**Dependencies:** Stages 5–9 (SDK + CLI architecture)

---

## Stage 13 — Evals and Benchmarks (1 hour)

**Goal:** Understand how agent quality is measured.

| Read | Source | What you'll learn |
|---|---|---|
| [evals/README.md](libs/evals/README.md) | `libs/evals/` | Overview of evaluation strategy |
| [evals/deepagents_evals/README.md](libs/evals/deepagents_evals/README.md) | `deepagents_evals/` | Unit-test-style eval framework |
| [evals/deepagents_harbor/README.md](libs/evals/deepagents_harbor/README.md) | `deepagents_harbor/` | Harbor terminal-bench evaluation |
| [evals/deepagents_harbor/deepagents_wrapper.md](libs/evals/deepagents_harbor/deepagents_wrapper.md) | `deepagents_harbor/deepagents_wrapper.py` | Harbor↔deepagents integration |
| [evals/deepagents_evals/radar.md](libs/evals/deepagents_evals/radar.md) | `deepagents_evals/radar.py` | Radar chart generation for multi-model comparison |
| [evals/Makefile.md](libs/evals/Makefile.md) | `libs/evals/Makefile` | How to run evals across different model providers |

---

## Stage 14 — DevOps and CI/CD (30–60 min)

**Goal:** Understand the development infrastructure.

| Read | What you'll learn |
|---|---|
| [devops/README.md](devops/README.md) | Full CI/CD architecture, secrets, design decisions |
| [devops/github-workflows/ci.md](devops/github-workflows/ci.md) | Main CI pipeline: change detection → lint → test → benchmark |
| [devops/github-workflows/release.md](devops/github-workflows/release.md) | Release pipeline: build → test-pypi → publish → GitHub release |
| [devops/pre-commit-config.md](devops/pre-commit-config.md) | Local development hooks: linting, version sync, lockfile checks |
| [devops/Makefile.md](devops/Makefile.md) | Root monorepo Makefile targets |
| [devops/github-workflows/README.md](devops/github-workflows/README.md) | All 20 workflows at a glance |

---

## Summary: Learning Order at a Glance

```
Stage 0: Orientation (project overview)
    ↓
Stage 1: LangGraph + LangChain (external prerequisite)
    ↓
Stage 2: SDK data types + model resolution
    ↓
Stage 3: Backend Protocol (BackendProtocol → StateBackend → FilesystemBackend → BaseSandbox → ...)
    ↓
Stage 4: Middleware System (FilesystemMiddleware → MemoryMiddleware → SkillsMiddleware → ...)
    ↓
Stage 5: Agent Factory (create_deep_agent — wires everything together)
    ↓
Stage 6: Examples (SDK in practice)
    ↓
Stage 7: Partner sandboxes ─────────────────────────────┐
    │                                                    │
Stage 8: CLI architecture overview ←────────────────────┘
    ↓
Stage 9: CLI sessions + server management
    ↓
Stage 10: TUI widgets (optional, for UI work)
    ↓
Stage 11: CLI integrations (MCP, hooks, skills, subagents)
    ↓
Stage 12: ACP protocol adapter
    ↓
Stage 13: Evals and benchmarks
    ↓
Stage 14: DevOps and CI/CD
```

---

## Component Dependency Map

The following shows which source files you must understand before others:

```
_version.py           (no deps)
_models.py            → _version.py
backends/protocol.py  (no SDK deps)
backends/utils.py     → backends/protocol.py
backends/state.py     → backends/protocol.py
backends/filesystem.py → backends/protocol.py, utils.py
backends/sandbox.py   → backends/protocol.py, utils.py
backends/langsmith.py → backends/sandbox.py
backends/local_shell.py → backends/filesystem.py
backends/store.py     → backends/protocol.py
backends/composite.py → backends/protocol.py, all other backends

middleware/_utils.py  (no SDK deps)
middleware/filesystem.py → backends/*, middleware/_utils.py
middleware/memory.py  → backends/protocol.py, middleware/_utils.py
middleware/skills.py  → backends/protocol.py, middleware/_utils.py
middleware/subagents.py → backends/protocol.py, middleware/_utils.py
middleware/async_subagents.py → middleware/_utils.py, langgraph_sdk
middleware/summarization.py → backends/protocol.py, middleware/_utils.py
middleware/patch_tool_calls.py → middleware/_utils.py

graph.py              → _models.py, backends/*, middleware/*

deepagents_cli/*      → graph.py (via deepagents.create_deep_agent)
deepagents_acp/*      → graph.py (via deepagents.create_deep_agent)
deepagents_evals/*    → deepagents_cli (via harbor) + graph.py
partners/*/           → backends/sandbox.py (via BaseSandbox)
examples/*/           → graph.py (via deepagents.create_deep_agent)
```

---

## Recommended First Contribution Path

If you want to make your first code change, here is the recommended path:

1. **Read Stage 0–5** to understand the core architecture
2. **Run the CLI** locally: `cd libs/cli && make install && deepagents`
3. **Run the tests**: `cd libs/deepagents && make test`
4. **Pick a small area** — for example:
   - A new backend: subclass `BaseSandbox`, implement `execute()`
   - A new middleware: implement `wrap_model_call()` and `before_agent()`
   - A CLI widget: add a new Textual widget in `deepagents_cli/widgets/`
5. **Follow AGENTS.md** for code conventions and commit message format
