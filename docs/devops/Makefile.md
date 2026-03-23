# Makefile (Root)

## Overview

The root-level `Makefile` provides monorepo-wide commands for managing all packages. It automatically discovers package directories by globbing for `libs/*/Makefile` and `libs/partners/*/Makefile`, so new packages are picked up without modifying this file.

## Location

`/Makefile`

## Package Discovery

```makefile
PACKAGE_DIRS := $(sort $(patsubst %/,%,$(dir $(wildcard libs/*/Makefile libs/partners/*/Makefile))))
```

Sorted alphabetically to ensure deterministic ordering.

## Python Version Mapping

```makefile
python_version = $(if $(filter libs/acp,$1),3.14,3.12)
```

- `libs/acp`: Python 3.14
- All other packages: Python 3.12

## Targets

### `help` (default)

Displays available targets with descriptions. Auto-generated from `##` comments in the Makefile.

```
make help
```

### `lock`

Regenerates all `uv.lock` files across all package directories.

```
make lock
```

Calls `uv lock --directory <dir> --python <version>` for each package directory.

### `lock-check`

Verifies all `uv.lock` files are up-to-date without modifying them.

```
make lock-check
```

Calls `uv lock --check --directory <dir> --python <version>` for each package. Fails fast (`set -e`) if any lockfile is stale.

Used by:
- `check_lockfiles.yml` CI workflow
- `lock-check` pre-commit hook

### `lint`

Runs `make lint` in every package directory.

```
make lint
```

Each package's own `Makefile` defines the actual lint commands (typically `ruff check` and `ruff format --check`).

### `format`

Runs `make format` in every package directory.

```
make format
```

Each package's own `Makefile` defines the format commands (typically `ruff format` and `ruff check --fix`).

## Usage Examples

```bash
# Update all lockfiles after changing pyproject.toml
make lock

# Verify lockfiles in CI
make lock-check

# Lint the entire monorepo
make lint

# Format the entire monorepo
make format
```

## Notes

- Individual package `Makefile`s define `test`, `lint`, `format`, `evals`, and other targets.
- The root Makefile delegates lint and format to per-package Makefiles; it does not define `test` at the root level (CI runs per-package tests directly).
