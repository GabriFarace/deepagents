# Script: `check_version_equality.py`

## Overview

Verifies that the version number in `pyproject.toml` matches the `__version__` string in `_version.py` for both the SDK and CLI packages. Prevents releases with mismatched version numbers that could cause confusion or broken installs.

## Location

`.github/scripts/check_version_equality.py`

## Usage in CI/CD

Called in:
- `check_versions.yml` workflow: `python .github/scripts/check_version_equality.py`
- Pre-commit hook (`version-equality` in `.pre-commit-config.yaml`)

Run from any directory — the script resolves paths relative to itself.

## Checked Packages

Hardcoded in the `PACKAGES` constant:

| `pyproject.toml` | `_version.py` |
|---|---|
| `libs/deepagents/pyproject.toml` | `libs/deepagents/deepagents/_version.py` |
| `libs/cli/pyproject.toml` | `libs/cli/deepagents_cli/_version.py` |

## Functions

### `_get_pyproject_version(path: Path) -> str`

Reads a `pyproject.toml` file and returns the value of `project.version`. Raises `ValueError` if the key is missing.

### `_get_version_py(path: Path) -> str`

Reads a `_version.py` file and extracts the `__version__` string using the regex `^__version__\s*=\s*"([^"]+)"`. Raises `ValueError` if not found.

### `main() -> int`

Core logic:
1. Resolves the repository root (two levels above the script)
2. For each package pair, checks that both files exist
3. Compares versions from `pyproject.toml` and `_version.py`
4. Collects errors and reports them
5. Returns `0` if all match, `1` if any mismatch

## Exit Codes

| Code | Meaning |
|---|---|
| `0` | All versions match |
| `1` | One or more mismatches (or missing files) |

## Example Output (Failure)

```
Version mismatch detected:
  deepagents: pyproject.toml=0.1.5, _version.py=0.1.4
```

## Example Output (Success)

```
deepagents versions match: 0.1.5
deepagents_cli versions match: 0.2.1
```

## Dependencies

- Standard library only: `re`, `sys`, `tomllib`, `pathlib`
