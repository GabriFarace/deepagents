# Script: `check_extras_sync.py`

## Overview

Validates that optional extras in `[project.optional-dependencies]` stay in sync with required dependencies in `[project.dependencies]`. When the same package name appears in both sections, their version constraints must match. This prevents silent version drift — for example, bumping a required dependency but forgetting to update the corresponding extra.

## Location

`.github/scripts/check_extras_sync.py`

## Usage in CI/CD

Called in:
- `check_extras_sync.yml` workflow: `python .github/scripts/check_extras_sync.py libs/cli/pyproject.toml`
- Pre-commit hook (`extras-sync` in `.pre-commit-config.yaml`)

Takes a `pyproject.toml` path as the first argument. Defaults to `pyproject.toml` in the current directory.

## Functions

### `_normalize(name: str) -> str`

PEP 503 normalizes a package name for comparison:
- Lowercases the name
- Replaces `-`, `.` with `_`

This ensures `my-package`, `my_package`, and `My.Package` are treated as the same package.

### `_parse_dep(dep: str) -> tuple[str, str]`

Parses a PEP 508 dependency string (e.g. `openai>=1.0,<2.0`) into a tuple of `(normalized_name, version_spec)`. Raises `ValueError` if the string cannot be parsed.

### `main(pyproject_path: Path) -> int`

Core logic:
1. Reads and parses the `pyproject.toml` using `tomllib`
2. Builds a map of `normalized_name -> version_spec` from `[project.dependencies]`
3. Iterates `[project.optional-dependencies]` and compares specs for any package that also appears in required deps
4. Collects mismatches and reports them
5. Returns `0` on success, `1` if mismatches found

## Exit Codes

| Code | Meaning |
|---|---|
| `0` | All extras are in sync |
| `1` | One or more version mismatches found |

## Example Output (Failure)

```
Extra / required dependency version mismatch:
  [langchain] openai: extra has '>=1.0' but required dep has '>=1.5'

Update the optional extras in [project.optional-dependencies] to match [project.dependencies].
```

## Dependencies

- Standard library only: `sys`, `tomllib`, `pathlib`, `re`
