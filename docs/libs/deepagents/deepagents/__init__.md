# `deepagents/__init__.py`

## High-Level Purpose

This is the top-level package initializer for the `deepagents` SDK. It re-exports the most commonly used public symbols so that users can import them directly from `deepagents` without needing to know the internal module structure.

## Exports

| Symbol | Source Module | Purpose |
|---|---|---|
| `__version__` | `deepagents._version` | Current SDK version string |
| `create_deep_agent` | `deepagents.graph` | Factory function to build a configured deep agent |
| `AsyncSubAgent` | `deepagents.middleware.async_subagents` | TypedDict spec for remote async subagents |
| `AsyncSubAgentMiddleware` | `deepagents.middleware.async_subagents` | Middleware for managing async subagents |
| `FilesystemMiddleware` | `deepagents.middleware.filesystem` | Middleware providing file tools to agents |
| `MemoryMiddleware` | `deepagents.middleware.memory` | Middleware for AGENTS.md context loading |
| `CompiledSubAgent` | `deepagents.middleware.subagents` | TypedDict spec for pre-compiled subagents |
| `SubAgent` | `deepagents.middleware.subagents` | TypedDict spec for declarative subagents |
| `SubAgentMiddleware` | `deepagents.middleware.subagents` | Middleware exposing the `task` tool |

## Important Notes

- The `__all__` list explicitly declares what is part of the public API.
- Backend classes (`StateBackend`, `FilesystemBackend`, etc.) are accessible via `deepagents.backends` but are not re-exported at the top level.
- Middleware not listed here (e.g., `SkillsMiddleware`, `SummarizationMiddleware`) must be imported directly from their submodules.

## Example Usage

```python
from deepagents import create_deep_agent, __version__
from deepagents import SubAgent, FilesystemMiddleware
```
