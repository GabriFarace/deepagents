# `libs/cli/` — deepagents-cli Package

This package contains the `deepagents` CLI command — a full-featured interactive terminal application for running AI agents.

## Package Structure

```
libs/cli/
├── deepagents_cli/          ← Python package (see deepagents_cli/README.md)
├── frontend/                ← Optional web frontend (Vite/TS, served by langgraph dev)
├── tests/
│   └── unit_tests/          ← Unit tests (no server required)
├── DEV.md                   ← Developer notes for CLI contributors
├── THREAT_MODEL.md          ← Security threat model
└── pyproject.toml           ← Package config
```

## Quick Reference

```bash
# Run the CLI
cd libs/cli && uv run deepagents

# Run unit tests
uv run pytest tests/unit_tests/

# Lint
make lint

# Format
make format
```

## See Also

- [deepagents_cli/README.md](deepagents_cli/README.md) — full architecture documentation
