# `libs/partners/daytona/pyproject.toml`

## Package Identity

| Field | Value |
|-------|-------|
| Name | `langchain-daytona` |
| Version | `0.0.4` |
| Description | Daytona sandbox integration for Deep Agents |
| Python requirement | `>=3.11,<4.0` |
| Build backend | `hatchling` |
| Wheel packages | `["langchain_daytona"]` |

## Runtime Dependencies

| Package | Version Constraint | Purpose |
|---------|--------------------|---------|
| `deepagents` | `>=0.4.10,<0.5` | Core agent framework (local editable in dev; upper bound pins to a compatible release) |
| `daytona` | (any) | Daytona workspace SDK |

## Test Dependencies

| Package | Version Constraint |
|---------|--------------------|
| `pytest` | `>=7.3.0,<10.0.0` |
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
