# `libs/cli/deepagents_cli/agent.py`

> CLI-specific agent factory and agent-directory utilities.

## Position in the system

`server_graph.py` calls `create_cli_agent()` after resolving model, tools, MCP,
and sandbox configuration. This file adapts the SDK's generic
`create_deep_agent()` to CLI behavior.

## Functions and classes

### `ShellAllowListMiddleware`

Rejects disallowed shell commands in non-interactive mode by returning an
error `ToolMessage` before execution. This avoids interrupt/resume cycles in
automation while preserving shell safety.

### Agent config helpers

`load_async_subagents()`, `_is_agent_dir_entry()`,
`get_available_agent_names()`, `list_agents()`, and `reset_agent()` manage
configured agent directories and remote async subagent definitions.

### Prompt and tool-description helpers

`build_model_identity_section()`, `get_system_prompt()`, and the
`_format_*_description()` helpers build CLI-specific prompt/tool text for
coding, web, task, file, and execute behavior.

### `_add_interrupt_on()`

Builds the SDK HITL interrupt map for risky tools.

### `create_cli_agent(...)`

Chooses backend, prompt, middleware, shell policy, memory/skills, async
subagents, MCP tools, and approval behavior, then calls `create_deep_agent()`.

## Gotchas

Interactive and non-interactive shell safety differ: interactive runs can use
LangGraph interrupts, while non-interactive runs need deterministic allow-list
rejection.
