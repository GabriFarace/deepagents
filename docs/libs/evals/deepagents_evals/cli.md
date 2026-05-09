# `evals/deepagents_evals/cli.py`

> Unified, agent-friendly CLI for the Deep Agents evaluation suite.

## Position in the system

This module belongs to the evaluation CLI/reporting layer. It reads pytest/Harbor outputs, aggregates results, and renders human-facing summaries or charts.

## Imports and module-level state

This file imports `__future__, argparse, json, os, subprocess, sys, pathlib, typing`.
Module constants worth noticing: `EXIT_OK`, `EXIT_EVAL_FAILURES`, `EXIT_CONFIG`, `EXIT_NO_REPORTS`, `_PACKAGE_DIR`, `_EVALS_DIR`, `_REPO_ROOT`, `_CATEGORIES_JSON`, `_KNOWN_TIERS`, `_MODEL_ENV_VAR`.

## Functions and classes

### `main(argv: Sequence[str] | None=None)`

Entry point for the `deepagents-evals` console script. Args: argv: Optional argv override (mostly for tests). When `None`, `argparse` reads from `sys.argv`. Returns: One of the `EXIT_*` constants. Key arguments are `argv`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `parse_args`, `func`, `error`, `hasattr`. It runs synchronously in the caller and returns directly.

## Gotchas

Most behavior is delegated through framework protocols, so preserve method signatures when editing even if an argument appears unused locally.
