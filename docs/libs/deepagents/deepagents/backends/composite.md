# `deepagents/backends/composite.py`

## High-Level Purpose

`CompositeBackend` routes file operations to different backends based on path prefix. This lets you combine backends: for example, route `/workspace/**` to a local filesystem backend and `/remote/**` to a cloud sandbox backend. The agent sees a single unified backend interface.

---

## Key Class

### `CompositeBackend`

Implements `BackendProtocol` (or `SandboxBackendProtocol` if any sub-backend supports `execute`).

**Constructor parameters:**

| Parameter | Type | Description |
|---|---|---|
| `routes` | `list[tuple[str, BackendProtocol]]` | `(prefix, backend)` pairs; first match wins |
| `default` | `BackendProtocol \| None` | Fallback for paths that don't match any prefix |

**Example:**

```python
from deepagents.backends import CompositeBackend, FilesystemBackend, StateBackend

backend = CompositeBackend(
    routes=[
        ("/workspace/", FilesystemBackend(root_dir="/workspace")),
        ("/scratch/", StateBackend()),
    ],
    default=StateBackend(),
)
```

---

## Routing Rules

- Routes are checked in order; first matching prefix wins
- If no route matches and `default` is set: uses `default`
- If no route matches and `default` is `None`: raises `BackendError`
- Permissions rules passed to `create_deep_agent()` should scope to the prefixes used in routes

---

## Architecture Notes

**execute() routing:** `execute()` is dispatched to the backend whose route matches the current working directory. If no route matches, uses the first backend that implements `SandboxBackendProtocol`.

---

## See Also

- [README.md](README.md) — backend comparison
- [protocol.md](protocol.md) — `BackendProtocol` interface
