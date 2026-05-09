# `libs/deepagents/deepagents/backends/`

> Backend abstraction for the SDK's file, search, upload/download, and shell
> execution tools.

## Position in the system

Filesystem tools in `FilesystemMiddleware` never hard-code where files live.
They call a backend method, and the selected backend decides whether the data
comes from LangGraph state, the local filesystem, a store, or a sandbox.

```
LLM tool call
  │
  ▼
FilesystemMiddleware tool
  │
  ▼
BackendProtocol method
  ├─ StateBackend
  ├─ FilesystemBackend
  ├─ LocalShellBackend
  ├─ StoreBackend
  ├─ CompositeBackend
  └─ partner sandboxes
```

## Files

| Source | Doc | Role |
|---|---|---|
| `__init__.py` | [`__init__.md`](./__init__.md) | Public backend re-exports. |
| `protocol.py` | [`protocol.md`](./protocol.md) | Shared result types and backend contracts. |
| `state.py` | [`state.md`](./state.md) | Default in-state virtual filesystem. |
| `filesystem.py` | [`filesystem.md`](./filesystem.md) | Direct disk-backed file operations. |
| `local_shell.py` | [`local_shell.md`](./local_shell.md) | Filesystem backend plus local command execution. |
| `sandbox.py` | [`sandbox.md`](./sandbox.md) | Base class for execution-backed sandboxes. |
| `store.py` | [`store.md`](./store.md) | LangGraph `BaseStore` persistence. |
| `composite.py` | [`composite.md`](./composite.md) | Path-prefix routing across multiple backends. |
| `langsmith.py` | [`langsmith.md`](./langsmith.md) | LangSmith sandbox integration. |
| `utils.py` | [`utils.md`](./utils.md) | Shared backend helpers. |

## Selection guide

Use `StateBackend` for ephemeral examples and tests. Use `FilesystemBackend`
when files should map to a real directory but shell execution is not needed.
Use `LocalShellBackend` for local CLI-style coding agents. Use partner
sandbox backends when commands need isolation or remote infrastructure. Use
`StoreBackend` when the virtual filesystem should persist across threads.

## Gotchas

`permissions` are enforced by `FilesystemMiddleware`, not by
`BackendProtocol`. Calling a backend directly bypasses those agent-level
permission rules.
