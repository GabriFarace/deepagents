# `libs/partners/quickjs/pyproject.toml`

## Package Identity

| Field | Value |
|-------|-------|
| Name | `langchain-quickjs` |
| Version | `0.0.1` |
| Description | QuickJS integration package for Deep Agents |
| Python requirement | `>=3.11,<4.0` |
| Build backend | `hatchling` |
| Wheel packages | `["langchain_quickjs"]` |

## Runtime Dependencies

| Package | Version Constraint | Purpose |
|---------|--------------------|---------|
| `deepagents` | (any) | Core agent framework (local path in dev) |
| `quickjs` | `>=1.19.4,<2` | Embedded JavaScript engine |

## Test Dependencies

| Package | Version Constraint |
|---------|--------------------|
| `pytest` | `>=7.3.0,<9.0.0` |
| `pytest-cov` | any |
| `pytest-socket` | any |
| `pytest-xdist` | any |
| `pytest-timeout` | `>=2.3.1,<3.0.0` |
| `pytest-asyncio` | `>=1.3.0` |
| `pytest-watcher` | `>=0.3.4,<1.0.0` |
| `ruff` | `>=0.13.1,<0.16.0` |
| `ty` | `>=0.0.1,<1.0.0` |
| `twine` | any |
| `build` | any |

## Linting Configuration

- Selects all Ruff rules (`"ALL"`)
- Ignores: `COM812`, `ISC001`, `ANN401`, `ASYNC109` (StructuredTool async wrapper should mirror sync tool parameters)
- Docstring convention: Google style
- `ban-relative-imports = "all"`
- Tests ignore `S101`, `D`, `ANN`, `ARG`, `PLR2004`, `FBT`, `INP001`, `SLF001`

## Type Checking

- `ty` type checker, Python 3.11 target, extra path `../../deepagents`

## pytest Configuration

- `asyncio_mode = "auto"`
- Strict markers and config
- Custom markers: `requires`, `compile`, `scheduled`
