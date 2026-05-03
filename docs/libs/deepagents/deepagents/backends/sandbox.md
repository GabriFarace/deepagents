# `deepagents/backends/sandbox.py`

## High-Level Purpose

`sandbox.py` provides `BaseSandbox`, the abstract base class for all cloud sandbox backends (Modal, Daytona, RunLoop, QuickJS). It implements the filesystem half of `SandboxBackendProtocol` by layering all file operations (`ls`, `read`, `write`, etc.) on top of a single `execute()` method — because in a remote sandbox, the filesystem is only reachable via shell commands.

Concrete sandbox implementations (in `libs/partners/`) override only `execute()`.

---

## Key Class

### `BaseSandbox`

Implements `SandboxBackendProtocol`.

**Abstract method (must override):**
```python
async def execute(self, command: str, env: dict | None = None) -> ExecuteResult:
    ...
```

**Derived filesystem methods (implemented in terms of `execute`):**

- `ls(path)` → runs `ls -la --json {path}` (or equivalent)
- `read(path)` → runs `cat {path}` (and `base64` for binary)
- `write(path, data)` → runs `mkdir -p + echo/tee` pipeline
- `edit(path, old, new)` → reads, Python-replaces, writes back
- `glob(pattern)` → runs `find` with pattern
- `grep(path, pattern)` → runs `grep -rn`

All commands are run in the sandbox's remote environment.

---

## `LangSmithSandbox`

Also in this file. Concrete implementation that executes commands in a LangSmith-managed sandbox environment. Used when `LANGSMITH_SANDBOX_API_KEY` is set.

---

## Partner Sandbox Backends

Partner backends in `libs/partners/` extend `BaseSandbox` and override only `execute()`:

| Package | Sandbox |
|---|---|
| `deepagents-modal` | Modal serverless functions |
| `deepagents-daytona` | Daytona cloud workspaces |
| `deepagents-runloop` | RunLoop cloud sandboxes |
| `deepagents-quickjs` | In-process QuickJS (JavaScript sandbox) |

---

## See Also

- [README.md](README.md) — when to use sandbox backends
- [local_shell.md](local_shell.md) — local alternative (no isolation)
