# `deepagents/backends/filesystem.py`

## High-Level Purpose

`FilesystemBackend` provides direct read/write access to local disk files without shell execution. It is the simplest persistent backend — useful when you need the agent to read and write files but don't need it to run shell commands.

---

## Key Class

### `FilesystemBackend`

Implements `BackendProtocol` (no `execute`).

**Constructor parameters:**

| Parameter | Default | Description |
|---|---|---|
| `root_dir` | `os.getcwd()` | All relative paths are resolved relative to this directory |

All file operations use `asyncio.to_thread` wrapping synchronous `pathlib` calls.

---

## Behavior Notes

- Paths must be absolute POSIX paths
- Binary files are auto-detected and base64-encoded in results
- `edit()` reads the file, performs the string replacement, and writes it back atomically
- `glob()` uses `pathlib.Path.glob()` with `**` recursive support
- `grep()` uses Python's `re.search()` on each line (literal string or regex depending on call site)

---

## See Also

- [README.md](README.md) — comparison with other backends
- [local_shell.md](local_shell.md) — adds `execute()` on top of filesystem access
