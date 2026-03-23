# `editor.py`

## High-Level Purpose

This module provides external editor support for composing prompts in the CLI. It allows users to open their preferred editor (from `$VISUAL` or `$EDITOR` environment variables) with the current chat input pre-populated in a temporary `.md` file.

Activated via the `ctrl+x` keybinding in the TUI.

## Module-Level Constants

| Constant | Type | Description |
|---|---|---|
| `GUI_WAIT_FLAG` | `dict[str, str]` | Mapping of GUI editor names to their blocking flags (e.g., `code: '--wait'`, `subl: '-w'`) |
| `VIM_EDITORS` | `set[str]` | Set of vim-family editor names that receive `-i NONE` (avoids viminfo errors in temp environments) |

**Supported GUI editors with auto-wait:**
- `code` (VS Code) — `--wait`
- `cursor` — `--wait`
- `zed` — `--wait`
- `atom` — `--wait`
- `subl` (Sublime Text) — `-w`
- `windsurf` — `--wait`

## Functions

### `resolve_editor() -> list[str] | None`

Resolves the editor command from environment variables.

**Resolution order:**
1. `$VISUAL`
2. `$EDITOR`
3. Platform default: `notepad` on Windows, `vi` otherwise

**Returns:** Tokenized command list (via `shlex.split`), or `None` if the environment variable was set but empty after tokenization.

### `_prepare_command(cmd: list[str], filepath: str) -> list[str]`

Builds the full command list with appropriate flags for the editor type.

- Auto-injects `--wait` / `-w` for known GUI editors.
- Auto-injects `-i NONE` for vim-family editors to avoid viminfo errors in temp environments.
- Appends the filepath as the last argument.

**Returns:** Complete command list ready for `subprocess.run`.

### `open_in_editor(current_text: str) -> str | None`

Opens the current chat input text in an external editor.

**Parameters:**
- `current_text`: The text to pre-populate in the editor.

**Process:**
1. Resolves the editor via `resolve_editor()`.
2. Creates a temporary `.md` file prefixed `deepagents-edit-` with `current_text`.
3. Launches the editor via `subprocess.run`, blocking until the editor closes.
4. Reads the file contents back.
5. Deletes the temporary file.

**Returns:** The edited text with normalized line endings, or `None` if:
- The editor exited with a non-zero status.
- The editor was not found.
- The result was empty or whitespace-only.

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `os`, `shlex`, `subprocess`, `tempfile`, `sys` | stdlib | Editor resolution and execution |
| `pathlib.Path` | stdlib | File path handling |
