# `partners/quickjs/langchain_quickjs/_ptc.py`

> Programmatic tool calling (PTC) support for ``REPLMiddleware``.

## Position in the system

This partner package implements or supports a BackendProtocol-compatible sandbox. The core SDK can use these classes through its backend abstraction without depending directly on the provider.

## Imports and module-level state

This file imports `__future__, typing, langchain_core.tools, langchain_quickjs`.

## Functions and classes

### `filter_tools_for_ptc(tools: Sequence[BaseTool], config: PTCOption, *, self_tool_name: str)`

Return the subset of ``tools`` exposed inside the REPL. ``self_tool_name`` is the REPL's own tool name; it is *always* excluded to prevent the model from recursing ``tools.eval("tools.eval(...)")``. If the model wants a nested eval, it can just write nested code in one call — that's the whole point of PTC. ``config`` is allowlist-only: - ``str`` entries: expose matching tool names from ``tools``. - ``BaseTool`` entries: expose those tools directly (minus ``self_tool_name``). Mixed lists are supported and merged. Explicit ``BaseTool`` entries are included first, then name-matched agent tools are appended. Duplicate tool names are deduplicated. Warning: PTC tool calls execute through the REPL bridge and currently do not respect `interrupt_on` / HITL approval hooks for each individual tool invocation. Key arguments are `tools`, `config`, `self_tool_name`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `isinstance`, `TypeError`, `set`, `add`, `append`. It runs synchronously in the caller and returns directly.

### `to_camel_case(name: str)`

Convert ``snake_case`` / ``kebab-case`` → ``camelCase``. Key arguments are `name`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `to_camel_case`. It runs synchronously in the caller and returns directly.

### `is_valid_js_identifier(name: str)`

Return whether `name` is a valid JavaScript identifier. Key arguments are `name`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `is_valid_js_identifier`. It runs synchronously in the caller and returns directly.

### `is_valid_ptc_tool_name(name: str)`

Return whether a tool can be exposed as `tools.<camelCaseName>`. Key arguments are `name`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `is_valid_ptc_tool_name`. It runs synchronously in the caller and returns directly.

### `render_ptc_prompt(tools: Sequence[BaseTool], *, tool_name: str='eval')`

Build the `tools` namespace section of the system prompt. Key arguments are `tools`, `tool_name`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `render_ptc_prompt`. It runs synchronously in the caller and returns directly.

## Gotchas

Most behavior is delegated through framework protocols, so preserve method signatures when editing even if an argument appears unused locally.
