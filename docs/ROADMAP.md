# `deepagents` — Learning Roadmap

This is a **learning path**, not a table of contents. Each stage builds on
the previous one. Read the docs (high-level) first; dive into the source code
(deep dive) afterward as needed.

> **Time estimates** in the table are for reading the docs only. Add 2–3×
> for actually exploring the source.

---

## How to use this roadmap

- **Each stage is a self-contained learning unit.** You should be able to
  pause at the end of any stage and come back later.
- Links labelled **read** point to docs in this tree; **source** points to
  source code under `libs/`.
- A stage's **prerequisites** are stages you must complete first. There are
  no forward dependencies — every stage is self-sufficient given its
  prerequisites.
- The roadmap is intentionally **CLI-weighted**: stages 8–11 spend more time
  on the CLI than on the SDK because that's where the user is most curious.

---

## Overview

| # | Stage | Time | Prereqs |
|---|---|---|---|
| 1 | Orientation | 30 min | — |
| 2 | Prerequisites: LangChain | 60 min | 1 |
| 3 | Prerequisites: LangGraph | 60 min | 2 |
| 4 | The deep-agent flow | 45 min | 3 |
| 5 | Backends | 45 min | 4 |
| 6 | Middleware | 90 min | 4 |
| 7 | Profiles & models | 30 min | 4 |
| 8 | Generic CLI architecture | 30 min | 4 |
| 9 | CLI bootstrap & server lifecycle | 60 min | 8 |
| 10 | CLI runtime: TUI, streaming, widgets | 90 min | 9 |
| 11 | CLI extensions: slash commands, hooks, sessions, MCP, skills, deploy | 90 min | 10 |
| 12 | ACP adapter | 30 min | 4 |
| 13 | Sandboxes (partners) | 30 min | 5 |
| 14 | Examples & evals | 45 min | 6, 11 |

Total: ~12 hours of reading; ~30–35 hours including source exploration.

---

## Stage 1 — Orientation (30 min)

**Goal.** Know what `deepagents` is, what shape the repo has, and how the
packages depend on each other.

| Read | Source | What you learn |
|---|---|---|
| `../README.md` | — | Project pitch + 5-minute quickstart |
| `../CLAUDE.md` | — | Repo map + dependency graph + key concepts (high level) |
| `../AGENTS.md` | — | Development conventions |
| `./README.md` | — | This docs tree's conventions |

**Key insight.** `deepagents` is a thin opinionated shell over LangChain's
`create_agent()` plus a stack of `AgentMiddleware` subclasses. Almost every
"feature" (filesystem tools, sub-agents, planning, HITL) is a middleware.

---

## Stage 2 — Prerequisites: LangChain (60 min)

**Goal.** Understand `ChatModel`, messages, `@tool`, `bind_tools`,
`create_agent`, and **`AgentMiddleware`** — the system every deepagents
middleware extends.

| Read | What you learn |
|---|---|
| `prerequisites/langchain.md` | The full LangChain surface area deepagents uses |

**Key concepts.** Read §5 (Middleware) carefully — the rest of the
documentation is unintelligible without it. Also internalise: the agent
loop is `model → tools → model → ...` until `tool_calls` is empty;
`wrap_model_call` middlewares nest like Express middleware (first wraps
all others); jump targets are `"end"`, `"model"`, `"tools"`.

---

## Stage 3 — Prerequisites: LangGraph (60 min)

**Goal.** Understand `StateGraph`, `MessagesState`, `add_messages`, the
agent loop's graph form, checkpointers + threads, the seven streaming modes,
interrupts (HITL foundation), and subgraphs (sub-agent foundation).

| Read | What you learn |
|---|---|
| `prerequisites/langgraph.md` | The full LangGraph surface area deepagents uses |

**Key concepts.**

1. State + reducers + nodes + edges + super-steps = Pregel.
2. `messages` channel uses `add_messages` (append + replace-by-id +
   deserialise).
3. Threads = checkpoint sequences; `thread_id` is the persistent cursor.
4. `version="v2"` streaming gives a uniform `StreamPart` shape.
5. `interrupt(...)` + `Command(resume=...)` = HITL.
6. Subgraphs run with isolated state; results bubble up through reducers.

---

## Stage 4 — The deep-agent flow (45 min)

**Goal.** See how `create_deep_agent()` assembles a working agent: model
resolution, default tools, default middleware stack, system-prompt
construction, and how state flows through it.

| Read | Source | What you learn |
|---|---|---|
| `libs/deepagents/README.md` | `libs/deepagents/` | Package entry points |
| `libs/deepagents/deepagents/README.md` | `libs/deepagents/deepagents/` | Module map |
| `libs/deepagents/deepagents/graph.md` | `graph.py` | The factory: parameters, defaults, middleware ordering, the system prompt assembly |
| `libs/deepagents/deepagents/_models.md` | `_models.py` | How model strings resolve to `ChatModel` instances |
| `libs/deepagents/deepagents/_tools.md` | `_tools.py` | The default toolset (todo list, etc.) |

**Key insight.** `create_deep_agent()` is `create_agent()` with carefully
chosen middleware in a specific order plus a backend abstraction. There is
no magic — you can replicate the result by hand if you understand each
middleware.

---

## Stage 5 — Backends (45 min)

**Goal.** Understand how every file/shell tool dispatches through
`BackendProtocol` and the trade-offs between the eight implementations.

| Read | Source | What you learn |
|---|---|---|
| `libs/deepagents/deepagents/backends/README.md` | `backends/` | Selection guide + protocol overview |
| `libs/deepagents/deepagents/backends/protocol.md` | `protocol.py` | The contract (read/write/edit/ls/grep/glob/execute + result TypedDicts) |
| `libs/deepagents/deepagents/backends/state.md` | `state.py` | Default ephemeral backend (files live in graph state) |
| `libs/deepagents/deepagents/backends/filesystem.md` | `filesystem.py` | Direct disk access |
| `libs/deepagents/deepagents/backends/local_shell.md` | `local_shell.py` | Disk + local shell (CLI default) |
| `libs/deepagents/deepagents/backends/sandbox.md` | `sandbox.py` | The base class for partner sandboxes |
| `libs/deepagents/deepagents/backends/store.md` | `store.py` | Persistent cross-thread (LangGraph `BaseStore`) |
| `libs/deepagents/deepagents/backends/composite.md` | `composite.py` | Path-prefix routing |
| `libs/deepagents/deepagents/backends/langsmith.md` | `langsmith.py` | LangSmith integration |
| `libs/deepagents/deepagents/backends/utils.md` | `utils.py` | Shared helpers |

**Key insight.** Tools never touch the filesystem directly. Every tool
calls a `BackendProtocol` method. Swapping backends swaps the entire
storage layer transparently.

---

## Stage 6 — Middleware (90 min)

**Goal.** Master the deepagents middleware stack. Each middleware is a
distinct concept; together they define the agent's behaviour.

| Read | Source | What you learn |
|---|---|---|
| `libs/deepagents/deepagents/middleware/README.md` | `middleware/` | The full stack, ordering, and rationale |
| `…/subagents.md` | `subagents.py` | The `task` tool (synchronous sub-agent dispatch); subagent definition shape; how the parent state interacts with the child graph |
| `…/async_subagents.md` | `async_subagents.py` | Background sub-agent spawning |
| `…/filesystem.md` | `filesystem.py` | All file/shell tools (read/write/edit/glob/grep/ls/execute); their system prompt addendum |
| `…/skills.md` | `skills.py` | The skill catalog system; how skills are loaded and exposed |
| `…/memory.md` | `memory.py` | `AGENTS.md` discovery and injection |
| `…/summarization.md` | `summarization.py` | Context compaction triggers and prompt |
| `…/permissions.md` | `permissions.py` | HITL approval gates (built on LangGraph interrupts) |
| `…/patch_tool_calls.md` | `patch_tool_calls.py` | Repair of dangling/malformed tool calls |

**Key concepts.** Stack order matters: filesystem tools must register
before sub-agent middleware sees them; permissions must wrap them last;
summarisation must run after token-counting. Read `__init__.py` for the
canonical default-stack assembly.

---

## Stage 7 — Profiles & models (30 min)

**Goal.** Understand per-model adjustments: how the same agent behaves
differently when you swap `claude-sonnet-4-6` for `claude-opus-4-7` or
`gpt-5-codex`.

| Read | Source | What you learn |
|---|---|---|
| `libs/deepagents/deepagents/profiles/README.md` | `profiles/` | Profile dispatch and key constants |
| `…/profiles/harness/` | `profiles/harness/*.py` | Per-model tweaks (system prompt, prompt caching, tool descriptions) |
| `…/profiles/provider/` | `profiles/provider/*.py` | Per-provider tweaks (OpenAI vs OpenRouter quirks) |

**Key insight.** A profile is a bundle of small middleware overrides
applied conditionally based on the chosen model id.

---

## Stage 8 — Generic CLI architecture (30 min)

**Goal.** Develop the conceptual map of an agent CLI before diving into
deepagents-cli specifics. Understand the trade-offs each design decision
encodes.

| Read | What you learn |
|---|---|
| `libs/cli/cli_architecture.md` | The shell of an agent CLI; single-process vs server-backed; TUI frameworks; streaming; slash commands; tool rendering & approvals; MCP; sessions; hooks. Comparisons to Claude Code and Codex. |

**Key insight.** deepagents-cli is **server-backed** (it spawns
`langgraph dev` as a subprocess), while Claude Code and Codex are
**single-process**. This buys you persistence, hot reload, ACP mode, and
remote sub-agents at the cost of startup latency.

---

## Stage 9 — CLI bootstrap & server lifecycle (60 min)

**Goal.** Trace what happens from `$ deepagents` to a running TUI talking
to a `langgraph dev` subprocess.

| Read | Source | What you learn |
|---|---|---|
| `libs/cli/README.md` | `libs/cli/` | Package shape |
| `libs/cli/deepagents_cli/README.md` | `deepagents_cli/` | Module map |
| `…/main.md` | `main.py` | CLI entry, arg parsing, environment bootstrap |
| `…/server.md` | `server.py` | The langgraph subprocess controller |
| `…/server_manager.md` | `server_manager.py` | Lifecycle: spawn, health-check, shutdown |
| `…/server_graph.md` | `server_graph.py` | The graph the spawned server runs |
| `…/agent.md` | `agent.py` | The agent factory used by the server graph |
| `…/config.md`, `…/model_config.md`, `…/configurable_model.md` | `config.py`, `model_config.py`, `configurable_model.py` | Config loading & model selection |
| `…/non_interactive.md` | `non_interactive.py` | The `-p` non-interactive mode |

**Key insight.** The CLI never imports the SDK directly. It writes a
`langgraph.json` to a temp dir, spawns `langgraph dev`, then talks to the
server via the `langgraph-sdk` HTTP client.

---

## Stage 10 — CLI runtime: TUI, streaming, widgets (90 min)

**Goal.** Understand how the TUI is built, how it consumes the LangGraph
SSE stream, and how each widget renders messages, tool calls, approval
prompts, and notifications.

| Read | Source | What you learn |
|---|---|---|
| `…/app.md` | `app.py` | The Textual `App` subclass; layout; bindings; lifecycle |
| `…/remote_client.md` | `remote_client.py` | HTTP/SSE client to the LG server; backpressure; cancellation |
| `…/event_bus.md` | `event_bus.py` | Internal pub/sub between widgets |
| `…/input.md` | `input.py` | User-input dispatch |
| `…/widgets/README.md` | `widgets/` | Widget catalog |
| `…/widgets/messages.md` + `message_store.md` | `widgets/messages.py`, `message_store.py` | Message rendering pipeline |
| `…/widgets/chat_input.md` | `widgets/chat_input.py` | Multiline input + autocomplete |
| `…/widgets/approval.md` + `…/widgets/ask_user.md` | `widgets/approval.py`, `widgets/ask_user.py` | HITL approval flow |
| `…/widgets/tool_widgets.md` + `tool_renderers.md` + `diff.md` | `widgets/tool_widgets.py`, etc. | Per-tool rendering (read, edit, bash, search) |
| `…/widgets/notification_*.md` | `widgets/notification_*.py` | Notification center |
| `…/tool_display.md` + `formatting.md` | `tool_display.py`, `formatting.py` | Cross-cutting rendering helpers |
| `…/theme.md` | `theme.py` | Theming system |

**Key insight.** Streaming is end-to-end SSE: the LG server streams
`{type, ns, data}` parts; `remote_client.py` decodes them; the event bus
routes them to widgets. The approval widget *blocks* the current request
until the user resolves it (because the underlying LangGraph interrupt is
blocked too).

---

## Stage 11 — CLI extensions (90 min)

**Goal.** Cover everything around the runtime — slash commands, hooks,
sessions, MCP, skills, deploy.

| Read | Source | What you learn |
|---|---|---|
| `…/command_registry.md` | `command_registry.py` | Slash-command definition + dispatch |
| `…/hooks.md` | `hooks.py` | User-extensible lifecycle hooks (pre-tool, post-tool, etc.) |
| `…/sessions.md` | `sessions.py` | On-disk session metadata; thread resume |
| `…/state_migration.md` | `state_migration.py` | State-shape migrations across versions |
| `…/mcp_tools.md` + `…/mcp_auth.md` + `…/mcp_commands.md` + `…/mcp_trust.md` | `mcp_*.py` | MCP server discovery, auth, trust, slash-commands |
| `…/mcp_providers/README.md` | `mcp_providers/` | Built-in MCP providers (github, slack) |
| `…/skills/README.md` | `skills/` | Skill load, invocation, slash-commands |
| `…/built_in_skills/README.md` | `built_in_skills/` | The shipped skill-creator |
| `…/deploy/README.md` | `deploy/` | The `deepagents deploy` bundling pipeline |
| `…/integrations/README.md` | `integrations/` | Sandbox factory hooks |
| `…/onboarding.md` | `onboarding.py` | First-run onboarding |
| `…/auth_store.md` | `auth_store.py` | Credential storage |
| `…/notifications.md` | `notifications.py` | Notification dispatch |
| `…/update_check.md` | `update_check.py` | Self-update prompts |

**Key insight.** Each subsystem here is a small, self-contained module.
The only place they interact is via the event bus and the slash-command
registry.

---

## Stage 12 — ACP adapter (30 min)

**Goal.** Understand how to expose a deepagents agent over the ACP wire
protocol.

| Read | Source | What you learn |
|---|---|---|
| `libs/acp/README.md` | `libs/acp/deepagents_acp/` | What ACP is, what's bridged |
| `libs/acp/deepagents_acp/server.md` | `server.py` | The protocol bridge |
| `libs/acp/deepagents_acp/utils.md` | `utils.py` | Shared helpers |

**Key insight.** ACP is wire-protocol-only — the underlying agent is
the same `create_deep_agent()` you've already studied.

---

## Stage 13 — Sandboxes (partners) (30 min)

**Goal.** Survey the four sandbox adapters and understand the trade-offs:
in-process JS (`quickjs`) vs cloud VM (`daytona`, `runloop`) vs serverless
container (`modal`).

| Read | Source |
|---|---|
| `libs/partners/README.md` | `libs/partners/` |
| `libs/partners/daytona/README.md` | `libs/partners/daytona/` |
| `libs/partners/modal/README.md` | `libs/partners/modal/` |
| `libs/partners/quickjs/README.md` | `libs/partners/quickjs/` |
| `libs/partners/runloop/README.md` | `libs/partners/runloop/` |

**Key insight.** Each adapter implements `BackendProtocol` (specifically
the sandbox-execution methods) so the rest of the system doesn't care
which is in use.

---

## Stage 14 — Examples & evals (45 min)

**Goal.** Wrap up by skimming the example agents and the evaluation
harness.

| Read | Source |
|---|---|
| `examples/README.md` | `examples/` |
| (one short doc per example agent) | `examples/*/` |
| `libs/evals/README.md` | `libs/evals/` |
| `libs/evals/deepagents_evals/cli.md` | `deepagents_evals/cli.py` |
| `libs/evals/deepagents_harbor/README.md` | `deepagents_harbor/` |

**Key insight.** Each example is a "lite" version of one part of the
system — `deep_research` is the canonical multi-stage agent;
`text-to-sql-agent` shows backend swapping; `repl_swarm` shows REPL-driven
sub-agents; `nvidia_deep_agent` shows provider-specific profiles. Pick
one or two that match what you want to build.

---

## After the roadmap

You will have:

- A working mental model of the agent loop, the middleware stack, and the
  CLI's server-backed architecture.
- The ability to add a new middleware, a new tool, a new backend, a new
  slash command, or a new MCP integration.
- Enough context to read CI/CD, infra, and release tooling on your own
  when those are documented in a future pass.
