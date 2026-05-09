# `libs/cli/deepagents_cli/_cli_context.py`

> Lightweight runtime context type for CLI model overrides.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Functions and classes

### `CLIContext`

Runtime context passed via `context=` to the LangGraph graph.

Additional notes from the source docstring:

```text
Carries per-invocation overrides that `ConfigurableModelMiddleware`
reads from `request.runtime.context`.
```

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
