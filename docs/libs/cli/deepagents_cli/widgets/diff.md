# `widgets/diff.py`

## High-Level Purpose

This module provides the `compose_diff_lines` generator function and related helpers for displaying unified diffs in the Textual TUI. Each diff line is rendered as a separate `Static` widget with appropriate CSS classes for syntax coloring, enabling theme-aware diff rendering that updates automatically when the user changes themes.

## Functions

### `compose_diff_lines(diff: str, max_lines: int | None = 100) -> ComposeResult`

Yields per-line `Static` widgets for a unified diff.

**Parameters:**
- `diff`: Unified diff string (from `difflib.unified_diff` or similar).
- `max_lines`: Maximum number of diff lines to show. `None` for unlimited.

**Yields:** `Static` widgets — one per diff line — with appropriate CSS classes.

If `diff` is empty, yields a single `Static` with "No changes detected" in dim style.

**CSS classes used:**
- `.diff-line-added` — Lines starting with `+` (not `+++`)
- `.diff-line-removed` — Lines starting with `-` (not `---`)
- `.diff-line-header` — Lines starting with `@@`
- `.diff-line-file` — `+++` and `---` file header lines
- No class — Context lines (unchanged)

### `_compose_diff_content(diff: str, max_lines: int | None) -> ComposeResult`

Internal implementation for non-empty diffs. Computes and prepends a stats header (`+N -M`) before yielding line widgets.

**Stats header format:** `+{additions}` in green, `-{deletions}` in pink/red.

**Line number display:** Hunk headers (`@@...@@`) include line numbers that are extracted and formatted with consistent width padding.

## Design Notes

**Theme-aware coloring:** Background colors for added/removed lines use Textual CSS variables (`.diff-line-added { background: $diff-added-bg }`) rather than inline styles, so they automatically update when the user switches themes.

**Line number extraction:** Line numbers are parsed from `@@` hunk headers using regex. Maximum line number determines the width used for consistent column alignment.

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `re` | stdlib | Hunk header line number extraction |
| `textual.widgets.Static` | textual | Per-line widget |
| `textual.content.Content` | textual | Styled content assembly |
| `theme.get_theme_colors` | `deepagents_cli.theme` | Color access |
| `get_glyphs`, `is_ascii_mode` | `deepagents_cli.config` | Glyph selection |
