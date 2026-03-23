# `deepagents/backends/__init__.py`

## High-Level Purpose

Package initializer for the `deepagents.backends` subpackage. Re-exports all backend classes and related types so that users and internal code can import from `deepagents.backends` directly.

## Exports

| Symbol | Source Module | Purpose |
|---|---|---|
| `DEFAULT_EXECUTE_TIMEOUT` | `local_shell` | Default shell execution timeout (120 seconds) |
| `BackendContext` | `store` | Dataclass passed to namespace factory functions |
| `BackendProtocol` | `protocol` | Abstract base class for all backends |
| `CompositeBackend` | `composite` | Routes file ops to different backends by path prefix |
| `FilesystemBackend` | `filesystem` | Reads/writes real files on disk |
| `LangSmithSandbox` | `langsmith` | Wraps a LangSmith sandbox for isolated execution |
| `LocalShellBackend` | `local_shell` | Filesystem backend with local shell execution |
| `NamespaceFactory` | `store` | Type alias for namespace factory callables |
| `StateBackend` | `state` | Stores files in ephemeral LangGraph agent state |
| `StoreBackend` | `store` | Persistent cross-thread storage via LangGraph BaseStore |

## Usage

```python
from deepagents.backends import StateBackend, FilesystemBackend, StoreBackend
from deepagents.backends import CompositeBackend, LangSmithSandbox, LocalShellBackend
from deepagents.backends import BackendProtocol
```
