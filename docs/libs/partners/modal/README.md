# `libs/partners/modal` — langchain-modal

## What This Package Does

`langchain-modal` provides the `ModalSandbox` backend, connecting Deep Agents to [Modal](https://modal.com/) cloud sandboxes. Modal sandboxes offer serverless container environments with optional GPU acceleration, ideal for data analysis and ML workloads.

## Directory Layout

```
libs/partners/modal/
├── langchain_modal/
│   ├── __init__.py     # Re-exports ModalSandbox
│   └── sandbox.py      # ModalSandbox implementation
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
import modal
from langchain_modal import ModalSandbox
from deepagents import create_deep_agent

sandbox = modal.Sandbox.create(app=my_app, image=my_image)
backend = ModalSandbox(sandbox=sandbox)
agent = create_deep_agent(model="anthropic:claude-sonnet-4", backend=backend)
```

See the `nvidia_deep_agent` example for a production-style usage with GPU images and file seeding.

## Key Design Decisions

- **Direct filesystem API for file I/O**: `download_files` and `upload_files` use `sandbox.open()` rather than shell commands for efficiency.
- **Shell for all other operations**: `execute()` uses `bash -c`, and all file manipulation tools (read, write, edit, ls, grep, glob) go through shell commands via `BaseSandbox`.
- **GPU-ready**: Modal supports attaching GPU resources to sandboxes, making this backend well-suited for NVIDIA RAPIDS/cuML workflows.
