# `evals/scripts/composite_radar.py`

> Generate a composite radar chart by overlaying multiple GitHub Actions eval runs.

## Position in the system

This is an executable maintenance/reporting script for the eval package. It is normally called from the command line or from the eval CLI wrappers rather than imported by the SDK.

## Imports and module-level state

This file imports `__future__, argparse, json, shutil, subprocess, sys, tempfile, pathlib`.
Module constants worth noticing: `_EVALS_DIR`, `_DEFAULT_REPO`, `_ARTIFACT_NAME`, `_SUMMARY_FILENAME`.

## Functions and classes

### `main()`

Entry point. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `ArgumentParser`, `add_argument`, `parse_args`, `mkdir`, `which`, `print`. It runs synchronously in the caller and returns directly.

## Gotchas

Most behavior is delegated through framework protocols, so preserve method signatures when editing even if an argument appears unused locally.
