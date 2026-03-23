# `widgets/status.py`

## High-Level Purpose

This module defines the `StatusBar` widget — the single-row bar docked to the bottom of the CLI. It displays contextual information about the current session: input mode, auto-approve state, working directory, git branch, token count, and the active model name.

## Classes

### `ModelLabel`

**Inherits from:** `textual.widget.Widget`

A label that displays the current model name with smart truncation. When the full `provider:model` text doesn't fit in the available width:
1. Drops the provider prefix first (shows just `model`).
2. If the model name still doesn't fit, left-truncates it with a leading ellipsis (`…`) so the most distinctive tail remains visible.

**Reactive Attributes:**
| Attribute | Type | Description |
|---|---|---|
| `provider` | `reactive[str]` | Provider name (e.g., `'anthropic'`) |
| `model` | `reactive[str]` | Model name (e.g., `'claude-sonnet-4-6'`) |

**Methods:**
- `get_content_width(container, viewport) -> int` — Returns intrinsic width so `width: auto` works.
- `render() -> RenderResult` — Renders the model label with width-aware truncation.

### `StatusBar`

**Inherits from:** `textual.containers.Horizontal`

The bottom status bar. Docked to the bottom of the app with height 1.

**Sections displayed (left to right):**
1. Mode badge (hidden in normal mode, colored in shell/command modes)
2. Auto-approve indicator (shown when auto-approve is enabled)
3. Working directory (truncated)
4. Git branch (if in a git repo)
5. Token count
6. Model name (`ModelLabel`)

**Reactive Attributes:**
| Attribute | Type | Description |
|---|---|---|
| `mode` | `reactive[str]` | Current input mode |
| `auto_approve` | `reactive[bool]` | Auto-approve state |
| `cwd` | `reactive[str]` | Current working directory |
| `git_branch` | `reactive[str \| None]` | Active git branch |
| `token_count` | `reactive[int \| None]` | Context token count |
| `spinner_status` | `reactive[SpinnerStatus]` | Spinner state |

**Key Methods:**
- `set_model(provider: str, model: str) -> None` — Updates the model display.
- `set_token_count(n: int | None) -> None` — Updates or hides the token count.
- `update_git_branch() -> None` — Refreshes the git branch display asynchronously.

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `textual.reactive.reactive` | textual | Reactive attributes for auto-update |
| `textual.containers.Horizontal` | textual | Base horizontal layout |
| `get_glyphs` | `deepagents_cli.config` | Unicode/ASCII glyph selection |
