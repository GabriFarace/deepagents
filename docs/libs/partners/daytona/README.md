# `libs/partners/daytona` — langchain-daytona

## What This Package Does

`langchain-daytona` provides the `DaytonaSandbox` backend, which connects Deep Agents to [Daytona](https://www.daytona.io/) cloud development sandboxes. Daytona sandboxes offer persistent, networked development environments where agents can execute code and manage files.

## Directory Layout

```
libs/partners/daytona/
├── langchain_daytona/
│   ├── __init__.py     # Re-exports DaytonaSandbox
│   └── sandbox.py      # DaytonaSandbox implementation
├── tests/
│   ├── test_import.py
│   ├── unit_tests/test_import.py
│   └── integration_tests/test_integration.py
├── pyproject.toml
├── Makefile
└── LICENSE
```

## How to Use

```python
import daytona
from langchain_daytona import DaytonaSandbox
from deepagents import create_deep_agent

sandbox = daytona.create()  # or use an existing sandbox
backend = DaytonaSandbox(sandbox=sandbox)
agent = create_deep_agent(model="anthropic:claude-sonnet-4", backend=backend)
```

## Key Design Decisions

- **Polling-based execution**: Daytona does not natively support synchronous command execution with streaming output. `DaytonaSandbox` uses a session-based polling loop to wait for command completion.
- **Configurable polling interval**: The `sync_polling_interval` parameter accepts either a fixed float or a callable that adapts the polling delay based on elapsed time (e.g., exponential backoff).
- **Inherits file operations**: All file operations (read, write, edit, ls, glob, grep) are provided by `BaseSandbox` and execute via shell commands routed through the `execute()` method.

## Package Metadata

| Field | Value |
|-------|-------|
| Name | `langchain-daytona` |
| Version | `0.0.4` |
| Python requirement | `>=3.11,<4.0` |
| Dependencies | `deepagents>=0.4.10,<0.5`, `daytona` |
