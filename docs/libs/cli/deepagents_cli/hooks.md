# `libs/cli/deepagents_cli/hooks.py`

> External lifecycle hook loading and dispatch.

## Position in the system

Hooks let users run local commands around CLI events without modifying the CLI.

## Functions and classes

### `_load_hooks()`

Reads configured hook definitions and returns normalized dictionaries.

### `_run_single_hook(command, event, payload_bytes)`

Runs one hook subprocess with serialized payload on stdin.

### `_dispatch_hook_sync(...)`

Runs matching hooks for an event.

### `dispatch_hook(...)` and `dispatch_hook_fire_and_forget(...)`

Async and non-blocking public dispatch surfaces.

## Gotchas

Hook failures should be visible but should not casually crash the TUI.
