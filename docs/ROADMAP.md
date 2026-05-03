# deepagents Learning Roadmap

A dependency-ordered path through the codebase. Each stage builds on the previous one. No stage has a forward dependency.

---

## How to Use This Roadmap

- Each stage is a self-contained learning unit
- **Doc links** point to docs (high-level understanding)
- **Source links** point to actual source files (implementation detail)
- Time estimates are for reading docs only; add 2–3× for exploring source code
- Skip stages that cover tools you already know (e.g., skip Stage 1 if you know LangGraph)

---

## Stage 1 — Prerequisites (external reading, ~1 hour)

**Goal:** Understand the frameworks deepagents is built on before looking at any deepagents code.

**Concepts to internalize:**

| Concept | Resource | What you need to understand |
|---|---|---|
| LangGraph basics | [LangGraph docs](https://langchain-ai.github.io/langgraph/) | `CompiledStateGraph`, `invoke()`, `astream()`, `astream_events()`, checkpointing, interrupts |
| LangChain tools | [LangChain tools docs](https://python.langchain.com/docs/concepts/tools/) | `BaseTool`, `StructuredTool`, tool binding to models |
| LangChain models | [LangChain chat models](https://python.langchain.com/docs/concepts/chat_models/) | `BaseChatModel`, `init_chat_model()` |
| Textual TUI | [Textual docs](https://textual.textualize.io/) | `App`, `Widget`, `Screen`, reactive properties, `push_screen()` |

**Key insight:** deepagents is a thin configuration layer on top of LangGraph. Understanding LangGraph's state machine model, interrupt mechanism, and SSE streaming is essential for everything that follows.

---

## Stage 2 — Repository Orientation (15 min)

**Goal:** Know what each package does and how they depend on each other.

| Read | What you'll learn |
|---|---|
| [libs/README.md](libs/README.md) | Package dependency graph; when to use which package |
| Root `README.md` | Project mission, current version, quick start |
| Root `AGENTS.md` | Development conventions, code style, commit format |

**Key concepts:**
- `deepagents` (SDK) is the core; everything else wraps it
- The CLI doesn't embed the agent — it talks to a LangGraph server subprocess
- Partner backends (Modal, Daytona, etc.) implement `BackendProtocol`

**Dependencies:** Stage 1

---

## Stage 3 — The SDK Entry Point (30 min)

**Goal:** Understand what `create_deep_agent()` does and what it returns.

| Read | Source | What you'll learn |
|---|---|---|
| [libs/deepagents/deepagents/graph.md](libs/deepagents/deepagents/graph.md) | `libs/deepagents/deepagents/graph.py` | All parameters, default middleware stack, system prompt assembly |
| [libs/deepagents/deepagents/README.md](libs/deepagents/deepagents/README.md) | — | The three-layer model (backend + middleware + model) |
| [libs/deepagents/deepagents/_models.md](libs/deepagents/deepagents/_models.md) | `libs/deepagents/deepagents/_models.py` | How model strings are resolved |

**Key insights:**
- All parameters to `create_deep_agent()` are optional
- The return value is a standard LangGraph `CompiledStateGraph` — any LangGraph pattern works with it
- Middleware runs on every model call, not once at startup

**Dependencies:** Stages 1, 2

---

## Stage 4 — Backend Protocol (20 min)

**Goal:** Understand how file and shell access is abstracted.

| Read | Source | What you'll learn |
|---|---|---|
| [libs/deepagents/deepagents/backends/README.md](libs/deepagents/deepagents/backends/README.md) | — | All backend types and when to use each |
| [libs/deepagents/deepagents/backends/protocol.md](libs/deepagents/deepagents/backends/protocol.md) | `backends/protocol.py` | `BackendProtocol` interface, file data format, permission rules |

**Key insights:**
- Every backend presents the same interface; swapping backends requires no agent code changes
- File data is always `{content, encoding, created_at, modified_at}`
- `execute()` only exists on `SandboxBackendProtocol` — if the backend doesn't have it, there's no shell tool

**Dependencies:** Stages 2, 3

---

## Stage 5 — FilesystemMiddleware (25 min)

**Goal:** Understand the tools the agent actually uses to read, write, and run things.

| Read | Source | What you'll learn |
|---|---|---|
| [libs/deepagents/deepagents/middleware/README.md](libs/deepagents/deepagents/middleware/README.md) | — | Middleware stack ordering and composition model |
| [libs/deepagents/deepagents/middleware/filesystem.md](libs/deepagents/deepagents/middleware/filesystem.md) | `middleware/filesystem.py` | All six built-in tools: ls, read_file, write_file, edit_file, glob, grep, execute |

**Key insights:**
- `FilesystemMiddleware` is required — it cannot be excluded
- `edit_file` requires exact string match by design (forces the agent to read first)
- Permissions are path-based rules checked at tool-call time, not backend time

**Dependencies:** Stage 4

---

## Stage 6 — CLI Startup Flow (45 min)

**Goal:** Trace exactly what happens between `deepagents` (shell command) and "ready for input".

This is the most important stage for understanding the CLI architecture.

| Read | Source | What you'll learn |
|---|---|---|
| [libs/cli/deepagents_cli/README.md](libs/cli/deepagents_cli/README.md) | — | CLI architecture overview and key design decisions |
| [libs/cli/deepagents_cli/main.md](libs/cli/deepagents_cli/main.md) | `main.py` | Argument parsing, mode dispatch, deferred imports |
| [libs/cli/deepagents_cli/config.md](libs/cli/deepagents_cli/config.md) | `config.py` | Settings singleton, dotenv loading, lazy bootstrap |
| [libs/cli/deepagents_cli/server_manager.md](libs/cli/deepagents_cli/server_manager.md) | `server_manager.py` | Workspace scaffolding, `langgraph dev` subprocess start |
| [libs/cli/deepagents_cli/server.md](libs/cli/deepagents_cli/server.md) | `server.py` | `ServerProcess`: start, readiness poll, stop |
| [libs/cli/deepagents_cli/server_graph.md](libs/cli/deepagents_cli/server_graph.md) | `server_graph.py` | `make_graph()`: reads env config, creates agent |
| [libs/cli/deepagents_cli/agent.md](libs/cli/deepagents_cli/agent.md) | `agent.py` | `create_cli_agent()`: tools, middleware, backend, HITL config |

**Startup sequence:**
```
cli_main()
 └─ run_textual_cli_async()
     └─ run_textual_app()  [CLIApp]
         └─ CLIApp.on_mount()
             └─ start_server_and_get_agent()
                 ├─ scaffold workspace + write langgraph.json
                 ├─ start "langgraph dev" subprocess
                 │   └─ make_graph() called inside subprocess
                 │       └─ create_cli_agent() → create_deep_agent()
                 ├─ poll /ok until ready
                 └─ return RemoteAgent client
```

**Key insight:** The CLI never imports the agent code directly. The agent runs in a subprocess that the TUI communicates with over HTTP. This is why you can hot-reload the agent without restarting the TUI.

**Dependencies:** Stages 3, 4, 5

---

## Stage 7 — TUI Widget System (30 min)

**Goal:** Understand how the TUI renders and what each widget does.

| Read | Source | What you'll learn |
|---|---|---|
| [libs/cli/deepagents_cli/app.md](libs/cli/deepagents_cli/app.md) | `app.py` | `CLIApp` state, message queue, command routing, `_run_agent()` |
| [libs/cli/deepagents_cli/widgets/README.md](libs/cli/deepagents_cli/widgets/README.md) | `widgets/` | Widget hierarchy, all widget types |
| [libs/cli/deepagents_cli/widgets/chat_input.md](libs/cli/deepagents_cli/widgets/chat_input.md) | `widgets/chat_input.py` | Input field, autocomplete, `@file` mentions |
| [libs/cli/deepagents_cli/widgets/messages.md](libs/cli/deepagents_cli/widgets/messages.md) | `widgets/messages.py` | All message widget types |
| [libs/cli/deepagents_cli/widgets/status.md](libs/cli/deepagents_cli/widgets/status.md) | `widgets/status.py` | Status bar: model, tokens, spinner |

**Key insight:** The message queue (`_message_queue: deque`) and `BypassTier` system are what make commands feel responsive even while the agent is running. IMMEDIATE_UI commands open modals instantly; most others wait.

**Dependencies:** Stage 6

---

## Stage 8 — Message Flow and Streaming (30 min)

**Goal:** Trace a user message from keypress to rendered response.

| Read | Source | What you'll learn |
|---|---|---|
| [libs/cli/deepagents_cli/remote_client.md](libs/cli/deepagents_cli/remote_client.md) | `remote_client.py` | `RemoteAgent.astream()`, `StreamHandler`, UIActions |
| [libs/cli/deepagents_cli/widgets/message_store.md](libs/cli/deepagents_cli/widgets/message_store.md) | `widgets/message_store.py` | O(1) widget lookup by ID |

**Message flow:**
```
ChatInput → InputSubmitted event
 └─ app._handle_submit()
     └─ _enqueue_message()
         └─ _process_queue() [polling timer]
             └─ _run_agent(message)
                 └─ RemoteAgent.astream()  [HTTP POST + SSE]
                     └─ StreamHandler.handle_chunk()
                         └─ UIAction → widget mutation
```

**Key insight:** Token streaming works because LangGraph's `stream_mode=["messages"]` delivers `AIMessageChunk` events. Each chunk appends a few tokens to the `AssistantMessage` widget, producing the typewriter effect.

**Dependencies:** Stages 6, 7

---

## Stage 9 — HITL Approval System (20 min)

**Goal:** Understand how tool call approvals work end-to-end.

| Read | Source | What you'll learn |
|---|---|---|
| [libs/deepagents/deepagents/middleware/human_in_the_loop.md](libs/deepagents/deepagents/middleware/human_in_the_loop.md) | `middleware/human_in_the_loop.py` | How LangGraph interrupts are generated |
| [libs/cli/deepagents_cli/widgets/approval.md](libs/cli/deepagents_cli/widgets/approval.md) | `widgets/approval.py` | `ApprovalMenu` modal — approve, reject, edit |

**HITL flow:**
```
Agent calls "execute" tool
 └─ HumanInTheLoopMiddleware → LangGraph interrupt()
     └─ Server emits __interrupt__ SSE event
         └─ StreamHandler → InterruptAction
             └─ CLIApp._handle_interrupt()
                 └─ push_screen(ApprovalMenu)  ← user sees modal
                     └─ await decision
                         └─ RemoteAgent.resume(decision)
                             └─ Agent continues (or skips if rejected)
```

**Key insight:** LangGraph's native interrupt mechanism does the heavy lifting. The CLI just converts interrupt events to modals and decision dicts back to resume calls.

**Dependencies:** Stages 3, 8

---

## Stage 10 — Sessions and Persistence (15 min)

**Goal:** Understand how conversation threads are stored and resumed.

| Read | Source | What you'll learn |
|---|---|---|
| [libs/cli/deepagents_cli/sessions.md](libs/cli/deepagents_cli/sessions.md) | `sessions.py` | `SessionStore`, thread metadata, `-r` resume flow |

**Key insight:** LangGraph handles checkpoint storage (via `AsyncSqliteSaver`) — that's the full message history. `sessions.py` adds a separate lightweight metadata table (initial prompt, git branch, timestamps) to make threads human-browsable.

**Dependencies:** Stage 6

---

## Stage 11 — MCP Tool Loading (20 min)

**Goal:** Understand how external tools from MCP servers are discovered and loaded.

| Read | Source | What you'll learn |
|---|---|---|
| [libs/cli/deepagents_cli/mcp_tools.md](libs/cli/deepagents_cli/mcp_tools.md) | `mcp_tools.py` | Config discovery, validation, trust, session manager |

**Key insight:** MCP tools are loaded in the server subprocess at startup. They are discovered before the graph is compiled and passed to `create_cli_agent()` like any other `BaseTool`. The MCP session manager keeps stdio subprocesses alive for the duration of the CLI session.

**Dependencies:** Stage 6

---

## Stage 12 — Slash Commands and Skills (15 min)

**Goal:** Understand the slash command system and how skills extend agent behavior.

| Read | Source | What you'll learn |
|---|---|---|
| [libs/cli/deepagents_cli/command_registry.md](libs/cli/deepagents_cli/command_registry.md) | `command_registry.py` | `SlashCommand`, `BypassTier`, all built-in commands |
| [libs/deepagents/deepagents/middleware/skills.md](libs/deepagents/deepagents/middleware/skills.md) | `middleware/skills.py` | Skill file format, injection, priority |

**Key insight:** Skills are prompt templates, not code. The `/skill:name` command injects the skill's content into the system prompt for the next invocation. No new tools are added.

**Dependencies:** Stages 7, 5

---

## Stage 13 — Memory, Summarization, and Non-interactive Mode (20 min)

**Goal:** Understand long-running session management.

| Read | Source | What you'll learn |
|---|---|---|
| [libs/deepagents/deepagents/middleware/memory.md](libs/deepagents/deepagents/middleware/memory.md) | `middleware/memory.py` | `MemoryMiddleware`, AGENTS.md loading, update guidelines |
| [libs/deepagents/deepagents/middleware/summarization.md](libs/deepagents/deepagents/middleware/summarization.md) | `middleware/summarization.py` | Context compaction, trigger threshold, summary storage |
| [libs/cli/deepagents_cli/non_interactive.md](libs/cli/deepagents_cli/non_interactive.md) | `non_interactive.py` | Headless `-n` mode, shell allow-list, max-turns |

**Dependencies:** Stages 5, 8

---

## Stage 14 — Advanced Topics and Subagents (30 min)

**Goal:** Understand multi-agent patterns, hooks, and deployment.

| Read | Source | What you'll learn |
|---|---|---|
| [libs/deepagents/deepagents/middleware/subagents.md](libs/deepagents/deepagents/middleware/subagents.md) | `middleware/subagents.py` | `task` tool, subagent invocation, parallel dispatch |
| [libs/deepagents/deepagents/middleware/async_subagents.md](libs/deepagents/deepagents/middleware/async_subagents.md) | `middleware/async_subagents.py` | Background tasks, polling pattern |
| [libs/cli/deepagents_cli/subagents.md](libs/cli/deepagents_cli/subagents.md) | `subagents.py` | Loading subagent YAML files from `~/.deepagents/agents/` |
| [libs/cli/deepagents_cli/hooks.md](libs/cli/deepagents_cli/hooks.md) | `hooks.py` | Lifecycle hooks, `hooks.json` format |
| [libs/acp/README.md](libs/acp/README.md) | `libs/acp/` | ACP server mode |
| [examples/README.md](examples/README.md) | `examples/` | Example agents for each major pattern |

**Key insight:** For most use cases, synchronous `task` delegation (Stage 14) is simpler — it blocks until the sub-agent returns. Async subagents are only needed when you want the parent to do other work concurrently.

**Dependencies:** Stages 3, 8

---

## After the Roadmap

**Contributing:**
- Read `AGENTS.md` at repo root for commit/PR conventions
- Run `make check` in any package before submitting a PR
- Unit tests live in `tests/unit_tests/` (no network required)

**Finding things not in this roadmap:**
- CI/CD: `.github/workflows/` — 20 workflows; `ci.yml` is the main one
- Release process: `.github/workflows/release.yml` — release-please automation
- Dev notes: `libs/cli/DEV.md` — CLI-specific development tips
- Security model: `libs/cli/THREAT_MODEL.md`
- Eval tasks: `libs/evals/EVAL_CATALOG.md`

**File navigation cheatsheet:**

| Task | File |
|---|---|
| Add a new CLI slash command | `command_registry.py` + `app.py` |
| Add a new backend | Implement `BackendProtocol` in `backends/` |
| Add a new middleware | Extend `AgentMiddleware` in `middleware/` |
| Change default tools | `agent.py::create_cli_agent()` |
| Change the system prompt | `graph.py::BASE_AGENT_PROMPT` or `default_agent_prompt.md` |
| Add a new MCP server | Edit `~/.deepagents/.mcp.json` |
| Add a new skill | Create `~/.deepagents/{agent}/skills/{name}/SKILL.md` |
| Add a new subagent | Create `~/.deepagents/{agent}/agents/{name}.md` |
