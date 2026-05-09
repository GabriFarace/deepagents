# `libs/cli/deepagents_cli/widgets/agent_selector.py`

> Interactive agent selector screen for `/agents` command.

## Position in the system

This widget sits on the Textual side of the CLI. `app.py` composes these screens and controls around streamed events from the LangGraph server, while the widget itself owns presentation, local interaction state, and the small event messages it emits upward.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from textual.binding import Binding, BindingType`

- `from textual.containers import Vertical`

- `from textual.content import Content`

- `from textual.screen import ModalScreen`

- `from textual.widgets import OptionList, Static`

- `from textual.widgets.option_list import Option`

- `from deepagents_cli import theme`

- `from deepagents_cli.config import Glyphs, get_glyphs, is_ascii_mode`

- `from deepagents_cli.model_config import clear_default_agent, save_default_agent`


## Functions and classes

### `AgentSelectorScreen`

Modal dialog for switching between available agents.

Additional notes from the source docstring:

```text
Displays agents found in `~/.deepagents/` in an `OptionList`. Returns the
selected agent name on Enter, or `None` on Esc (no change).
`Ctrl+S` toggles the highlighted agent as the persisted default
(`[agents].default`), mirroring the model selector's affordance.
```

Methods worth reading inside this class:

- `compose(self)`: Compose the screen layout.

- `_build_options(self)`: Build option entries with `(current)` / `(default)` suffixes.

- `_format_label(self, name: str)`: Render an agent's label with `(current)` / `(default)` markers.

- `_current_index(self)`: Return the index of the current agent in the option list, or 0.

- `_help_text(glyphs: Glyphs)`: Build the help-line text shown beneath the option list.

- `on_mount(self)`: Apply ASCII border if needed.

- `on_option_list_option_selected(self, event: OptionList.OptionSelected)`: Dismiss with the selected agent name.

- `action_cancel(self)`: Cancel without switching agents.

- `action_cursor_down(self)`: Move the option list cursor down (Tab).

- `action_cursor_up(self)`: Move the option list cursor up (Shift+Tab).

- `action_set_default(self)`: Toggle the highlighted agent as the persisted default.

- `_refresh_options(self, option_list: OptionList, highlighted: int, new_default: str | None)`: Rebuild option labels in place to track the new `(default)` state.

- `_restore_help_text(self)`: Restore the default help text after a transient message.

- `_option_list(self)`: Return the agent `OptionList`, or `None` when the screen is empty.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

## Gotchas

Widget classes are coupled to Textual message names, CSS classes, and screen lifecycle hooks. Changing identifiers here can break bindings in `app.py` or stylesheet selectors even when type checks pass.
