# `libs/partners/runloop/pyproject.toml`

## Package Identity

| Field | Value |
|-------|-------|
| Name | `langchain-runloop` |
| Version | `0.0.3` |
| Description | Runloop sandbox integration for Deep Agents |
| Python requirement | `>=3.11,<4.0` |
| Build backend | `hatchling` |
| Wheel packages | `["langchain_runloop"]` |

## Runtime Dependencies

| Package | Version Constraint | Purpose |
|---------|--------------------|---------|
| `deepagents` | `>=0.4.3` | Core agent framework (local editable install in dev) |
| `runloop-api-client` | (any) | Runloop API SDK |

## Test Dependencies

| Package | Version Constraint |
|---------|--------------------|
| `pytest` | `>=7.3.0,<9.0.0` |
| `pytest-cov` | any |
| `pytest-socket` | any |
| `pytest-xdist` | any |
| `pytest-timeout` | `>=2.3.1,<3.0.0` |
| `pytest-asyncio` | `>=1.3.0` |
| `ruff` | `>=0.13.1,<0.16.0` |
| `ty` | `>=0.0.1,<1.0.0` |
| `langchain-tests` | `>=1.1.4` |

## Linting Configuration

- Selects all Ruff rules (`"ALL"`)
- Ignores: `COM812`, `ISC001`, `ANN401`
- Docstring convention: Google style
- `ban-relative-imports = "all"`
- Tests ignore `D` and `S101`

## Type Checking

- `ty` type checker, Python 3.11 target, extra path `../../deepagents`

## pytest Configuration

- `asyncio_mode = "auto"`
- Strict markers and config
- Custom markers: `requires`, `compile`, `scheduled`
