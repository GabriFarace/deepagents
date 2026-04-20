# `widgets/agent_selector.py`

## High-Level Purpose

Provides `AgentSelectorScreen`, a modal dialog that lets the user browse and hot-swap between the agents installed in `~/.deepagents/` without restarting the CLI. It is activated by the `/agents` slash command.

Selecting a different agent triggers a three-phase switch in `DeepAgentsApp`:
1. **UI teardown** — running tasks are guarded against re-entry.
2. **Server restart** — the backing `langgraph` subprocess is restarted with the new `assistant_id`.
3. **State reset** — the thread is reset, skills are refreshed, and the recent agent is persisted to `[agents].recent` in `config.toml`.

The choice is also persisted so subsequent bare `deepagents` launches resume the last-used agent.

## Classes

### `AgentSelectorScreen`

**Inherits from:** `ModalScreen[str | None]`

Displays an `OptionList` of all real subdirectories in `~/.deepagents/` (symlinks excluded). Returns the selected agent name string on `Enter`, or `None` on `Esc` (no change).

**Key Bindings:**
| Key | Action |
|---|---|
| `escape` | Cancel — dismiss with `None` |
| `tab` | Move cursor down |
| `shift+tab` | Move cursor up |

Arrow keys and `Enter` are handled natively by the embedded `OptionList`.

**Behavior:**
- The currently active agent is highlighted in the list.
- Directory names that contain Rich markup characters are safely escaped via `Content.from_markup` to prevent rendering errors.
- When no agents exist yet (empty `~/.deepagents/`), a descriptive empty-state message is shown instead of an empty list.

## Dependencies

| Import | Source | Purpose |
|---|---|---|
| `ModalScreen` | `textual.screen` | Base modal screen |
| `OptionList`, `Option` | `textual.widgets` | Scrollable list |
| `Content` | `textual.content` | Safe Rich markup handling |
| `theme`, `config` | `deepagents_cli` | Brand colors, ASCII mode detection |
