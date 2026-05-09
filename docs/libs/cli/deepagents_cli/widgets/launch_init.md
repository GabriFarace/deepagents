# `libs/cli/deepagents_cli/widgets/launch_init.py`

> Onboarding screens for the interactive CLI.

## Position in the system

This widget sits on the Textual side of the CLI. `app.py` composes these screens and controls around streamed events from the LangGraph server, while the widget itself owns presentation, local interaction state, and the small event messages it emits upward.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from textual.app import ScreenStackError`

- `from textual.binding import Binding, BindingType`

- `from textual.containers import Vertical`

- `from textual.content import Content`

- `from textual.screen import ModalScreen`

- `from textual.widgets import Input, Static`

- `from deepagents_cli import theme`

- `from deepagents_cli.config import get_glyphs, is_ascii_mode`

- `from deepagents_cli.extras_info import MODEL_PROVIDER_EXTRAS, SANDBOX_EXTRAS`


## Functions and classes

### `_normalize_name(value: str)`

Normalize submitted onboarding names for display.

Additional notes from the source docstring:

```text
Args:
    value: Raw submitted name.

Returns:
    The stripped name, title-cased when it was entered in lowercase.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `LaunchNameScreen`

First-step onboarding screen that asks for the user's name.

Additional notes from the source docstring:

```text
Dismissal values:

- Non-empty stripped/title-cased name when the user submits one.
- `""` when the user submits an empty input (continue, but skip name memory).
- `None` when the user dismisses with Escape (skip remaining onboarding).
```

Methods worth reading inside this class:

- `compose(self)`: Compose the name-entry screen.

- `on_mount(self)`: Apply ASCII border when needed.

- `on_input_submitted(self, event: Input.Submitted)`: Dismiss with the submitted name.

- `action_skip(self)`: Skip the onboarding sequence.

- `action_cancel(self)`: Alias for `action_skip` invoked by the global Esc binding.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `LaunchDependenciesScreen`

Onboarding screen that summarizes installed optional integrations.

Methods worth reading inside this class:

- `compose(self)`: Compose the dependency summary screen.

- `on_mount(self)`: Apply ASCII border when needed.

- `_format_section(self, *, title: str, ready: bool)`: Format one status section.

- `_extra_names(self, names: frozenset[str], *, ready: bool)`: Return sorted extra names matching a category and readiness state.

- `action_continue(self)`: Continue onboarding.

- `action_skip(self)`: Skip the remaining onboarding sequence.

- `action_cancel(self)`: See `LaunchNameScreen.action_cancel`.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `_format_extra_names(names: list[str])`

Format extra names for compact display.

Additional notes from the source docstring:

```text
Args:
    names: Extra names to display.

Returns:
    Comma-separated extra names, or a placeholder when empty.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Widget classes are coupled to Textual message names, CSS classes, and screen lifecycle hooks. Changing identifiers here can break bindings in `app.py` or stylesheet selectors even when type checks pass.
