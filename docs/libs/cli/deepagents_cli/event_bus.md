# `libs/cli/deepagents_cli/event_bus.py`

> External event ingress for the Textual CLI app.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from deepagents_cli.command_registry import BypassTier`


## Functions and classes

### `ExternalEvent`

A transport-independent event delivered from outside the TUI.

Methods worth reading inside this class:

- `__post_init__(self)`: Validate invariants for direct construction.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `EventSource`

Source of external events for the CLI app.

Additional notes from the source docstring:

```text
Implementations must be safe to `stop()` even when `start()` failed
partway through; the app always invokes `stop()` from a `finally` block.
```

Methods worth reading inside this class:

- `start(self, sink: Callable[[ExternalEvent], Awaitable[None]])`: Start forwarding events to `sink`.

- `serve_forever(self)`: Park until the source is cancelled or its transport dies.

- `stop(self)`: Stop forwarding events and release transport resources.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `UnixSocketEventSource`

Line-delimited JSON event source over a local Unix domain socket.

Additional notes from the source docstring:

```text
The listener creates its parent directory with mode `0o700` and binds the
socket inside it under a transient `umask(0o077)` so the socket inherits
`0o600` from the moment of `bind()`. Stale sockets at the configured path
are removed on start, but only after a `stat` confirms the path is a
socket — a regular file or directory at that path is left untouched and
causes start to fail loudly.
```

Methods worth reading inside this class:

- `start(self, sink: Callable[[ExternalEvent], Awaitable[None]])`: Start listening for newline-delimited JSON events.

- `serve_forever(self)`: Park until the underlying server is cancelled or fails.

- `stop(self)`: Close the listener and remove the socket path.

- `_handle_client(self, reader: asyncio.StreamReader, writer: asyncio.StreamWriter)`: Read newline-delimited JSON envelopes from one client.

- `_handle_one_line(self, line: bytes, writer: asyncio.StreamWriter)`: Decode and dispatch one envelope, replying with ACK or NACK.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `_write_ack(writer: asyncio.StreamWriter, correlation_id: str | None)`

Write the success acknowledgement frame, echoing `correlation_id`.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_write_nack(writer: asyncio.StreamWriter, error: str, correlation_id: str | None)`

Write a failure response frame; never raises on a closed socket.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `default_unix_socket_path()`

Return the default per-process Unix socket path.

Additional notes from the source docstring:

```text
Prefers `XDG_RUNTIME_DIR` (per-user, tmpfs-backed, auto-cleaned on
logout) and falls back to the system temp dir when the runtime
directory is unset.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_unlink_existing_socket(path: Path)`

Remove a stale Unix socket without touching other filesystem entries.

Additional notes from the source docstring:

```text
Args:
    path: Candidate socket path to remove.

Raises:
    FileNotFoundError: If `path` does not exist.
    FileExistsError: If `path` exists but is not a Unix socket.
    OSError: If the entry exists but cannot be removed.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `decode_external_event(data: bytes, *, source: str)`

Decode one newline-delimited JSON external event.

Additional notes from the source docstring:

```text
Args:
    data: Raw JSON line.
    source: Transport-specific source label attached to the event.

Returns:
    Parsed external event.

Raises:
    TypeError: If the envelope is not a JSON object.
    ValueError: If any envelope field is missing, of the wrong type, or
        otherwise invalid.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
