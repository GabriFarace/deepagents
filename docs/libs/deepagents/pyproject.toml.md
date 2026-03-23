# `pyproject.toml`

## High-Level Purpose

The `pyproject.toml` file defines the `deepagents` Python package metadata, dependencies, build system configuration, linting rules, and test configuration. It is the single source of truth for package publishing and development tooling.

## Project Metadata

| Field | Value |
|---|---|
| `name` | `deepagents` |
| `version` | `0.5.0a2` |
| `description` | General purpose 'deep agent' with sub-agent spawning, todo list capabilities, and mock file system. Built on LangGraph. |
| `license` | MIT |
| `requires-python` | `>=3.11,<4.0` |
| `keywords` | agents, ai, llm, langgraph, langchain, deep-agent, sub-agents, agentic |

## Runtime Dependencies

| Package | Version Constraint | Purpose |
|---|---|---|
| `langchain-core` | `>=1.2.21,<2.0.0` | Base types: `BaseChatModel`, messages, tools |
| `langsmith` | `>=0.3.0` | LangSmith integration |
| `langchain` | `>=1.2.11,<2.0.0` | `create_agent`, middleware, `init_chat_model` |
| `langchain-anthropic` | `>=1.4.0,<2.0.0` | `ChatAnthropic`, `AnthropicPromptCachingMiddleware` |
| `langchain-google-genai` | `>=4.2.0,<5.0.0` | Google Gemini model support |
| `wcmatch` | latest | Extended glob pattern matching (used in filesystem backend) |

## Development Dependencies (test group)

Key test dependencies:
- `pytest`, `pytest-asyncio`, `pytest-cov`, `pytest-xdist` — test runner and utilities
- `pytest-socket` — network isolation for unit tests (`--disable-socket`)
- `pytest-benchmark`, `pytest-codspeed` — performance benchmarking
- `pytest-watcher` — file-watching test runner
- `ruff` — linting and formatting
- `ty` — type checking
- `langchain-tests` — LangChain test utilities
- `langchain-openai` — OpenAI integration for integration tests
- `twine`, `build` — package publishing

## Build System

```toml
[build-system]
requires = ["setuptools>=73.0.0", "wheel"]
build-backend = "setuptools.build_meta"
```

Package data includes `py.typed` (PEP 561 marker) and `*.md` files.

## Ruff Linting Configuration

- **Line length:** 150
- **Rules:** `ALL` enabled by default, with specific ignores:
  - `COM812`, `ISC001` — conflict with ruff formatter
  - `PERF203` — try-except in loop
  - `SLF001` — private member access
  - `PLR0913` — too many arguments
  - `PLC0414` — import alias conventions
- **Unfixable:** `B028` (not auto-corrected)
- **Import style:** Google-style docstrings, `force-single-line = false`, `known-first-party = ["deepagents"]`
- **Test file relaxations:** Missing type annotations, missing docstrings, magic values, security assertions all relaxed in `tests/**`
- **Script relaxations:** Blind exceptions and standalone module warnings relaxed in `scripts/**`

## Pytest Configuration

```toml
[tool.pytest.ini_options]
asyncio_mode = "auto"       # Automatically handle async tests
addopts = "-m 'not benchmark'"  # Skip benchmark tests by default
```

Custom markers:
- `benchmark` — wall-time benchmarks excluded from default test runs

## Type Checker Configuration

```toml
[tool.ty.environment]
python-version = "3.11"

[tool.ty.rules]
division-by-zero = "error"
```

Uses `ty` (not mypy or pyright). Targets Python 3.11 minimum.

## Project URLs

| Key | URL |
|---|---|
| Homepage | `https://docs.langchain.com/oss/python/deepagents/overview` |
| Documentation | `https://reference.langchain.com/python/deepagents/` |
| Repository | `https://github.com/langchain-ai/deepagents` |
