# `libs/acp` — deepagents-acp

## What This Package Does

`deepagents-acp` is the Agent Client Protocol (ACP) adapter for the Deep Agents framework. It exposes a Deep Agent (a LangGraph `CompiledStateGraph`) as an ACP-compliant server that a front-end client (such as the Deep Agents CLI or any ACP-compatible UI) can connect to.

The package bridges two worlds:
- **ACP** (the wire protocol): sessions, prompts, tool call display, HITL permission requests, plan updates, mode/model configuration.
- **Deep Agents / LangGraph** (the execution engine): streaming graphs, checkpointed state, tool execution, interrupts.

## Directory Layout

```
libs/acp/
├── deepagents_acp/
│   ├── __init__.py         # Package marker
│   ├── __main__.py         # python -m deepagents_acp entry point
│   ├── server.py           # AgentServerACP — the core ACP adapter
│   └── utils.py            # Content block converters + shell command parser
├── tests/
│   ├── chat_model.py       # GenericFakeChatModel for testing
│   ├── test_agent.py       # Integration tests for AgentServerACP
│   ├── test_command_allowlist.py  # Tests for command-type allowlist
│   ├── test_main.py        # Smoke test for __main__
│   ├── test_model_switching.py    # Tests for model switching
│   └── test_utils.py       # Unit tests for utils.py
├── pyproject.toml          # Package metadata and dependencies
└── Makefile                # Development workflow targets
```

## How Files Relate

- `server.py` is the heart of the package. `AgentServerACP` implements every ACP protocol method and imports utilities from `utils.py`.
- `utils.py` is a pure-function support module: no state, no classes. It converts ACP content blocks to LangChain dicts and parses shell command strings for the security allowlist.
- `__main__.py` is a thin entry point that calls `_serve_test_agent()` from `server.py`.
- `tests/chat_model.py` provides `GenericFakeChatModel` used across all test files.
- `tests/test_agent.py` covers `AgentServerACP` end-to-end.
- `tests/test_command_allowlist.py` covers both `extract_command_types()` and the allowlist logic.
- `tests/test_model_switching.py` covers the `models` config option and `set_config_option`.
- `tests/test_utils.py` covers individual conversion functions in isolation.
- `tests/test_main.py` is a minimal import smoke test.

## Key Concepts

### Session Lifecycle
1. Client calls `new_session(cwd=...)` → server generates a UUID session ID and initializes mode/model state.
2. Client calls `prompt([...blocks...], session_id=...)` → server streams the agent graph, forwarding updates.
3. Client may call `cancel(session_id)` to abort mid-stream.
4. Mode or model changes via `set_config_option` / `set_session_mode` trigger `_reset_agent()`.

### Human-in-the-Loop (HITL)
When the LangGraph graph emits a `__interrupt__` update, `AgentServerACP._handle_interrupts()` pauses streaming and calls `client.request_permission()`. The client responds with `approve`, `reject`, or `approve_always`. `approve_always` adds the command signature to `_allowed_command_types[session_id]` for future auto-approval.

### Command Allowlist
`extract_command_types()` normalizes shell commands into security-relevant signatures (e.g., `"python -m pytest"`, `"npm install"`). This prevents `approve_always` from unintentionally approving a broader class of commands than the user intended.

### Plan Display
When the agent calls `write_todos`, `AgentServerACP._handle_todo_update()` sends an `AgentPlanUpdate` to the client, which the UI can render as a task list. When all tasks complete (or the plan is rejected), `_clear_plan()` sends an empty update.
