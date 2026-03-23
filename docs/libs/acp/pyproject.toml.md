# `pyproject.toml` — deepagents-acp

## Package Metadata

| Field | Value |
|-------|-------|
| Name | `deepagents-acp` |
| Version | `0.0.4` |
| Description | Agent Client Protocol integration for Deep Agents |
| Python requirement | `>=3.11` |
| License | MIT |
| Build backend | `hatchling` |

## Runtime Dependencies

| Dependency | Version Constraint | Purpose |
|------------|--------------------|---------|
| `agent-client-protocol` | `>=0.8.0` | The ACP protocol library providing `Agent`, schema types, and helpers |
| `deepagents` | (unpinned) | Core Deep Agents SDK |
| `python-dotenv` | `>=1.2.1` | Environment variable loading |

## Optional/Group Dependencies

- **`examples`**: `langchain-openai>=1.1.11` (for running example agents)
- **`test`**: `pytest`, `pytest-asyncio`, `pytest-cov`, `pytest-mock`, `pytest-socket`, `pytest-timeout`, `pytest-watcher`, `ruff`, `ty`

## Tool Configuration

### Pytest
- `asyncio_mode = "auto"` — All async test functions are run automatically without needing `@pytest.mark.asyncio`.

### Ruff (Linting)
- `line-length = 100`
- `select = ["ALL"]` with specific ignores for formatter compatibility and `ANN401`
- Google-style docstrings (`pydocstyle.convention = "google"`)
- Relative imports banned (`ban-relative-imports = "all"`)
- Per-file ignores for `tests/**` relax many annotation, docstring, and security rules

### Type Checker (ty)
- `python-version = "3.11"`
- `division-by-zero = "error"`

## Project URLs

- Homepage / Documentation: `https://docs.langchain.com/oss/python/deepagents/overview`
- Repository: `https://github.com/langchain-ai/deepagents`
