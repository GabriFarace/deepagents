# `widgets/chat_input.py`

## High-Level Purpose

This module defines the `ChatInput` widget — the primary text input area for the CLI. It provides:

- Multi-line text input using Textual's `TextArea`
- Slash-command autocompletion
- `@file` mention autocompletion (fuzzy file search)
- Input history (arrow key navigation)
- Input mode switching (normal, shell `!`, command `/`)
- Paste handling with media path detection
- `shift+enter` insertion of newlines
- VSCode terminal `shift+enter` emulation (backslash + enter)

## Module-Level Constants

| Constant | Value | Description |
|---|---|---|
| `_PASTE_BURST_CHAR_GAP_SECONDS` | `0.03` | Max time between chars for paste-burst detection |
| `_PASTE_BURST_FLUSH_DELAY_SECONDS` | `0.08` | Idle timeout before flushing paste-burst buffer |
| `_PASTE_BURST_START_CHARS` | `{"'", '"'}` | Characters that can start dropped-path payloads |
| `_BACKSLASH_ENTER_GAP_SECONDS` | `0.15` | Max gap for detecting terminal shift+enter (backslash + enter) |

## Classes

### `CompletionOption`

**Inherits from:** `textual.widgets.Static`

A clickable completion option displayed in the autocomplete popup. Highlighted when selected.

### `ChatInput`

**Inherits from:** `textual.containers.Vertical`

The main chat input container. Composes a `TextArea` (multi-line input) with an autocomplete popup and a mode indicator.

**Key Reactive Attributes:**
| Attribute | Type | Description |
|---|---|---|
| `mode` | `str` | Current input mode: `'normal'`, `'shell'`, `'command'` |
| `is_busy` | `bool` | Whether the agent is currently processing |

**Inner Messages:**
- `Submitted(text: str, mode: str)` — Posted when the user submits input (Enter key).

**Key Methods:**

- `get_text() -> str` — Returns current text content.
- `set_text(text: str) -> None` — Replaces current text content.
- `clear() -> None` — Clears the input.
- `focus_input() -> None` — Gives focus to the underlying `TextArea`.
- `on_key(event)` — Handles Enter (submit), shift+enter (newline), Up/Down (history), Tab (autocomplete).
- `on_paste(event)` — Handles paste events, detecting media paths.

**Mode Switching:**

The input mode is determined by the first character of the input:
- `!` → shell mode (commands run in the local shell)
- `/` → command mode (slash commands like `/model`, `/threads`)
- Everything else → normal mode (sent to the AI agent)

**Autocomplete:**

Uses `MultiCompletionManager` with two controllers:
- `SlashCommandController` — completes `/command` names on `/` trigger
- `FuzzyFileController` — completes `@filename` mentions on `@` trigger

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `textual.widgets.TextArea` | textual | Multi-line input |
| `SLASH_COMMANDS` | `deepagents_cli.command_registry` | Available slash commands |
| `MODE_PREFIXES`, `PREFIX_TO_MODE` | `deepagents_cli.config` | Mode prefix constants |
| `IMAGE_PLACEHOLDER_PATTERN`, `VIDEO_PLACEHOLDER_PATTERN` | `deepagents_cli.input` | Media detection |
| `MultiCompletionManager`, `FuzzyFileController`, `SlashCommandController` | `widgets.autocomplete` | Autocomplete controllers |
| `HistoryManager` | `widgets.history` | Input history persistence |
