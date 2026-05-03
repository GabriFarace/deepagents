# `deepagents/backends/` — Storage & Execution Backends

Backends are the layer that gives an agent access to files and shell. Every backend implements `BackendProtocol` — a uniform interface for filesystem operations. When you pass a backend to `create_deep_agent()`, the middleware stack (especially `FilesystemMiddleware`) uses it for all file and shell tool calls.

---

## Backend Inventory

| Backend | File | Shell (`execute`) | Storage | Use case |
|---|---|---|---|---|
| [`StateBackend`](state.md) | In LangGraph state | No | Ephemeral per-thread | Testing, in-memory agents |
| [`FilesystemBackend`](filesystem.md) | Local disk | No | Persistent | Read-only disk access |
| [`LocalShellBackend`](local_shell.md) | Local disk | Yes (local shell) | Persistent | CLI / trusted environments |
| [`StoreBackend`](store.md) | LangGraph Store | No | Cross-thread persistent | Shared memory between sessions |
| [`CompositeBackend`](composite.md) | Routes by prefix | Depends | Varies | Combine multiple backends |
| [`BaseSandbox`](sandbox.md) | Via `execute()` | Yes (sandboxed) | Depends on sandbox | Untrusted code execution |
| [`LangSmithSandbox`](sandbox.md) | Via `execute()` | Yes (remote) | Remote | LangSmith-managed sandbox |

Partner backends (Daytona, Modal, RunLoop, QuickJS) extend `BaseSandbox`.

---

## Choosing a Backend

**For local development:** `LocalShellBackend` — gives the agent full disk and shell access. Used by the CLI.

**For untrusted code (user-submitted scripts, CI):** Use a partner sandbox backend (Modal, Daytona, etc.). They implement the same `BackendProtocol` so nothing else changes.

**For in-memory/testing:** `StateBackend` — files live in LangGraph state, no disk access. Zero setup.

**For read-only filesystem access (no shell):** `FilesystemBackend` with `root_dir` set.

**For cross-thread persistent data (e.g., shared notes):** `StoreBackend` with a LangGraph `BaseStore`.

**For multi-zone access (local + remote):** `CompositeBackend` routing by path prefix.

---

## Common Interface (BackendProtocol)

All backends implement:

| Method | Description |
|---|---|
| `ls(path)` | List directory contents |
| `read(path)` | Read a file |
| `write(path, data)` | Write a file |
| `edit(path, old, new)` | Replace text in a file |
| `glob(pattern)` | Find files matching a pattern |
| `grep(path, pattern)` | Search file contents |
| `upload_files(files)` | Bulk upload |
| `download_files(paths)` | Bulk download |

`SandboxBackendProtocol` adds:

| Method | Description |
|---|---|
| `execute(command, env)` | Run a shell command; return stdout/stderr/exit code |

---

## File Data Format (v2)

All file data uses this format:
```python
{
    "content": "file content as string or base64",
    "encoding": "utf-8" | "base64",
    "created_at": "2024-01-01T00:00:00Z",
    "modified_at": "2024-01-01T00:00:00Z",
}
```

Binary files are base64-encoded. Backends transparently encode/decode.

---

## See Also

- [protocol.md](protocol.md) — full `BackendProtocol` interface definition
- [state.md](state.md) — ephemeral in-memory backend
- [local_shell.md](local_shell.md) — local disk + shell (used by CLI)
- [composite.md](composite.md) — routing multiple backends
- [sandbox.md](sandbox.md) — sandbox base class for partner backends
