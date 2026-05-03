# `libs/deepagents/` — Core SDK Package

This package provides the `deepagents` Python SDK — the foundation that all other packages (CLI, ACP, evals) build on.

## Package Structure

```
libs/deepagents/
├── deepagents/              ← Python package (see deepagents/README.md)
│   ├── __init__.py          ← Public exports
│   ├── graph.py             ← create_deep_agent() — main entry point
│   ├── _models.py           ← model resolution
│   ├── backends/            ← storage backends
│   └── middleware/          ← agent middleware
├── tests/                   ← unit + integration tests
└── pyproject.toml
```

## Quick Reference

```bash
# Install
uv add deepagents

# Minimal usage
from deepagents import create_deep_agent
graph = create_deep_agent()
result = graph.invoke({"messages": [{"role": "user", "content": "hello"}]})
```

## See Also

- [deepagents/README.md](deepagents/README.md) — SDK architecture
- [deepagents/graph.md](deepagents/graph.md) — full API reference
