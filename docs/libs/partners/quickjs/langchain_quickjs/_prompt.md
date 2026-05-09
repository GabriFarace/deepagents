# `partners/quickjs/langchain_quickjs/_prompt.py`

> Prompt/rendering helpers for REPL and PTC system prompts.

## Position in the system

This partner package implements or supports a BackendProtocol-compatible sandbox. The core SDK can use these classes through its backend abstraction without depending directly on the provider.

## Imports and module-level state

This file imports `__future__, contextlib, inspect, json, re, typing, pydantic`.
Module constants worth noticing: `_CAMEL_SEP`, `_JS_IDENTIFIER`, `_REPL_SYSTEM_PROMPT_TEMPLATE`.

## Functions and classes

### `render_repl_system_prompt(*, tool_name: str, timeout: float, memory_limit_mb: int, snapshot_between_turns: bool)`

Render the base REPL system prompt text for ``REPLMiddleware``. Key arguments are `tool_name`, `timeout`, `memory_limit_mb`, `snapshot_between_turns`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `format`. It runs synchronously in the caller and returns directly.

### `to_camel_case(name: str)`

Convert ``snake_case`` / ``kebab-case`` → ``camelCase``. Key arguments are `name`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `sub`, `upper`, `group`. It runs synchronously in the caller and returns directly.

### `is_valid_js_identifier(name: str)`

Return whether `name` is a valid JavaScript identifier. Key arguments are `name`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `fullmatch`. It runs synchronously in the caller and returns directly.

### `is_valid_ptc_tool_name(name: str)`

Return whether a tool can be exposed as `tools.<camelCaseName>`. Key arguments are `name`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `is_valid_js_identifier`, `to_camel_case`. It runs synchronously in the caller and returns directly.

### `render_ptc_prompt(tools: Sequence[BaseTool], *, tool_name: str='eval')`

Build the `tools` namespace section of the system prompt. Key arguments are `tools`, `tool_name`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `join`, `to_camel_case`, `append`, `splitlines`, `strip`. It runs synchronously in the caller and returns directly.

## System prompts and tool descriptions

### `_REPL_SYSTEM_PROMPT_TEMPLATE`

```text
### Interpreter

An `{tool_name}` tool is available. It runs JavaScript in a persistent REPL.
{state_persistence_line}
- Top-level `await` works; Promises resolve before the call returns.
- Sandboxed: no filesystem, no stdlib, no network, no real clock, no `fetch`, no `require`.
- Timeout: {timeout}s per call. Memory: {memory_limit_mb} MB total.
- `console.log` output is captured and returned alongside the result.
```

## Gotchas

Most behavior is delegated through framework protocols, so preserve method signatures when editing even if an argument appears unused locally.
