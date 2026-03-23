# `tests/test_main.py`

## High-Level Purpose

Minimal smoke test verifying that the `deepagents_acp.__main__` module can be imported without errors.

## Test Functions

### `test_import_main_module() -> None`

**Purpose:** Import `deepagents_acp.__main__` and verify no `ImportError` or other exception is raised.

**Key Logic:** Uses a bare `from deepagents_acp import __main__` import.

## Important Imports and Dependencies

None beyond the standard pytest runner. The test exercises the module-level import of `deepagents_acp.__main__`.
