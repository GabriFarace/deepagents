# `widgets/theme_selector.py`

## High-Level Purpose

This module defines the `ThemeSelectorScreen` — an interactive modal for the `/theme` slash command. It lists all available themes, provides live preview on navigation, and persists the user's selection.

## Classes

### `ThemeSelectorScreen`

**Inherits from:** `textual.screen.ModalScreen[str | None]`

A modal dialog for theme selection with live preview. Navigating the option list applies a live preview by swapping the app theme. Returns the selected theme name on Enter, or `None` on Esc (restoring the original theme).

**Bindings:**
| Key | Action | Description |
|---|---|---|
| `escape` | `cancel` | Cancel and restore original theme |

**Constructor:**
```python
ThemeSelectorScreen(current_theme: str)
```

**Parameters:**
- `current_theme`: The currently active theme name (to highlight in the list).

**Layout (CSS):**
- Centered modal with `width: 50`, `max-width: 90%`
- `OptionList` widget for theme selection (max height 16 rows)
- Help text below the list

**Behavior:**
1. Builds an `OptionList` from `theme.ThemeEntry.REGISTRY`.
2. Pre-selects the current theme.
3. On `OptionList.OptionHighlighted`, applies the theme as a live preview via `app.theme`.
4. On `escape`, restores the original theme and dismisses with `None`.
5. On `Enter`, dismisses with the selected theme name.

**Methods:**

- `on_mount() -> None` — Builds the option list and pre-selects current theme.
- `on_option_list_option_highlighted(event) -> None` — Applies live preview of the highlighted theme.
- `action_cancel() -> None` — Restores original theme and dismisses with `None`.

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `textual.screen.ModalScreen` | textual | Modal dialog base |
| `textual.widgets.OptionList` | textual | Theme option list |
| `theme.ThemeEntry.REGISTRY` | `deepagents_cli.theme` | Available themes |
| `get_glyphs`, `is_ascii_mode` | `deepagents_cli.config` | Glyph selection |
