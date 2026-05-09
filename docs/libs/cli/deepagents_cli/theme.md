# `libs/cli/deepagents_cli/theme.py`

> LangChain brand colors and semantic constants for the CLI.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Functions and classes

### `ThemeColors`

Complete set of semantic colors for one theme variant.

Additional notes from the source docstring:

```text
Every field must be a 7-character hex color string (e.g., `'#7AA2F7'`).
```

Methods worth reading inside this class:

- `__post_init__(self)`: Validate that every field is a valid hex color.

- `merged(cls, base: ThemeColors, overrides: dict[str, str])`: Create a new `ThemeColors` by overlaying overrides onto a base.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `ThemeEntry`

Metadata for a registered theme.

Methods worth reading inside this class:

- `__post_init__(self)`: Validate that the label is a non-empty string.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `_builtin_themes()`

Return the built-in theme entries as a mutable dict.

Additional notes from the source docstring:

```text
Textual built-ins are discovered from `textual.theme.BUILTIN_THEMES` so
newly shipped Textual themes appear automatically. They are not registered
via `register_theme()` — Textual's own `$primary`, `$background`, etc.
apply. The `colors` field provides fallback values for app-specific CSS
vars (`$mode-bash`, `$mode-command`) and Python-side styling. For standard
properties (primary, secondary, etc.), `get_theme_colors()` dynamically
resolves from the actual Textual theme at runtime so the Python and CSS
color systems stay in sync.

Returns:
    Dict of built-in theme names to `ThemeEntry` instances.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_builtin_names()`

Names of built-in themes; lazily computed and cached.

Additional notes from the source docstring:

```text
User `[themes.<name>]` sections matching a built-in name override its colors
rather than creating a new theme. Derived from `_builtin_themes()` so the
set stays in sync automatically. Lazy because `_builtin_themes()` imports
`textual.theme.BUILTIN_THEMES`, and we don't want to pull Textual onto the
`deepagents --help` / `deepagents -v` cold-start path.

Returns:
    Frozen set of built-in theme names.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_load_user_themes(builtins: dict[str, ThemeEntry], *, config_path: Path | None=None)`

Load user-defined themes from `config.toml` into `builtins` (mutated).

Additional notes from the source docstring:

```text
**New themes** — each `[themes.<name>]` section (where `<name>` is not a
built-in) must have:

- `label` (str) — human-readable name shown in the theme picker.
- `dark` (bool, optional) — whether this is a dark-mode variant.

    Defaults to `False` (light).

**Built-in overrides** — if `<name>` matches a built-in theme, only color
fields are read; `label` and `dark` are inherited from the built-in.

All `ThemeColors` fields are optional. For new themes, omitted fields
fall back to the built-in dark or light palette based on the `dark` flag.

For built-in overrides, omitted fields retain the existing built-in colors.

Invalid themes (bad hex, missing required keys) are logged as warnings
and skipped — they never crash startup.

Example `config.toml` snippet:

```toml
# New custom theme
[themes.my-solarized]
label = "My Solarized"
dark = true
primary = "#268BD2"
warning = "#B58900"

# Override built-in theme colors
[themes.langchain]
primary = "#FF5500"
```

Args:
    builtins: Mutable dict to update (new themes are added, built-in
        overrides replace existing entries).
    config_path: Override for the config file path (testing).
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_build_registry(*, config_path: Path | None=None)`

Build and freeze the theme registry (built-in + user themes).

Additional notes from the source docstring:

```text
Args:
    config_path: Override for the config file path (testing).

Returns:
    Read-only mapping of theme names to `ThemeEntry` instances.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `get_registry()`

Return the read-only theme registry, building it on first access.

Additional notes from the source docstring:

```text
Lazy so that `theme.py` can be imported on the `deepagents --help` cold
path without pulling in `textual.theme.BUILTIN_THEMES` (which transitively
imports Textual, ~470ms). Inside a Textual app, the build is microseconds
because Textual is already loaded; callers like `_register_custom_themes()`
iterate the result during `App.__init__`, which warms the cache before any
user-facing surface (e.g. the theme picker) reads it.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `reload_registry()`

Rebuild the theme registry from disk.

Additional notes from the source docstring:

```text
Re-reads `~/.deepagents/config.toml` for user-defined themes so that
`/reload` can pick up config changes without restarting the app.

Returns:
    The new frozen registry.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `get_css_variable_defaults(*, dark: bool=True, colors: ThemeColors | None=None)`

Return custom CSS variable defaults for the given mode.

Additional notes from the source docstring:

```text
Most styling is handled by Textual's built-in CSS variables (`$primary`,
`$text-muted`, `$error-muted`, etc.).  This function only returns
app-specific semantic variables that have no Textual equivalent.

Args:
    dark: Selects `DARK_COLORS` or `LIGHT_COLORS` when `colors` is None.
    colors: Explicit color set to use. Takes precedence over `dark`.

Returns:
    Dict of CSS variable names to hex color values.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_resolve_app(widget_or_app: object)`

Resolve a widget or App to the App instance.

Additional notes from the source docstring:

```text
Args:
    widget_or_app: Textual `App` or a mounted widget.

Returns:
    The resolved App instance.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_colors_from_textual_theme(app: object)`

Construct `ThemeColors` from the app's active Textual theme.

Additional notes from the source docstring:

```text
Reads standard properties (primary, secondary, etc.) from the resolved
theme so Python-side styling matches CSS.  `muted` falls back to the
dark/light base unconditionally (no Textual equivalent).
`mode_bash` is derived from the theme's `error` color, and `mode_command`
from `secondary`, falling back to the base palette when non-hex.

Non-hex values (e.g. `ansi_blue` in the ANSI theme) are detected and fall
back to the base palette automatically.

Args:
    app: The Textual App instance.

Returns:
    `ThemeColors` derived from the active theme.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `get_theme_colors(widget_or_app: App | object | None=None)`

Return the `ThemeColors` for the active Textual theme.

Additional notes from the source docstring:

```text
For custom themes (LangChain-branded and user-defined), the pre-built
`ThemeColors` from the registry is returned directly.  For Textual built-in
themes, colors are resolved dynamically from the actual theme properties so
Python-side styling stays in sync with CSS variables.

Textual widget code should call this instead of reading the module-level
ANSI constants, which are intended for Rich console output only.

Args:
    widget_or_app: Textual `App`, a mounted widget, or `None`.

Returns:
    `ThemeColors` for the active theme.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
