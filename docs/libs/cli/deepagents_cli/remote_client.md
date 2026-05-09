# `libs/cli/deepagents_cli/remote_client.py`

> Thin wrapper around LangGraph `RemoteGraph` for HTTP/SSE graph access.

## Position in the system

The UI and non-interactive runner talk to `RemoteAgent`, which talks to the
LangGraph server. This file is the wire-format boundary.

## Functions and classes

### `_require_thread_id(config)`

Extracts `config.configurable.thread_id` and raises if missing. Every CLI run
is attached to a LangGraph thread.

### `RemoteAgent`

Lazily constructs `RemoteGraph`, streams graph execution, normalizes update
interrupts, converts streamed message dictionaries into LangChain message
objects, and exposes state get/update helpers.

### Conversion helpers

`_prepare_config()`, `_convert_interrupts()`, `_convert_ai_message()`,
`_convert_human_message()`, `_convert_tool_message()`, and
`_convert_message_data()` adapt serialized server data to the shapes expected
by CLI rendering code.

## Gotchas

Streamed messages are converted for rendering; full state snapshots may remain
server-serialized.
