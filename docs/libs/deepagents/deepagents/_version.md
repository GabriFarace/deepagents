# `deepagents/_version.py`

## High-Level Purpose

Single-source-of-truth for the SDK version number. This module is intentionally minimal to allow automated tooling (e.g., release scripts) to update the version in one place.

## Constants

### `__version__`

- **Type:** `str`
- **Current value:** `"0.5.0a2"`
- **Purpose:** Identifies the installed version of the `deepagents` package. This value is imported by `deepagents/__init__.py` for re-export and also embedded in agent metadata by `graph.py` via the `versions` metadata key.

## Dependencies

None. This module has no imports and no dependencies on any other part of the package.
