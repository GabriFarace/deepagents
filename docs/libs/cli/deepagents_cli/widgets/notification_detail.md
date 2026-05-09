# `libs/cli/deepagents_cli/widgets/notification_detail.py`

> Generic detail modal for a single pending notification.

## Position in the system

This widget sits on the Textual side of the CLI. `app.py` composes these screens and controls around streamed events from the LangGraph server, while the widget itself owns presentation, local interaction state, and the small event messages it emits upward.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from textual.binding import Binding, BindingType`

- `from textual.containers import Vertical`

- `from textual.content import Content`

- `from textual.message import Message`

- `from textual.screen import ModalScreen`

- `from textual.widgets import Static`

- `from deepagents_cli import theme`

- `from deepagents_cli.config import get_glyphs, is_ascii_mode`


## Functions and classes

### `DetailActionActivated`

Posted when an `_ActionOption` is clicked with the mouse.

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `_ActionOption`

Clickable single-line action row.

Methods worth reading inside this class:

- `action(self)`: Return the underlying action.

- `set_selected(self, selected: bool)`: Toggle selection styling.

- `_render(self)`: This builder composes a derived artifact from validated inputs for the agent runtime, deployment bundle, or UI layer.

- `on_click(self, event: Click)`: Dispatch a click as a `DetailActionActivated` message.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `NotificationDetailScreen`

Modal displaying a single notification's title, body, and actions.

Additional notes from the source docstring:

```text
Activation returns the chosen `ActionId` via `dismiss()`; Esc
returns `None` so the caller can keep the underlying notification
center open.
```

Methods worth reading inside this class:

- `compose(self)`: Compose the modal layout.

- `on_mount(self)`: Apply ASCII borders and highlight the primary action by default.

- `_set_selected(self, new_index: int)`: Move the selection cursor to *new_index*.

- `action_move_up(self)`: Move the cursor up one row (wraps at the top).

- `action_move_down(self)`: Move the cursor down one row (wraps at the bottom).

- `action_activate(self)`: Dismiss with the highlighted action's id.

- `action_cancel(self)`: Close without firing any action.

- `on_detail_action_activated(self, message: DetailActionActivated)`: Handle a mouse click on an action row.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

## Gotchas

Widget classes are coupled to Textual message names, CSS classes, and screen lifecycle hooks. Changing identifiers here can break bindings in `app.py` or stylesheet selectors even when type checks pass.
