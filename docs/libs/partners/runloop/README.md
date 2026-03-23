# `libs/partners/runloop` — langchain-runloop

## What This Package Does

`langchain-runloop` provides the `RunloopSandbox` backend, connecting Deep Agents to [Runloop](https://runloop.ai/) devboxes. Runloop offers on-demand, pre-configured development environments suitable for code execution tasks.

## Directory Layout

```
libs/partners/runloop/
├── langchain_runloop/
│   ├── __init__.py     # Re-exports RunloopSandbox
│   └── sandbox.py      # RunloopSandbox implementation
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
from runloop_api_client import RunloopApi
from langchain_runloop import RunloopSandbox
from deepagents import create_deep_agent

api = RunloopApi(bearer_token="your-token")
devbox = api.devboxes.create()
backend = RunloopSandbox(devbox=devbox)
agent = create_deep_agent(model="anthropic:claude-sonnet-4", backend=backend)
```

## Key Design Decisions

- **Runloop native file API**: `download_files` and `upload_files` use `devbox.file.download/upload` directly.
- **Shell for all other operations**: Like other partner sandboxes, shell command routing through `execute()` and `BaseSandbox` handles file manipulation tools.
- **Stable ID storage**: The devbox ID is captured at construction time to ensure stability.
