# `repl/langchain_repl/_foreign_function_docs.py`

> Render compact prompt-facing documentation for QuickJS foreign functions.

## Position in the system

This package provides an optional REPL tool middleware for LangChain/deepagents. It is independent of the main SDK and can be mounted as middleware when an agent needs compact programmable tool orchestration.

## Imports and module-level state

This file imports `__future__, contextlib, inspect, typing, langchain_core.tools`.
Module constants worth noticing: `_ELLIPSIS_TUPLE_ARG_COUNT`.

## Functions and classes

### `get_ptc_implementations(ptc: list[Callable[..., Any] | BaseTool] | None)`

Return configured PTC implementations keyed by exported function name. Key arguments are `ptc`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `isinstance`, `getattr`. It runs synchronously in the caller and returns directly.

### `render_external_functions_section(implementations: dict[str, Callable[..., Any] | BaseTool], *, add_docs: bool)`

Build the optional prompt section describing foreign functions. Key arguments are `implementations`, `add_docs`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `join`, `render_foreign_function_section`. It runs synchronously in the caller and returns directly.

### `render_foreign_function_section(implementations: dict[str, Callable[..., Any] | BaseTool])`

Render the complete prompt section for available foreign functions. Key arguments are `implementations`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `join`, `extend`, `items`. It runs synchronously in the caller and returns directly.

### `format_foreign_function_docs(name: str, implementation: Callable[..., Any] | BaseTool)`

Render a compact signature and docstring block for a foreign function. Key arguments are `name`, `implementation`. It does not keep durable module state; effects come from returned values or delegated calls. It runs synchronously in the caller and returns directly.

## Gotchas

Most behavior is delegated through framework protocols, so preserve method signatures when editing even if an argument appears unused locally.
