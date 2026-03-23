# `libs/acp/examples/demo_agent.py`

## High-Level Purpose

A runnable demo coding agent that integrates `AgentServerACP` with `create_deep_agent`. Shows how to build a multi-mode, model-switchable ACP agent backed by a `CompositeBackend` (local shell + ephemeral state backend). Demonstrates the full pattern of session context-driven agent factory, mode configuration, and model selection.

## Functions

### `_get_interrupt_config(mode_id: str) -> dict`

**Purpose:** Map a session mode identifier to an interrupt configuration dict.

**Parameters:**
- `mode_id`: One of `"ask_before_edits"`, `"accept_edits"`, or `"accept_everything"`.

**Return Value:** Dict mapping tool names to allowed decisions. Used as the `interrupt_on` parameter for `create_deep_agent`.

**Modes:**
| Mode ID | Interrupt on |
|---------|-------------|
| `ask_before_edits` | `edit_file`, `write_file`, `write_todos`, `execute` (approve/reject) |
| `accept_edits` | `write_todos`, `execute` (approve/reject) |
| `accept_everything` | Nothing (empty dict — auto-accept all) |

---

### `_serve_example_agent() -> None` (async)

**Purpose:** Assemble and start the ACP-served coding agent. Loads `.env`, creates the checkpointer, defines the agent factory and mode configuration, and calls `run_acp_agent`.

**Key Logic:**

1. Creates a `MemorySaver` checkpointer for in-memory thread persistence.

2. Defines `build_agent(context: AgentSessionContext) -> CompiledStateGraph` as an inner factory:
   - Reads `context.cwd` for the root directory.
   - Reads `context.mode` to get the interrupt configuration.
   - Defines a `create_backend()` inner factory that returns a `CompositeBackend`:
     - Default route: `LocalShellBackend(root_dir=_root_dir, inherit_env=True)` for filesystem + shell execution.
     - Routes `"/memories/"` and `"/conversation_history/"` to `StateBackend(tr)` if a `ToolRuntime` is provided (ephemeral, in-memory).
   - Calls `create_deep_agent` with `context.model`, the shared checkpointer, the backend factory, interrupt config, and `LocalContextMiddleware` injecting git/project context into the system prompt.

3. Configures three `SessionMode` objects:
   - `ask_before_edits` — Ask permission before edits, writes, shell commands, and plans
   - `accept_edits` — Auto-accept edit operations; ask before shell commands and plans
   - `accept_everything` — Auto-accept all operations

4. Defines available models for dynamic switching:
   - Anthropic: Claude Opus 4.6, Claude Sonnet 4.5, Claude Haiku 4.5
   - OpenAI: GPT-5.4 Pro, GPT-5.4, GPT-5.3 Codex

5. Creates `AgentServerACP(agent=build_agent, modes=modes, models=models)` and calls `run_acp_agent(acp_agent)`.

---

### `main() -> None`

**Purpose:** Synchronous entry point. Calls `asyncio.run(_serve_example_agent())`.

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| `run_agent` | `acp` | ACP server runner |
| `SessionMode`, `SessionModeState` | `acp.schema` | Mode configuration objects |
| `create_deep_agent` | `deepagents` | Agent factory |
| `CompositeBackend`, `LocalShellBackend`, `StateBackend` | `deepagents.backends` | Backend composition |
| `MemorySaver` | `langgraph.checkpoint.memory` | In-memory checkpointer |
| `AgentServerACP`, `AgentSessionContext` | `deepagents_acp.server` | ACP server class |
| `LocalContextMiddleware` | `examples.local_context` | Git/project context injection |
