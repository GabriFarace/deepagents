# `libs/cli/deepagents_cli/widgets/notification_center.py`

> Notification center modal for pending actionable notices.

## Position in the system

This widget sits on the Textual side of the CLI. `app.py` composes these screens and controls around streamed events from the LangGraph server, while the widget itself owns presentation, local interaction state, and the small event messages it emits upward.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from textual.binding import Binding, BindingType`

- `from textual.containers import Vertical, VerticalScroll`

- `from textual.content import Content`

- `from textual.message import Message`

- `from textual.screen import ModalScreen`

- `from textual.widgets import Static`

- `from deepagents_cli import theme`

- `from deepagents_cli.config import get_glyphs, is_ascii_mode`

- `from deepagents_cli.notifications import ActionId, UpdateAvailablePayload`


## Functions and classes

### `NotificationActionResult`

Dismissal payload identifying which action the user picked.

Additional notes from the source docstring:

```text
The screen returns this via `dismiss()` when the user drills into
a notification and selects an action; it returns `None` when the
user cancels with Esc without committing to an action.
```

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `NotificationRowClicked`

Posted when a notification row is clicked with the mouse.

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `NotificationSuppressRequested`

Posted when the user picks SUPPRESS from a notification's detail modal.

Additional notes from the source docstring:

```text
The center does not dismiss on SUPPRESS because the remaining
notifications should still be reachable in place. The app handles
this message by running the suppress dispatch and calling
`NotificationCenterScreen.reload` with the refreshed registry
snapshot.
```

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `_NotificationRow`

Clickable single-line row displaying a notification's title.

Methods worth reading inside this class:

- `notification(self)`: Return the underlying notification.

- `index(self)`: Return the row index in the parent list.

- `set_selected(self, selected: bool)`: Toggle selection styling.

- `_render(self)`: This builder composes a derived artifact from validated inputs for the agent runtime, deployment bundle, or UI layer.

- `on_click(self, event: Click)`: Dispatch a click as a `NotificationRowClicked` message.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `NotificationCenterScreen`

Modal listing pending notifications with drill-in details.

Additional notes from the source docstring:

```text
Each `PendingNotification` is a single row. Up/Down (or j/k)
moves the cursor; Enter or click pushes a detail modal for the
highlighted entry. The detail modal carries the action list and
dismisses with an `ActionId` or `None`. Esc on the center returns
`None`.
```

Methods worth reading inside this class:

- `compose(self)`: Compose the modal layout.

- `on_mount(self)`: Apply ASCII borders and highlight the first row.

- `_set_selected(self, new_index: int)`: Move the selection cursor to *new_index*.

- `action_move_up(self)`: Move the cursor up one row (wraps at the top).

- `action_move_down(self)`: Move the cursor down one row (wraps at the bottom).

- `action_activate(self)`: Drill into the highlighted notification.

- `action_cancel(self)`: Close without firing any action.

- `on_notification_row_clicked(self, message: NotificationRowClicked)`: Handle a mouse click on a notification row.

- `_drill_into(self, entry: PendingNotification)`: Push a detail modal for *entry*.

- `reload(self, notifications: list[PendingNotification])`: Rebuild the row list from a refreshed snapshot.

- `_detail_screen_for(entry: PendingNotification)`: Pick the appropriate detail modal for *entry*'s payload.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

## Gotchas

Widget classes are coupled to Textual message names, CSS classes, and screen lifecycle hooks. Changing identifiers here can break bindings in `app.py` or stylesheet selectors even when type checks pass.
