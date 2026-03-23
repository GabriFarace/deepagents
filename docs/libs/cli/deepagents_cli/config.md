# `config.py`

## High-Level Purpose

This module provides configuration, constants, and shared singletons for the CLI. It is the central source for:

- Bootstrap logic (dotenv loading, `LANGSMITH_PROJECT` override)
- A lazy-initialized `Settings` object loaded from `config.toml`
- A shared Rich `Console` instance
- Constants for input modes, shell tools, and glyphs
- Helper functions for display formatting, LangSmith project info, and agent directory management

Bootstrap is deferred until `settings` is first accessed, avoiding disk I/O on startup paths that never need configuration (e.g., `deepagents --help`).

## Key Design: Lazy Bootstrap

`settings` and `console` are module-level lazy attributes resolved via `__getattr__`. The first access triggers `_ensure_bootstrap()`, which:
1. Finds the nearest `.env` file from the user's working directory upward.
2. Loads it via `python-dotenv`.
3. Captures the original `LANGSMITH_PROJECT` value.
4. If `DEEPAGENTS_LANGSMITH_PROJECT` is set, overrides `LANGSMITH_PROJECT` for agent traces.

The bootstrap is idempotent and thread-safe (guarded by `_bootstrap_lock`).

## Module-Level Constants

| Constant | Description |
|---|---|
| `MODE_PREFIXES` | Dict mapping input mode names to their CLI prefixes (`!` for shell, `/` for command) |
| `PREFIX_TO_MODE` | Reverse mapping from prefix character to mode name |
| `MODE_DISPLAY_GLYPHS` | Unicode/ASCII glyphs for mode display |
| `SHELL_TOOL_NAMES` | Set of tool names that perform shell execution |

## Functions

### `_find_dotenv_from_start_path(start_path: Path) -> Path | None`

Traverses directories upward from `start_path` to find the nearest `.env` file.

**Parameters:**
- `start_path`: Directory to start searching from.

**Returns:** Path to the nearest `.env` file, or `None` if not found.

### `_load_dotenv(*, start_path: Path | None = None, override: bool = False) -> bool`

Loads environment variables from a `.env` file. If `start_path` is given, uses `_find_dotenv_from_start_path` to locate the file.

**Returns:** `True` if a dotenv file was loaded, `False` otherwise.

### `_ensure_bootstrap() -> None`

Runs one-time bootstrap: dotenv loading and `LANGSMITH_PROJECT` override. Thread-safe and idempotent. Exceptions are caught and logged — the CLI proceeds regardless.

### `is_ascii_mode() -> bool`

Returns `True` if the terminal should use ASCII-only characters instead of Unicode glyphs, determined from `settings.ascii_only` or terminal capability detection.

### `get_glyphs() -> Glyphs`

Returns the `Glyphs` instance appropriate for the current terminal mode (unicode or ASCII).

### `get_banner() -> str`

Returns the ASCII art banner for the welcome screen.

### `get_default_coding_instructions() -> str`

Returns the default system prompt addendum for coding tasks.

### `settings`

Lazy singleton `Settings` object loaded from `~/.deepagents/config.toml`. Provides:
- `user_deepagents_dir` — Path to `~/.deepagents/`
- `ascii_only` — Whether to use ASCII-only output
- Various display and behavior settings

### `console`

Lazy singleton `rich.console.Console` instance for formatted terminal output. Created on first access.

## Classes

### `Glyphs`

A dataclass containing unicode or ASCII glyphs used throughout the CLI:
- `checkmark` — `✓` or `x`
- `cross` — `✗` or `x`
- `arrow` — `→` or `->`
- `spinner_frames` — tuple of spinner animation frames
- And other display glyphs

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `dotenv` | `python-dotenv` | `.env` file loading |
| `tomllib` | stdlib (3.11+) | Config TOML parsing |
| `rich.console.Console` | `rich` | Terminal output formatting |
| `_version.__version__` | local | CLI version string |
