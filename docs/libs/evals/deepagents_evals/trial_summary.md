# `evals/deepagents_evals/trial_summary.py`

> Helpers for rendering trial summary tables in the GHA step summary.

## Position in the system

This module belongs to the evaluation CLI/reporting layer. It reads pytest/Harbor outputs, aggregates results, and renders human-facing summaries or charts.

## Imports and module-level state

This file imports `__future__`.

## Functions and classes

### `render_per_trial_category_matrix(trials: list[dict], cat_keys: list[str], labels: dict[str, str] | None=None, *, places: int=3)`

Build the per-trial-by-per-category correctness table as markdown lines. Each row is one trial; each column is one category. Cells are correctness scores formatted to `places` decimals; categories that did not run in a given trial render as `-` so a missing column is visually distinct from a 0.0 score. Args: trials: Per-trial summary dicts (each must carry `trial_index` and optionally `category_scores`). cat_keys: Category keys to render as columns, in display order. labels: Optional human-friendly labels keyed by category. Falls back to the raw key when a label is missing. places: Decimal places for score cells. Returns: Markdown lines (blank line, heading, blank line, header row, separator, data rows). An empty list when `trials` or `cat_keys` is empty so callers can unconditionally extend a buffer. Key arguments are `trials`, `cat_keys`, `labels`, `places`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `append`, `join`, `get`. It runs synchronously in the caller and returns directly.

## Gotchas

Most behavior is delegated through framework protocols, so preserve method signatures when editing even if an argument appears unused locally.
