# `libs/partners/quickjs` — deepagents-quickjs

## What This Package Does

`deepagents-quickjs` provides a JavaScript REPL middleware for Deep Agents, backed by [`quickjs-rs`](https://github.com/theduke/quickjs-rs) — a Rust-native binding to the embedded [QuickJS](https://bellard.org/quickjs/) JavaScript engine. It gives agents a persistent, per-thread scripting environment for computation, data manipulation, skill module execution, and calling agent tools directly from JavaScript (programmatic tool calling / PTC).

> **Version note:** This package was rewritten in v0.1.0. The middleware class is now `REPLMiddleware` (was `QuickJSMiddleware`), backed by `quickjs-rs` (was `quickjs`). The REPL is now **stateful per thread** (was stateless). The tool is named `"eval"` (was `"repl"`).

## Directory Layout

```
libs/partners/quickjs/
├── deepagents_quickjs/
│   ├── __init__.py           # Re-exports REPLMiddleware, PTCOption
│   ├── middleware.py         # REPLMiddleware — AgentMiddleware implementation
│   ├── _repl.py              # Per-thread Context management (quickjs_rs Runtime/Context)
│   ├── _ptc.py               # Programmatic tool calling bridge and budget enforcement
│   ├── _skills.py            # Skill module loader (dynamic ES module imports)
│   ├── _prompt.py            # System prompt builder
│   └── _format.py            # Result formatting and truncation
├── tests/unit_tests/
│   ├── test_end_to_end.py
│   ├── test_end_to_end_async.py
│   ├── test_middleware.py
│   └── benchmarks/           # Performance benchmarks (codspeed)
├── pyproject.toml
├── Makefile
└── LICENSE
```

## How to Use

```python
from deepagents_quickjs import REPLMiddleware
from deepagents import create_deep_agent

# Basic persistent REPL
middleware = REPLMiddleware()

# REPL with programmatic tool calling (PTC)
agent = create_deep_agent(
    model="anthropic:claude-sonnet-4-6",
    middleware=[
        REPLMiddleware(
            ptc=["read_file", "grep"],    # Tools callable from JS as tools.readFile(...)
            max_ptc_calls=128,            # Budget per eval call
            timeout=10.0,
            memory_limit=128 * 1024 * 1024,
        )
    ],
)

# REPL with skill module imports
from deepagents.middleware.skills import SkillsMiddleware
from deepagents.backends.filesystem import FilesystemBackend

skills_backend = FilesystemBackend(root_dir="/my/project", virtual_mode=True)
agent = create_deep_agent(
    skills=["/skills/"],
    backend=skills_backend,
    middleware=[
        REPLMiddleware(skills_backend=skills_backend)
    ],
)
# Inside JS: const { helper } = await import("@/skills/web-research")
```

## How Files Relate

- **`middleware.py`** is the public entry point. Assembles the `eval` tool and system prompt.
- **`_repl.py`** manages per-thread `quickjs_rs.Context` instances (one context per LangGraph `thread_id`, shared `Runtime` per `REPLMiddleware` instance).
- **`_ptc.py`** handles the PTC bridge: wrapping agent tools as JS-callable `tools.*` functions, enforcing the `max_ptc_calls` budget, and raising `PTCCallBudgetExceeded` on overflow.
- **`_skills.py`** loads skill module source files from `skills_backend` and installs them as dynamic ES modules under the `@/skills/<name>` import path.
- **`_prompt.py`** builds the system prompt fragment, including optional PTC docs and skill module listings.
- **`_format.py`** handles result formatting and truncation at `max_result_chars`.

## Key Design Decisions

- **Stateful REPL per thread:** Each LangGraph thread gets its own JS context. Variables and functions persist across `eval` calls within one conversation. Contexts are isolated between threads (no global leakage).
- **`quickjs-rs` (Rust) vs `quickjs` (Python):** The Rust binding is significantly faster and more memory-efficient. It supports one `Runtime` shared across threads with per-thread `Context` objects — the correct model for multi-tenant agent serving.
- **PTC budget:** `max_ptc_calls` (default 256) prevents runaway loops when the model calls `eval` with code that calls `tools.*` in a tight loop. Setting to `None` disables the budget and should only be used in trusted environments.
- **Skill modules as ES modules:** Skills with a `module` field in their SKILL.md are importable via `await import("@/skills/<name>")`, enabling rich JS tooling from skill directories.
- **JSON round-tripping for PTC:** Tools return strings; `_ptc.py` parses JSON responses into native JS objects where possible, so the JS caller gets arrays/objects rather than raw JSON strings.

## Package Metadata

| Field | Value |
|-------|-------|
| Name | `deepagents-quickjs` |
| Version | `0.1.0` |
| Python requirement | `>=3.11,<4.0` |
| Key dependency | `quickjs-rs` (Rust-native QuickJS binding) |
