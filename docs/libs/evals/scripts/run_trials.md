# `evals/scripts/run_trials.py`

> Run the eval suite N times for the same model/config and aggregate stats.

## Position in the system

This is an executable maintenance/reporting script for the eval package. It is normally called from the command line or from the eval CLI wrappers rather than imported by the SDK.

## Imports and module-level state

This file imports `__future__, argparse, json, os, statistics, subprocess, sys, dataclasses, pathlib, typing`.
Module constants worth noticing: `_EVALS_DIR`, `_DEFAULT_OUT_DIR`, `_MAX_TRIALS`, `_SCALAR_METRICS`, `_COUNT_FIELDS`, `_MIN_SAMPLES_FOR_STDEV`, `_MODEL_ENV_VAR`.

## Functions and classes

### `aggregate_trials(reports: list[dict[str, Any]])`

Aggregate per-trial eval reports into a single summary dict. Args: reports: Per-trial report dicts as written by the eval pytest reporter (one dict per `evals_report_trial_*.json`). Returns: A dict containing `trials` (list of per-trial input reports trimmed to the fields that matter for cross-trial comparison), `metrics` (mean / median / stdev / min / max for each scalar metric), `counts` (same stats for pass/fail/skip/total), and `category_scores` (per-category stats across trials). Raises: ValueError: If `reports` is empty. Key arguments are `reports`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `sorted`, `ValueError`, `str`, `len`, `to_dict`, `get`. It runs synchronously in the caller and returns directly.

### `main(argv: list[str] | None=None)`

Run N eval trials and write `<out-dir>/trials_summary.json`. With `--aggregate-only DIR`, skip trial execution and aggregate report files already on disk (e.g. CI artifacts). Returns: Process exit code: `0` on success, `1` when no usable reports were found (either no trial produced one, or every produced file was unreadable). Key arguments are `argv`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `aggregate_trials`, `mkdir`, `write_text`, `range`, `append`, `print`. It runs synchronously in the caller and returns directly.

## Gotchas

Most behavior is delegated through framework protocols, so preserve method signatures when editing even if an argument appears unused locally.
