# `libs/partners/modal/pyproject.toml`

## Package Identity

| Field | Value |
|-------|-------|
| Name | `langchain-modal` |
| Version | `0.0.2` |
| Description | Modal sandbox integration for Deep Agents |
| Python requirement | `>=3.11,<4.0` |
| Build backend | `hatchling` |
| Wheel packages | `["langchain_modal"]` |

## Runtime Dependencies

| Package | Version Constraint | Purpose |
|---------|--------------------|---------|
| `deepagents` | `>=0.4.3` | Core agent framework (local editable install in dev) |
| `modal` | (any) | Modal serverless compute SDK |

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
- Ignores: `COM812` (formatter conflict), `ISC001` (formatter conflict), `ANN401` (too strict for generic wrappers)
- Docstring convention: Google style, ignores `*args`/`**kwargs` documentation
- `ban-relative-imports = "all"`
- Tests ignore `D` (pydocstyle) and `S101` (assert usage)

## Type Checking

- `ty` type checker, Python 3.11 target, extra path pointing to `../../deepagents`

## pytest Configuration

- `asyncio_mode = "auto"`
- Strict markers and config
- Custom markers: `requires`, `compile`, `scheduled`
