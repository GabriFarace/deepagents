# `libs/cli/deepagents_cli/widgets/update_available.py`

> Dedicated modal for the update-available notification.

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

- `from deepagents_cli._version import CHANGELOG_URL`

- `from deepagents_cli.config import get_glyphs, is_ascii_mode`

- `from deepagents_cli.notifications import ActionId`

- `from deepagents_cli.widgets._links import open_url_async`


## Functions and classes

### `ChangelogClicked`

Posted when the changelog row is clicked with the mouse.

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `_ActionOption`

Clickable single-line action row.

Methods worth reading inside this class:

- `action(self)`: Return the underlying action.

- `set_selected(self, selected: bool)`: Toggle selection styling.

- `_render(self)`: This builder composes a derived artifact from validated inputs for the agent runtime, deployment bundle, or UI layer.

- `on_click(self, event: Click)`: Swallow the click without activating.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `_ChangelogOption`

Secondary row that opens the changelog URL in a browser.

Additional notes from the source docstring:

```text
Visually grouped with the action rows so Tab/Shift+Tab navigation
covers it uniformly, but activating it does not dismiss the modal
— it is a "view more info" action, not one of the three
mutually-exclusive update dispositions.
```

Methods worth reading inside this class:

- `set_selected(self, selected: bool)`: Toggle selection styling.

- `_render(self)`: This builder composes a derived artifact from validated inputs for the agent runtime, deployment bundle, or UI layer.

- `on_click(self, event: Click)`: Dispatch a click as a `ChangelogClicked` message.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `UpdateAvailableScreen`

Modal dedicated to the update-available notification.

Additional notes from the source docstring:

```text
Renders the entry's title and body, a "View changelog" row, and
one row per configured action. Dismisses with the selected
`ActionId`, or `None` on Esc. Activating the changelog row opens
the URL in a browser and leaves the modal open.
```

Methods worth reading inside this class:

- `compose(self)`: Compose the modal layout.

- `on_mount(self)`: Apply ASCII borders and highlight the primary action by default.

- `_set_selected(self, new_index: int)`: Move the selection cursor to *new_index*.

- `action_move_up(self)`: Move the cursor up one row (wraps at the top).

- `action_move_down(self)`: Move the cursor down one row (wraps at the bottom).

- `action_activate(self)`: Fire the highlighted option.

- `action_cancel(self)`: Close without firing any action.

- `on_changelog_clicked(self, message: ChangelogClicked)`: Handle a mouse click on the changelog row.

- `_open_changelog(self)`: Open `CHANGELOG_URL` in a browser without closing the modal.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

## Gotchas

Widget classes are coupled to Textual message names, CSS classes, and screen lifecycle hooks. Changing identifiers here can break bindings in `app.py` or stylesheet selectors even when type checks pass.
