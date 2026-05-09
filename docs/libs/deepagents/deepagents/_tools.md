# `libs/deepagents/deepagents/_tools.py`

> Helpers for inspecting caller-provided tools and applying model-profile tool
> description overrides without mutating user objects.

## Position in the system

Harness profiles can override tool descriptions for particular models. The
factory applies those overrides to top-level user tools and to declarative
subagent tools before passing them into LangChain's agent builder.

```
HarnessProfile.tool_description_overrides
  │
  ▼
_apply_tool_description_overrides(...)
  ├─ top-level tools in create_deep_agent()
  └─ inherited or subagent-specific tools
```

Tool exclusion is handled elsewhere by `_ToolExclusionMiddleware`; this file
only rewrites descriptions.

## Imports and module-level state

At runtime the file imports only `BaseTool`, `Any`, and `cast`. The callable,
mapping, and sequence types are imported under `TYPE_CHECKING`, so they do not
affect runtime import costs.

## Functions and classes

### `_tool_name(tool)`

`_tool_name()` extracts a name from any tool shape accepted by
`create_deep_agent()`: LangChain `BaseTool` instances, plain callables with a
`.name` attribute, and provider/tool dictionaries with a `"name"` key. It
returns `None` if the name is unavailable or not a string.

The helper is deliberately conservative. Description overrides are keyed by
tool name; applying an override when the name is ambiguous would rewrite the
wrong tool. Returning `None` lets the caller leave that tool untouched.

### `_apply_tool_description_overrides(tools, overrides)`

This function copies the caller's tool sequence into a new list and applies
description overrides where it can do so safely. Dict tools are shallow-copied
and their `"description"` key is replaced. `BaseTool` instances are copied via
Pydantic's `model_copy(update={...})`. Plain callables are returned unchanged
because changing their description would require wrapping them in a new tool,
which could alter identity, decorators, or LangChain's schema inference.

If `tools` is `None`, the function returns `None` so the downstream call to
`create_agent()` can preserve the same distinction between "no additional
tools" and "an empty tool list". Tools whose names do not appear in
`overrides` are reused by reference.

## Flow walk-through

1. `graph.py:496` rewrites top-level tools with the main harness profile's
   overrides.
2. `graph.py:577` picks a declarative subagent's own tools or inherited parent
   tools.
3. `graph.py:578` rewrites those subagent tools with that subagent model's
   profile overrides.
4. Later `_ToolExclusionMiddleware` can filter tools, including middleware
   injected tools; this helper does not remove anything.

## Gotchas

Callables are intentionally not rewritten. If a profile needs a different
description for a callable tool, expose it as a `BaseTool` or dict-shaped tool
before passing it to `create_deep_agent()`.
