# `libs/cli/deepagents_cli/widgets/theme_selector.py`

> Interactive theme selector screen for /theme command.

## Position in the system

This widget sits on the Textual side of the CLI. `app.py` composes these screens and controls around streamed events from the LangGraph server, while the widget itself owns presentation, local interaction state, and the small event messages it emits upward.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from textual.binding import Binding, BindingType`

- `from textual.containers import Vertical`

- `from textual.screen import ModalScreen`

- `from textual.widgets import OptionList, Static`

- `from textual.widgets.option_list import Option`

- `from deepagents_cli import theme`

- `from deepagents_cli.config import get_glyphs, is_ascii_mode`


## Functions and classes

### `ThemeSelectorScreen`

Modal dialog for theme selection with live preview.

Additional notes from the source docstring:

```text
Displays available themes in an `OptionList`. Navigating the option list
applies a live preview by swapping the app theme. Returns the selected
theme name on Enter, or `None` on Esc (restoring the original theme).
```

Methods worth reading inside this class:

- `compose(self)`: Compose the screen layout.

- `on_mount(self)`: Apply ASCII border if needed.

- `on_option_list_option_highlighted(self, event: OptionList.OptionHighlighted)`: Live-preview the highlighted theme.

- `on_option_list_option_selected(self, event: OptionList.OptionSelected)`: Commit the selected theme.

- `action_cancel(self)`: Restore the original theme and dismiss.

- `action_cursor_down(self)`: Move the option list cursor down (Tab).

- `action_cursor_up(self)`: Move the option list cursor up (Shift+Tab).


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

## Gotchas

Widget classes are coupled to Textual message names, CSS classes, and screen lifecycle hooks. Changing identifiers here can break bindings in `app.py` or stylesheet selectors even when type checks pass.
