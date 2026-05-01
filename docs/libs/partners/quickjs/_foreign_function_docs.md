# `langchain_quickjs/_prompt.py` (formerly `_foreign_function_docs.py`)

> **Note:** This file was renamed and restructured in v0.1.0. The old `_foreign_function_docs.py` generated TypeScript-like stubs for Python foreign functions. The new `_prompt.py` builds the full REPL system prompt, including optional PTC docs and skill module listings.

## High-Level Purpose

Builds the system prompt fragment injected by `REPLMiddleware.modify_request()`. Combines:
- Base REPL usage instructions
- Optional PTC tool documentation (TypeScript-like signatures)
- Optional skill module listing (when `skills_backend` is set)

## Functions

### `build_repl_system_prompt(*, ptc_tools, skill_names, add_ptc_docs) -> str`

Builds the complete system prompt fragment for the REPL tool.

**Parameters:**
- `ptc_tools` — Dict of tool name → `BaseTool` for PTC-enabled tools (empty dict if PTC disabled).
- `skill_names` — List of skill names importable as `@/skills/<name>` (empty list if no skills backend).
- `add_ptc_docs` — If `True`, includes TypeScript-like signatures and descriptions for each PTC tool.

**Returns:** Formatted string to append to the agent's system message.

### `render_ptc_tool_docs(tools: dict[str, BaseTool]) -> str`

Generates TypeScript-style function signatures and docstrings for each PTC tool. Used when `add_ptc_docs=True` to help the model understand the `tools.*` API.

**Example output:**
```typescript
// Available as: tools.readFile(path: string): Promise<string>
// Reads a file from the backend.
tools.readFile: (path: string) => Promise<string>
```

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| `BaseTool` | `langchain_core.tools` | Tool type inspection |
| `inspect` | stdlib | Signature introspection |
