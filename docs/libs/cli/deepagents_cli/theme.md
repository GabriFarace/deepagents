# `theme.py`

## High-Level Purpose

This module is the single source of truth for LangChain brand colors and semantic constants used throughout the CLI. It defines:

- The complete color palette for both dark and light themes
- `ThemeColors` dataclass for semantic color aliases
- `ThemeEntry` class with a registry of all available themes (built-in Textual themes, LangChain-branded themes, and user-defined themes)
- **Auto-discovery of Textual built-in themes** so newly shipped Textual themes appear in the picker without code changes
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

Semantic color aliases for a theme. Named attributes include:
- `primary` — primary accent color
- `success`, `error`, `warning` — semantic status colors
- `mode_bash`, `mode_command` — colors for input mode badges
- `diff_added_bg`, `diff_removed_bg` — diff display colors
- `muted` — secondary/muted text color

### `ThemeEntry`

Describes a complete Textual theme.

| Attribute | Type | Description |
|---|---|---|
| `label` | `str` | Human-readable label shown in the theme picker |
| `dark` | `bool` | Whether this is a dark theme |
| `colors` | `ThemeColors` | Semantic color palette |
| `custom` | `bool` | `False` for Textual built-ins (not registered via `register_theme()`); `True` for LangChain-branded and user-defined themes |
| `REGISTRY` | `ClassVar[dict[str, ThemeEntry]]` | All available themes by name |

## Module-Level Constants

| Constant | Description |
|---|---|
| `DEFAULT_THEME` | `"langchain"` — default dark theme name |
| `_TEXTUAL_THEME_LABELS` | Curated human-readable labels for Textual built-in themes |

### `_TEXTUAL_THEME_LABELS`

Maps Textual theme slugs to human-readable display labels for the theme picker. Themes shipped by Textual that are not in this mapping fall back to a humanized slug (e.g., `"rose-pine-moon"` → `"Rose Pine Moon"`).

Includes labels for: `textual-dark`, `textual-light`, `ansi-dark`, `ansi-light`, `catppuccin-frappe`, `rose-pine`, `rose-pine-dawn`, `rose-pine-moon`, `tokyo-night`.

## Functions

### `_builtin_themes() -> dict[str, ThemeEntry]`

**New in this version.** Returns all Textual built-in themes as `ThemeEntry` objects.

**Auto-discovery behavior:**
- Reads `textual.theme.BUILTIN_THEMES` at call time, so newly shipped Textual themes (e.g., a new `rose-pine-*` variant) appear automatically in the theme picker without any code change.
- Built-in themes are **not registered** via `register_theme()` — Textual's own CSS variables apply.
- The `colors` field provides fallback values for app-specific CSS variables.
- Labels are resolved from `_TEXTUAL_THEME_LABELS`; unlisted themes get a humanized slug label.

### `_builtin_names() -> frozenset[str]`

Lazily computed and cached set of built-in Textual theme names. Used by `_load_user_themes()` to detect if a `[themes.<name>]` config section overrides a built-in (vs. creating a new theme).

### `get_theme_colors(widget_or_app=None) -> ThemeColors`

Returns the `ThemeColors` for the currently active theme.

**Parameters:**
- `widget_or_app`: Optional Textual widget or `App` instance for theme-aware lookup. If `None`, returns colors for the default theme.

### `get_css_variable_defaults(*, dark: bool) -> dict[str, str]`

Returns the custom CSS variable defaults for a theme (dark or light). Used by `App.get_theme_variable_defaults()` to expose `$mode-bash` and `$mode-command` CSS variables.

### `_load_user_themes(builtins, *, config_path=None) -> None`

Loads user-defined themes from `~/.deepagents/config.toml` under `[themes.<name>]` sections.

**Behavior:**
- **New theme:** Must have a `label` (str). `dark` defaults to `False` (light). All color fields are optional and fall back to the dark/light base palette.
- **Built-in override:** Matches an existing built-in by name. Only color fields are read; `label` and `dark` are inherited from the built-in.
- Invalid themes are logged as warnings and skipped (startup never crashes on bad theme config).

**Example `~/.deepagents/config.toml`:**
```toml
[themes.my-solarized]
label = "My Solarized"
dark = true
primary = "#268BD2"

# Override a built-in's colors
[themes.tokyo-night]
primary = "#FF5500"
```

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `textual.theme.Theme` | textual | Textual theme registration |
| `textual.theme.BUILTIN_THEMES` | textual | Auto-discovery of built-in themes |
| `textual.app.App` | textual | App type hint for theme lookup |
| `dataclasses` | stdlib | `ThemeColors` and `ThemeEntry` definitions |
| `re` | stdlib | Color validation |
