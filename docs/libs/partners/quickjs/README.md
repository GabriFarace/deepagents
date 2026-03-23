# `libs/partners/quickjs` — langchain-quickjs

## What This Package Does

`langchain-quickjs` provides a JavaScript REPL middleware for Deep Agents, backed by the embedded [QuickJS](https://bellard.org/quickjs/) JavaScript engine. This gives agents a sandboxed scripting environment for computation, data manipulation, and calling Python-backed foreign functions from JavaScript.

## Directory Layout

```
libs/partners/quickjs/
├── langchain_quickjs/
│   ├── __init__.py                  # Re-exports QuickJSMiddleware
│   ├── middleware.py                # QuickJSMiddleware — AgentMiddleware implementation
│   ├── _foreign_functions.py        # Python-to-JS function bridge
│   └── _foreign_function_docs.py   # Prompt documentation renderer
├── tests/unit_tests/
│   ├── chat_model.py                # Fake chat model for tests
│   ├── smoke_tests/                 # System prompt snapshot tests
│   ├── test_end_to_end.py
│   ├── test_end_to_end_async.py
│   ├── test_foreign_function_docs.py
│   ├── test_import.py
│   └── test_system_prompt.py
├── pyproject.toml
├── Makefile
└── LICENSE
```

## How to Use

```python
from langchain_quickjs import QuickJSMiddleware
from deepagents import create_deep_agent

# Basic REPL with no foreign functions
middleware = QuickJSMiddleware()

# REPL with a Python function exposed to JavaScript
def add_numbers(a: int, b: int) -> int:
    """Add two numbers."""
    return a + b

middleware_with_ptc = QuickJSMiddleware(
    ptc=[add_numbers],
    add_ptc_docs=True,   # Include TypeScript-style docs in system prompt
    timeout=30,          # 30 second per-evaluation timeout
)

agent = create_deep_agent(model="anthropic:claude-sonnet-4", middleware=[middleware_with_ptc])
```

## How Files Relate

- **`middleware.py`** is the public entry point. It assembles the REPL tool and system prompt.
- **`_foreign_functions.py`** handles all the Python↔JavaScript bridging: wrapping callables, routing async via a background event loop, installing JS shims.
- **`_foreign_function_docs.py`** converts Python signatures and docstrings to TypeScript-like stubs for the model to read.

## Key Design Decisions

- **Stateless REPL**: Each `repl` tool invocation creates a fresh QuickJS context. Variables from previous calls do not persist. The system prompt makes this explicit.
- **JSON round-tripping**: Python functions returning lists/dicts have their values JSON-encoded on the Python side and auto-parsed on the JavaScript side via shim functions, giving the JS caller native arrays and objects.
- **Async bridging**: When JavaScript calls an async foreign function, the result is resolved on a shared background event loop thread, blocking the QuickJS call until complete.

## Package Metadata

| Field | Value |
|-------|-------|
| Name | `langchain-quickjs` |
| Version | `0.0.1` |
| Python requirement | `>=3.11,<4.0` |
| Dependencies | `deepagents`, `quickjs>=1.19.4,<2` |
