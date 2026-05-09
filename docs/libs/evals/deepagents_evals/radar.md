# `evals/deepagents_evals/radar.py`

> Radar chart generation for eval results.

## Position in the system

This module belongs to the evaluation CLI/reporting layer. It reads pytest/Harbor outputs, aggregates results, and renders human-facing summaries or charts.

## Imports and module-level state

This file imports `__future__, importlib.util, json, math, dataclasses, functools, pathlib, typing`.
Module constants worth noticing: `_CATEGORIES_JSON`, `ALL_CATEGORIES`, `EVAL_CATEGORIES`, `CATEGORY_LABELS`, `_LIGHT`, `_DARK`, `_THEMES`, `THEMES`, `_MODELS_REGISTRY_PATH`.

## Functions and classes

### `generate_radar(results: list[ModelResult], *, categories: list[str] | None=None, title: str='Eval Results', output: str | Path | None=None, figsize: tuple[float, float]=(10, 10), theme: str='light', _color_offset: int=0)`

Generate a radar chart comparing models across eval categories. Args: results: One `ModelResult` per model to plot. categories: Category axes to include. Defaults to `EVAL_CATEGORIES`. title: Chart title. output: If provided, save the figure to this path (PNG/SVG/PDF). figsize: Figure size in inches. theme: Color scheme — `"light"` or `"dark"`. Unrecognized values fall back to `"light"`. Returns: The matplotlib `Figure` object. Key arguments are `results`, `categories`, `title`, `output`, `figsize`, `theme`, `_color_offset`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `len`, `append`, `subplots`, `cast`, `set_facecolor`, `set_theta_offset`. It runs synchronously in the caller and returns directly.

### `generate_individual_radars(results: list[ModelResult], *, categories: list[str] | None=None, output_dir: str | Path='charts/individual', title_prefix: str='Eval Results', figsize: tuple[float, float]=(10, 10), theme: str='light')`

Generate one radar chart per model. Each chart is saved as `<sanitized_model_name>.png` inside `output_dir`. Args: results: One `ModelResult` per model. categories: Category axes to include. Defaults to `EVAL_CATEGORIES`. output_dir: Directory to write per-model PNGs. title_prefix: Prefix for each chart title (model name is appended). figsize: Figure size in inches. theme: Color scheme — `"light"` or `"dark"`. Unrecognized values fall back to `"light"`. Returns: List of paths to the saved PNG files. Key arguments are `results`, `categories`, `output_dir`, `title_prefix`, `figsize`, `theme`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `Path`, `mkdir`, `enumerate`, `generate_radar`, `append`. It runs synchronously in the caller and returns directly.

### `load_results_from_summary(path: str | Path)`

Load model results from an `evals_summary.json` file. The summary file is a JSON array of objects. Each object must have a `category_scores` dict mapping category names to `[0, 1]` correctness floats. The `model` key defaults to `"unknown"` if absent. Args: path: Path to `evals_summary.json`. Returns: List of `ModelResult` objects. Raises: FileNotFoundError: If `path` does not exist. json.JSONDecodeError: If the file contains invalid JSON. ValueError: If a score value in `category_scores` is not numeric. KeyError: If an entry is missing `category_scores`. Key arguments are `path`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `loads`, `read_text`, `str`, `append`, `get`, `float`. It runs synchronously in the caller and returns directly.

### `toy_data()`

Generate toy eval data for experimentation. Returns: List of `ModelResult` with plausible scores across all categories. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `ModelResult`. It runs synchronously in the caller and returns directly.

### `ModelResult`

Eval scores for a single model across categories. Attributes: model: Model identifier (e.g. `anthropic:claude-sonnet-4-6`). scores: Mapping of category name to correctness score in `[0, 1]`. This class inherits from `object` and is the main object for this part of the module.

## Gotchas

Most behavior is delegated through framework protocols, so preserve method signatures when editing even if an argument appears unused locally.
