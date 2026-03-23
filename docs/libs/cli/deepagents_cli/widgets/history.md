# `widgets/history.py`

## High-Level Purpose

This module provides `HistoryManager` — a command history manager with file persistence for the chat input. It supports navigating previous inputs with the up/down arrow keys, prefix-based history search, and concurrent-safe append-only writes to a JSON-lines file.

## Classes

### `HistoryManager`

Manages command history with file persistence.

**Design decisions:**
- Uses **append-only writes** for concurrent safety — multiple agents can write to the same history file without corruption.
- File compaction only happens when entries exceed 2x `max_entries` to minimize rewrites.
- History is stored as JSON-lines (one JSON string per line).

**Constructor:**

```python
HistoryManager(history_file: Path, max_entries: int = 100)
```

**Parameters:**
- `history_file`: Path to the JSON-lines history file (default: `~/.deepagents/history.jsonl`).
- `max_entries`: Maximum number of entries to keep.

**Key Attributes:**
| Attribute | Description |
|---|---|
| `history_file` | Path to the history file |
| `max_entries` | Max entries to retain |
| `_entries` | In-memory list of history entries (most recent last) |
| `_current_index` | Current navigation position (-1 means at the prompt) |
| `_temp_input` | Temporarily saved current input during history navigation |
| `_query` | Current search prefix (for prefix-based search) |

**Methods:**

#### `add(text: str) -> None`

Appends a new entry to history (both in memory and file). Avoids duplicate consecutive entries. Triggers compaction if `len(entries) > 2 * max_entries`.

#### `navigate_up(current_text: str) -> str | None`

Navigates backward in history (toward older entries). If `current_text` is non-empty, saves it as `_temp_input` for the down-navigation to restore.

**Returns:** Previous history entry, or `None` if at the beginning.

#### `navigate_down() -> str | None`

Navigates forward in history (toward newer entries). Restores `_temp_input` when reaching the present.

**Returns:** Next history entry or the saved `_temp_input`, or `None` if already at the current input.

#### `reset() -> None`

Resets navigation position to the current prompt (index -1).

#### `_load_history() -> None`

Reads history from the JSON-lines file into `_entries`. Handles both plain strings and JSON-encoded strings.

#### `_append_to_file(text: str) -> None`

Appends a single entry to the file (concurrent-safe). Creates parent directories if needed.

#### `_compact_history() -> None`

Rewrites the history file to remove entries beyond `max_entries`. Only called when exceeding 2x the limit.

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `json` | stdlib | JSON-lines encoding |
| `pathlib.Path` | stdlib | File path handling |
