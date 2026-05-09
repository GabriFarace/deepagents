# `libs/cli/deepagents_cli/widgets/notification_settings.py`

> Notification settings screen for /notifications command.

## Position in the system

This widget sits on the Textual side of the CLI. `app.py` composes these screens and controls around streamed events from the LangGraph server, while the widget itself owns presentation, local interaction state, and the small event messages it emits upward.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from textual.binding import Binding, BindingType`

- `from textual.containers import VerticalGroup`

- `from textual.screen import ModalScreen`

- `from textual.widgets import Checkbox, Static`

- `from deepagents_cli import theme`

- `from deepagents_cli.config import get_glyphs, is_ascii_mode`


## Functions and classes

### `NotificationSettingsScreen`

Modal dialog for managing startup warning preferences.

Additional notes from the source docstring:

```text
Each checkbox maps to a key in `[warnings].suppress` in
`~/.deepagents/config.toml`. Toggling a checkbox immediately
persists the change.
```

Methods worth reading inside this class:

- `compose(self)`: Compose the screen layout.

- `on_mount(self)`: Apply ASCII border if needed.

- `on_checkbox_changed(self, event: Checkbox.Changed)`: Persist warning suppression toggle to config.toml on change.

- `action_cancel(self)`: Close the screen.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

## Gotchas

Widget classes are coupled to Textual message names, CSS classes, and screen lifecycle hooks. Changing identifiers here can break bindings in `app.py` or stylesheet selectors even when type checks pass.
