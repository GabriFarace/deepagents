# `deepagents_evals/radar.py`

## High-Level Purpose

Generates radar (spider) charts comparing multiple LLM models across eval categories. Each axis of the radar chart represents an evaluation category (e.g., `file_operations`, `memory`, `hitl`), and the radial position encodes the correctness score on a `[0, 1]` scale.

## Module-Level Data

### `EVAL_CATEGORIES: list[str]`
The canonical list of evaluation category names, loaded from `categories.json`. Order determines clockwise axis placement on the chart.

### `CATEGORY_LABELS: dict[str, str]`
Human-readable display labels for each category axis, keyed by category name.

### `_COLORS: list[str]`
Eight visually distinct hex colors cycled across models on the chart.

## Classes

### `ModelResult`

**Purpose:** Immutable data container holding a model's eval scores.

**Definition:** `@dataclass(frozen=True)`

**Attributes:**

| Attribute | Type | Description |
|-----------|------|-------------|
| `model` | `str` | Model identifier, e.g. `"anthropic:claude-sonnet-4-6"` |
| `scores` | `dict[str, float]` | Category name → correctness score in `[0, 1]` |

## Functions

### `generate_radar(results, *, categories, title, output, figsize, _color_offset) -> Figure`

**Purpose:** Generate a single radar chart comparing multiple models.

**Parameters:**
- `results: list[ModelResult]` — Models to plot.
- `categories: list[str] | None` — Categories to include (defaults to `EVAL_CATEGORIES`).
- `title: str` — Chart title (default `"Eval Results"`).
- `output: str | Path | None` — If provided, save the chart to this path (PNG/SVG/PDF).
- `figsize: tuple[float, float]` — Figure size in inches (default `(10, 10)`).
- `_color_offset: int` — Internal offset into `_COLORS` (used by `generate_individual_radars`).

**Return Value:** The matplotlib `Figure` object.

**Key Logic:**
- Computes evenly spaced angles for each category axis, starting from the top (12 o'clock) going clockwise.
- For each model, extracts scores in category order (missing categories default to `0.0`), closes the polygon by repeating the first value, and plots as a filled polygon.
- Annotates each axis point with a percentage label.
- Saves to disk if `output` is provided, closing the figure afterward.

---

### `generate_individual_radars(results, *, categories, output_dir, title_prefix, figsize) -> list[Path]`

**Purpose:** Generate one radar chart per model and save each as a PNG.

**Parameters:**
- `results: list[ModelResult]` — Models to generate charts for.
- `categories: list[str] | None` — Category axes.
- `output_dir: str | Path` — Output directory (default `"charts/individual"`).
- `title_prefix: str` — Prefix for each chart title.
- `figsize: tuple[float, float]` — Figure size.

**Return Value:** List of `Path` objects for the saved PNG files.

---

### `load_results_from_summary(path: str | Path) -> list[ModelResult]`

**Purpose:** Load `ModelResult` objects from an `evals_summary.json` file.

**Parameters:**
- `path`: Path to a JSON file containing an array of objects with `"model"` and `"category_scores"` keys.

**Return Value:** List of `ModelResult` objects.

**Raises:** `FileNotFoundError`, `json.JSONDecodeError`, `ValueError`, `KeyError` on bad input.

---

### `toy_data() -> list[ModelResult]`

**Purpose:** Return a hard-coded set of four model results (Claude Sonnet/Opus, GPT-4.1, Gemini 2.5 Pro) with plausible scores for experimentation and testing.

---

### `_safe_filename(model: str) -> str`

**Purpose:** Convert a model identifier to a filesystem-safe filename stem (replaces `:`, `/`, and spaces with hyphens).

---

### `_short_model_name(model: str) -> str`

**Purpose:** Strip the `provider:` prefix from a model identifier and truncate to 30 characters for legend labels.

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| `matplotlib.pyplot` | `matplotlib` | Plotting (imported lazily inside `generate_radar`) |
| `json`, `math`, `pathlib` | stdlib | Data loading and geometry |
| `categories.json` | package data | Source of `EVAL_CATEGORIES` and `CATEGORY_LABELS` |
