# Script: `aggregate_evals.py`

## Overview

Aggregates individual per-model evaluation report JSON files into a unified summary. Generates Markdown tables written to the GitHub Actions step summary, a consolidated `evals_summary.json` artifact, and optionally per-category score tables.

## Location

`.github/scripts/aggregate_evals.py`

## Usage in CI/CD

Called in the `aggregate` job of `evals.yml`:

```
uv run --with tabulate python .github/scripts/aggregate_evals.py
```

Must be run from the repository root. Expects evaluation artifact directories at `evals_artifacts/**/evals_report.json`.

## Functions

### `_format_table(rows, headers) -> list[list[object]]`

Builds tabulate-ready row lists from a list of report dictionaries. Extracts these fields per row:

| Column | Field |
|---|---|
| `model` | Model identifier string |
| `passed` | Count of passing eval cases |
| `failed` | Count of failing eval cases |
| `skipped` | Count of skipped cases |
| `total` | Total cases |
| `correctness` | Correctness score (float) |
| `solve_rate` | Solve rate or `"n/a"` |
| `step_ratio` | Step efficiency ratio or `"n/a"` |
| `tool_call_ratio` | Tool call efficiency ratio or `"n/a"` |
| `median_duration_s` | Median duration in seconds |

### `_load_category_labels() -> dict[str, str]`

Loads human-readable category display labels from `libs/evals/deepagents_evals/categories.json`. Returns an empty dict on failure (with a warning to stderr).

### `_build_category_table(rows) -> list[str]`

Builds a per-category correctness table from report rows. Collects all unique categories across all models (preserving insertion order), then renders a single Markdown table showing each model's score per category. Returns an empty list if no category data is present.

### `main() -> None`

Entry point. Performs:
1. Globs `evals_artifacts/**/evals_report.json` and loads all reports
2. Writes `evals_summary.json` (sorted keys, 2-space indent)
3. Renders **Table 1**: sorted by provider then correctness (descending)
4. Renders **Table 2**: ranked by correctness then solve_rate (both descending)
5. Renders **Table 3**: per-category correctness (if category data available)
6. Writes output to `$GITHUB_STEP_SUMMARY` if set, and also prints to stdout

## Output Files

| File | Description |
|---|---|
| `evals_summary.json` | Aggregated JSON for offline analysis and radar chart generation |
| `$GITHUB_STEP_SUMMARY` | Markdown tables visible in GitHub Actions UI |

## Dependencies

- `tabulate` — installed at runtime with `uv run --with tabulate`
- Standard library: `glob`, `json`, `os`, `sys`, `pathlib`
