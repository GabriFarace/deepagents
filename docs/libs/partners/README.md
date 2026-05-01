# `libs/partners` — Partner Integrations

## What This Directory Contains

The `partners` directory contains LangChain partner packages that provide sandbox backend implementations and agent middleware for the Deep Agents framework. Each partner package is independently installable and published.

## Packages

| Package | Directory | PyPI Name | Description |
|---------|-----------|-----------|-------------|
| Daytona | `daytona/` | `langchain-daytona` | Sandbox backend for [Daytona](https://www.daytona.io/) cloud dev environments |
| Modal | `modal/` | `langchain-modal` | Sandbox backend for [Modal](https://modal.com/) serverless containers (supports GPU) |
| QuickJS | `quickjs/` | `langchain-quickjs` | Persistent JavaScript REPL middleware backed by quickjs-rs, with programmatic tool calling (PTC) and skill module imports |
| Runloop | `runloop/` | `langchain-runloop` | Sandbox backend for [Runloop](https://runloop.ai/) devboxes |

## Common Architecture

All sandbox backends (`daytona`, `modal`, `runloop`) follow the same pattern:
1. Accept an existing vendor sandbox/devbox object in `__init__`.
2. Implement `execute(command, *, timeout)` using the vendor's execution API.
3. Implement `download_files` / `upload_files` using the vendor's file API.
4. Inherit all shell-based file operations (read, write, edit, ls, glob, grep) from `deepagents.backends.sandbox.BaseSandbox`.

The `quickjs` package is different — it provides an `AgentMiddleware` rather than a sandbox backend, adding a JavaScript REPL tool to any agent.

## How to Use a Sandbox Backend

```python
# Example with any of the three sandbox backends
from langchain_<provider> import <Provider>Sandbox
from deepagents import create_deep_agent

backend = <Provider>Sandbox(sandbox=existing_sandbox_object)
agent = create_deep_agent(model="anthropic:claude-sonnet-4", backend=backend)
```

## Directory Structure

```
libs/partners/
├── daytona/
│   └── langchain_daytona/sandbox.py    # DaytonaSandbox
├── modal/
│   └── langchain_modal/sandbox.py      # ModalSandbox
├── quickjs/
│   └── langchain_quickjs/
│       ├── middleware.py               # REPLMiddleware (persistent JS REPL)
│       ├── _repl.py                    # Per-thread Context management (quickjs-rs)
│       ├── _ptc.py                     # Programmatic tool calling bridge + budget
│       ├── _skills.py                  # Skill module loader (dynamic ES imports)
│       ├── _prompt.py                  # System prompt builder
│       └── _format.py                  # Result formatting and truncation
└── runloop/
    └── langchain_runloop/sandbox.py    # RunloopSandbox
```
