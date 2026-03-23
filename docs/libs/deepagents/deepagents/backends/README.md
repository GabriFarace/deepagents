# `deepagents/backends/` — Backend Implementations

## Overview

The `backends` package provides pluggable file storage and execution backends for the deepagents SDK. All backends implement `BackendProtocol`, a unified interface for file operations (ls, read, write, edit, grep, glob, upload, download). Sandbox backends additionally implement `SandboxBackendProtocol`, which adds `execute()` for shell command execution.

Backends are instantiated by middleware (primarily `FilesystemMiddleware`) at tool-call time, abstracting storage details from the agent's tool layer.

## File Descriptions

| File | Purpose |
|---|---|
| `__init__.py` | Re-exports all public backend classes and types |
| `protocol.py` | Defines `BackendProtocol`, `SandboxBackendProtocol`, all result types (`ReadResult`, `WriteResult`, etc.), and `BackendFactory` type alias |
| `state.py` | `StateBackend` — ephemeral in-memory storage in LangGraph agent state; the default backend |
| `filesystem.py` | `FilesystemBackend` — direct filesystem access with optional virtual path mode |
| `sandbox.py` | `BaseSandbox` — abstract base that implements all file ops via `execute()` shell commands |
| `langsmith.py` | `LangSmithSandbox` — wraps a LangSmith sandbox with LangSmith-native write/upload/download |
| `local_shell.py` | `LocalShellBackend` — `FilesystemBackend` + unrestricted local shell execution |
| `store.py` | `StoreBackend` — persistent cross-thread storage via LangGraph `BaseStore` |
| `composite.py` | `CompositeBackend` — routes file operations to different backends by path prefix |
| `utils.py` | Shared helpers: formatting, path validation, grep/glob search, file data manipulation |

## Backend Hierarchy

```
BackendProtocol (abstract)
├── StateBackend          — LangGraph state (ephemeral, default)
├── FilesystemBackend     — Real filesystem
│   └── LocalShellBackend — Filesystem + local shell execution
├── StoreBackend          — LangGraph BaseStore (persistent)
└── CompositeBackend      — Routes to multiple backends by path

SandboxBackendProtocol extends BackendProtocol
└── BaseSandbox (abstract) — implements file ops via execute()
    └── LangSmithSandbox  — wraps LangSmith sandbox API
```

## Choosing a Backend

| Use Case | Recommended Backend |
|---|---|
| Default / web server / API | `StateBackend` (default) |
| Persistent memories across conversations | `StoreBackend` |
| Local development CLI / coding assistant | `FilesystemBackend` or `LocalShellBackend` |
| Sandboxed code execution | `LangSmithSandbox` or custom `BaseSandbox` |
| Mixed storage (ephemeral + persistent) | `CompositeBackend` |

## Key Design Patterns

### Factory Pattern
Backends like `StateBackend` need a `ToolRuntime` instance to access agent state. They are often passed as factory classes (`StateBackend` rather than `StateBackend(runtime)`) and instantiated at tool call time.

### `files_update` vs `None`
- **Checkpoint backends** (`StateBackend`): Write/edit operations return `files_update={path: data}`. The middleware merges this into LangGraph state via a `Command` object.
- **External backends** (`FilesystemBackend`, `StoreBackend`, `LangSmithSandbox`): Write/edit return `files_update=None` since data is already persisted externally.

### Async Support
All methods have async variants (`als`, `aread`, `awrite`, etc.). Base implementations use `asyncio.to_thread()` for sync fallback; `StoreBackend` provides native async implementations using `store.aget`/`store.aput`.
