# `integrations/` — External System Integrations

This directory contains adapters for external sandbox execution environments. It provides a clean abstraction for creating, using, and cleaning up isolated code execution sandboxes during CLI sessions.

## Overview

When the `--sandbox` flag is passed to the CLI, code execution is routed to a remote or containerized environment instead of the local machine. The integrations package provides the abstractions and implementations for this.

## Files

| File | Purpose |
|---|---|
| `__init__.py` | Package docstring only |
| `sandbox_provider.py` | Abstract `SandboxProvider` interface + error types |
| `sandbox_factory.py` | Sandbox lifecycle management (create, setup, cleanup) + provider loading |

## Supported Sandbox Providers

| Provider | Flag Value | Package Required |
|---|---|---|
| Local (no sandbox) | `none` | (none) |
| LangSmith | `langsmith` | `langsmith[sandbox]` (included in base deps) |
| AgentCore | `agentcore` | `langchain-agentcore-codeinterpreter` (optional extra) |
| Modal | `modal` | `modal` (optional extra) |
| Daytona | `daytona` | separate extra |
| Runloop | `runloop` | separate extra |

## Architecture

```
CLI (main.py)
    │
    ├── --sandbox langsmith → SandboxFactory.create_sandbox("langsmith")
    │                              → importlib loads langsmith provider module
    │                              → LangSmithSandboxProvider.get_or_create()
    │                              → returns SandboxBackendProtocol
    │
    └── --sandbox-setup PATH → _run_sandbox_setup(backend, PATH)
                                   → expand ${VAR}, run bash -c in sandbox
```

## SandboxProvider Interface

All providers implement:
- `get_or_create(*, sandbox_id=None, **kwargs) -> SandboxBackendProtocol` — synchronous
- `delete(*, sandbox_id, **kwargs) -> None` — synchronous
- `aget_or_create(...)` / `adelete(...)` — async wrappers (via `asyncio.to_thread`)

## Error Handling

- `SandboxError` — base for all sandbox errors
- `SandboxNotFoundError` — raised when `--sandbox-id` is provided but the sandbox no longer exists
