# `langchain_quickjs/_foreign_function_docs.py`

## High-Level Purpose

Renders compact prompt documentation for QuickJS foreign functions. Converts Python function signatures, type annotations, and docstrings into TypeScript-like stubs and JSDoc blocks that the LLM can use to understand how to call foreign functions from within the JavaScript REPL.

## Public API

### `render_external_functions_section(implementations: dict, *, add_docs: bool) -> str`

**Purpose:** Build the optional system prompt section describing available foreign functions.

**Parameters:**
- `implementations`: Name-keyed dict of callables or `BaseTool` objects.
- `add_docs`: If `False`, returns a simple bullet list of function names. If `True`, includes full TypeScript-like signatures and JSDoc comments.

**Return Value:** Empty string if no implementations. Otherwise a formatted section starting with `"\n\nAvailable foreign functions:\n..."`.

---

### `render_foreign_function_section(implementations: dict) -> str`

**Purpose:** Render the full prompt section with TypeScript-style signatures and referenced types.

**Return Value:** A markdown code block containing all function stubs, plus a "Referenced types" block if any return types are structured TypedDicts.

---

### `format_foreign_function_docs(name: str, implementation) -> str`

**Purpose:** Render a single function stub (signature + JSDoc) for one foreign function.

---

## Internal Type Annotation Rendering

The module contains a hierarchy of annotation renderers:

### `_format_annotation(annotation: Any) -> str`
Dispatches to specialized formatters based on the annotation type:
- Primitives (`str`, `bool`, `int`, `float`, `None`, `Any`) → TypeScript equivalents
- Collections (`list`, `set`, `tuple`) → `T[]` or tuple syntax
- `dict` → `Record<K, V>`
- `Union` / `UnionType` → `A | B`
- Unknown → `str(annotation)` with cleanup

### `_render_function_stub(name, implementation) -> str`
Produces a TypeScript-like function declaration, optionally prefixed with a JSDoc block derived from the Python docstring. Handles both `async function` and `function` prefixes based on whether the implementation is a coroutine.

### `_render_typed_dict_definition(annotation) -> str`
Renders a TypedDict as a TypeScript `type Foo = { field: T }` block.

### `_collect_referenced_types(implementations) -> list[type]`
Identifies TypedDict-like return types from foreign function signatures, collecting unique types for the "Referenced types" section.

### `_render_jsdoc(doc: str) -> str`
Converts a Google-style Python docstring into a JSDoc `/** ... */` block, extracting the summary lines and `@param` entries from the `Args:` section.

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| `inspect` | stdlib | Signature and docstring introspection |
| `typing` | stdlib | `get_type_hints`, `get_origin`, `get_args` |
| `BaseTool` | `langchain_core.tools` | LangChain tool type |
