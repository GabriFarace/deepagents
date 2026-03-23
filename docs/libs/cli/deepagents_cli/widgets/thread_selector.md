# `widgets/thread_selector.py`

## High-Level Purpose

This module defines the `ThreadSelectorScreen` — an interactive modal for the `/threads` slash command. It lists conversation threads with metadata columns, supports fuzzy search, allows deleting threads, and optionally opens the selected thread in LangSmith.

## Module-Level State

| Constant | Description |
|---|---|
| `_URL_FETCH_TIMEOUT` | `2.0s` — timeout for LangSmith thread-URL resolution |
| `_column_widths_cache` | Module-level cache for computed column widths (skips recalculation when data/config hasn't changed) |

## Column Configuration

Default columns and widths:

| Column | Default Width | Description |
|---|---|---|
| `thread_id` | 10 chars | Truncated UUID |
| `agent_name` | 12 chars | Agent name |
| `messages` | 4 chars | Message count |
| `created_at` | auto | Creation timestamp |
| `updated_at` | auto | Last update timestamp |
| `git_branch` | 16 chars | Git branch |
| `cwd` | auto | Working directory |
| `initial_prompt` | auto | First user message |

## Classes

### `ThreadOption`

**Inherits from:** `textual.widgets.Static`

A clickable thread row in the selector list. Displays all visible columns.

**Inner Message: `Clicked`**

Posted when a thread option is clicked. Contains `thread_id` and `thread_index`.

### `ThreadSelectorScreen`

**Inherits from:** `textual.screen.ModalScreen[str | None]`

Interactive thread selection modal.

**Bindings:**
| Key | Action | Description |
|---|---|---|
| `escape` | `cancel` | Close without selecting |
| `up / k` | `move_up` | Navigate up |
| `down / j` | `move_down` | Navigate down |
| `enter` | `select` | Open selected thread |
| `d / delete` | `delete_thread` | Delete selected thread |
| `o` | `open_in_langsmith` | Open thread in LangSmith |
| `v` | `toggle_verbose` | Toggle verbose column display |

**Constructor:**

```python
ThreadSelectorScreen(
    threads: list[ThreadInfo],
    *,
    verbose: bool = False,
    relative_time: bool = False
)
```

**Parameters:**
- `threads`: Pre-loaded list of `ThreadInfo` dicts to display.
- `verbose`: Whether to show all columns by default.
- `relative_time`: Whether to show timestamps as relative time.

**Behavior:**
1. Displays threads sorted by last update (most recent first).
2. Fuzzy search via `textual.fuzzy.Matcher` filters rows in real time.
3. A `Checkbox` column is available for multi-select (delete confirmation).
4. Column widths are computed once and cached.
5. Returns the selected `thread_id` string or `None` if cancelled.

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `textual.screen.ModalScreen` | textual | Modal dialog base |
| `textual.fuzzy.Matcher` | textual | Fuzzy search |
| `textual.widgets.Checkbox, Input` | textual | Selection and search widgets |
| `rich.cells.cell_len` | rich | Accurate column width calculation |
| `ThreadInfo` | `deepagents_cli.sessions` | Thread metadata type |
| `build_langsmith_thread_url` | `deepagents_cli.config` | LangSmith URL builder |
