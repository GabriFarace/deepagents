# `evals/scripts/generate_eval_catalog.py`

> Generate `EVAL_CATALOG.md` from eval test files and `categories.json`.

## Position in the system

This is an executable maintenance/reporting script for the eval package. It is normally called from the command line or from the eval CLI wrappers rather than imported by the SDK.

## Imports and module-level state

This file imports `__future__, argparse, ast, difflib, json, pathlib`.
Module constants worth noticing: `_EVALS_DIR`, `_TESTS_DIR`, `_CATEGORIES_JSON`, `_OUTPUT`, `_GITHUB_BASE`, `_HEADER`.

## Functions and classes

### `generate()`

Return the full markdown content for `EVAL_CATALOG.md`. Category ordering and display labels are read from `categories.json`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `append`, `sum`, `join`, `open`, `load`, `set`. It runs synchronously in the caller and returns directly.

### `main()`

Entry point. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `ArgumentParser`, `add_argument`, `parse_args`, `generate`, `read_text`, `print`. It runs synchronously in the caller and returns directly.

## Gotchas

Most behavior is delegated through framework protocols, so preserve method signatures when editing even if an argument appears unused locally.
