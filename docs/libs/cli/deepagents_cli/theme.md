# `theme.py`

## High-Level Purpose

This module is the single source of truth for LangChain brand colors and semantic constants used throughout the CLI. It defines:

- The complete color palette for both dark and light themes
- `ThemeColors` dataclass for semantic color aliases
- `ThemeEntry` class with a registry of all available themes
- User-defined theme support via `config.toml`
- Utilities for registering Textual themes and retrieving colors

CSS-side styling uses Textual CSS variables set via `register_theme()`, while Python code uses the `ThemeColors` instance via `ThemeEntry.REGISTRY`.

## Dark Theme Palette

| Constant | Value | Description |
|---|---|---|
| `LC_DARK` | `"#11121D"` | Background (blue-tinted dark) |
| `LC_CARD` | `"#1A1B2E"` | Surface/card elevated above background |
| `LC_BORDER_DK` | `"#25283B"` | Borders on dark backgrounds |
| `LC_BORDER_LT` | `"#3A3E57"` | Borders on lighter/hovered surfaces |
| `LC_BODY` | `"#C0CAF5"` | High-contrast body text |
| `LC_BLUE` | `"#7AA2F7"` | Primary accent blue |
| `LC_PURPLE` | `"#BB9AF7"` | Secondary accent / badges |
| `LC_GREEN` | `"#9ECE6A"` | Success / positive indicator |
| `LC_AMBER` | `"#EB8B46"` | Warning / caution |
| `LC_PINK` | `"#F7768E"` | Error / destructive |
| `LC_ORANGE` | `"#FF9E64"` | Dev install indicator / warm accent |
| `LC_MUTED` | `"#545C7E"` | Muted/secondary text |
| `LC_GREEN_BG` | `"#1C2A38"` | Diff addition background |
| `LC_PINK_BG` | `"#2A1F32"` | Diff removal / error background |
| `LC_PANEL` | `"#25283B"` | Panel/section background |

## Light Theme Palette

| Constant | Value | Description |
|---|---|---|
| `LC_LIGHT_BG` | `"#F5F5F7"` | Warm neutral white background |
| `LC_LIGHT_SURFACE` | `"#EAEAEE"` | Card surface (slightly darker) |
| `LC_LIGHT_BORDER` | `"#C8CAD0"` | Light theme borders |
| `LC_LIGHT_BORDER_HVR` | `"#A0A4B0"` | Hovered/focused borders |

## Classes

### `ThemeColors`

**Type:** `dataclass`

Semantic color aliases for a theme. Contains named attributes like:
- `primary` — primary accent color
- `success`, `error`, `warning` — semantic status colors
- `mode_bash`, `mode_command` — colors for input mode badges
- `diff_added_bg`, `diff_removed_bg` — diff display colors
- `muted` — secondary/muted text color

### `ThemeEntry`

Describes a complete Textual theme.

| Attribute | Type | Description |
|---|---|---|
| `name` | `str` | Textual theme name (e.g., `'langchain'`) |
| `label` | `str` | Human-readable label for display |
| `dark` | `bool` | Whether this is a dark theme |
| `colors` | `ThemeColors` | Semantic color palette |
| `REGISTRY` | `ClassVar[dict[str, ThemeEntry]]` | All registered themes by name |

## Module-Level Constants

| Constant | Description |
|---|---|
| `DEFAULT_THEME` | `"langchain"` — default dark theme name |

## Functions

### `get_theme_colors(widget_or_app=None) -> ThemeColors`

Returns the `ThemeColors` for the currently active theme.

**Parameters:**
- `widget_or_app`: Optional Textual widget or `App` instance for theme-aware lookup. If `None`, returns colors for the default theme.

**Returns:** `ThemeColors` instance from the active theme.

### `get_css_variable_defaults(*, dark: bool) -> dict[str, str]`

Returns the custom CSS variable defaults for a theme (dark or light). Used by `App.get_theme_variable_defaults()` to expose `$mode-bash` and `$mode-command` CSS variables.

### `_load_user_themes(config_path: Path | None = None) -> list[ThemeEntry]`

Loads user-defined themes from `~/.deepagents/config.toml` under `[themes.<name>]` sections. Each section must have `label` (str) and `dark` (bool); color fields are optional and fall back to built-in palette.

**Returns:** List of `ThemeEntry` instances for user-defined themes.

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `textual.theme.Theme` | textual | Textual theme registration |
| `textual.app.App` | textual | App type hint for theme lookup |
| `dataclasses` | stdlib | ThemeColors and ThemeEntry definitions |
| `re` | stdlib | Color validation |
