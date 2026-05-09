# `partners/quickjs/langchain_quickjs/_format.py`

> Formatting and output-coercion helpers for the QuickJS REPL.

## Position in the system

This partner package implements or supports a BackendProtocol-compatible sandbox. The core SDK can use these classes through its backend abstraction without depending directly on the provider.

## Imports and module-level state

This file imports `__future__, json, typing, langchain_core.messages, langgraph.types, pydantic, quickjs_rs`.
Module constants worth noticing: `_TRUNCATE_MARKER`, `_NATIVE_JS_SCALARS`.

## Functions and classes

### `format_handle(handle: Any)`

Describe a ``Handle`` value in REPL-style shorthand. Key arguments are `handle`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `get`, `to_python`, `dispose`. It runs synchronously in the caller and returns directly.

### `stringify(value: Any)`

Best-effort string form for a console arg or eval result. Key arguments are `value`. It does not keep durable module state; effects come from returned values or delegated calls. It runs synchronously in the caller and returns directly.

### `coerce_tool_output(value: Any)`

Coerce arbitrary tool return values to the JS-visible string output. Key arguments are `value`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `isinstance`, `reversed`. It runs synchronously in the caller and returns directly.

### `coerce_tool_output_for_ptc(value: Any)`

Coerce a tool result for the PTC bridge, preserving native types. The quickjs_rs ``register`` bridge marshals Python primitives, ``list``, and ``dict`` directly to native JS values, so the model can use them without an explicit ``JSON.parse``. This helper unwraps LangChain's ``ToolMessage`` / ``Command`` envelopes (matching ``coerce_tool_output``'s selection rules) and returns the underlying value typed. Compound returns are walked recursively: nested values that the binding cannot marshal natively (``datetime``, Pydantic models, custom classes) are stringified in place via ``str(value)`` so the surrounding object structure remains navigable from JS. Cyclic structures hit Python's recursion limit and surface as a host error in the eval — same outcome as ``json.dumps`` on a self-referencing dict. Key arguments are `value`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `isinstance`, `coerce_tool_output_for_ptc`, `reversed`. It runs synchronously in the caller and returns directly.

### `format_outcome(outcome: EvalOutcome, *, max_result_chars: int)`

Render an EvalOutcome-like object as the tool's wire format. Key arguments are `outcome`, `max_result_chars`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `join`, `append`. It runs synchronously in the caller and returns directly.

## Gotchas

Most behavior is delegated through framework protocols, so preserve method signatures when editing even if an argument appears unused locally.
