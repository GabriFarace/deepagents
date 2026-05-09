# `evals/scripts/generate_model_groups.py`

> Generate MODEL_GROUPS.md from the canonical model registry.

## Position in the system

This is an executable maintenance/reporting script for the eval package. It is normally called from the command line or from the eval CLI wrappers rather than imported by the SDK.

## Imports and module-level state

This file imports `__future__, argparse, importlib.util, pathlib, typing`.
Module constants worth noticing: `_REPO_ROOT`, `_EVALS_DIR`, `_OUTPUT`, `_HEADER`.

## Functions and classes

### `generate()`

Return the full markdown content for MODEL_GROUPS.md. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `join`, `append`, `len`, `extend`, `any`, `sorted`. It runs synchronously in the caller and returns directly.

### `main()`

Entry point. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `ArgumentParser`, `add_argument`, `parse_args`, `generate`, `read_text`, `print`. It runs synchronously in the caller and returns directly.

## Gotchas

Most behavior is delegated through framework protocols, so preserve method signatures when editing even if an argument appears unused locally.
