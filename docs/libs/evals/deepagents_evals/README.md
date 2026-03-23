# `libs/evals/deepagents_evals/`

## What This Directory Contains

Shared evaluation utilities for the Deep Agents evaluation suite. Currently contains a radar chart generation module for visualizing multi-model performance across evaluation categories.

## Files

| File | Description |
|------|-------------|
| `radar.py` | Radar chart generation: `ModelResult`, `generate_radar()`, `generate_individual_radars()`, `load_results_from_summary()` |
| `categories.json` | JSON data file listing evaluation category names (included as package data) |
| `__init__.py` | Package marker with docstring "Shared eval utilities for the Deep Agents evaluation suite." |

## How It Fits In

This package (`deepagents_evals`) provides analysis and visualization utilities. The actual agent execution and LangSmith integration is in the sibling `deepagents_harbor` package. Scripts in `libs/evals/scripts/` (such as `generate_radar.py` and `analyze.py`) import from both packages.

## Related Docs

- [`radar.md`](radar.md) — `ModelResult` dataclass, chart generation functions, and file format details
