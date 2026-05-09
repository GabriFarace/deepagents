# `libs/cli/deepagents_cli/widgets/status.py`

> Status bar widget for deepagents-cli.

## Position in the system

This widget sits on the Textual side of the CLI. `app.py` composes these screens and controls around streamed events from the LangGraph server, while the widget itself owns presentation, local interaction state, and the small event messages it emits upward.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from textual.containers import Horizontal`

- `from textual.content import Content`

- `from textual.css.query import NoMatches`

- `from textual.reactive import reactive`

- `from textual.widget import Widget`

- `from textual.widgets import Static`

- `from deepagents_cli._env_vars import HIDE_CWD, HIDE_GIT_BRANCH, is_env_truthy`

- `from deepagents_cli.config import get_glyphs`


## Functions and classes

### `ModelLabel`

A label that displays a model name, right-aligned with smart truncation.

Additional notes from the source docstring:

```text
When the full `provider:model` text doesn't fit, the provider is dropped
first. If the bare model name still doesn't fit, it is left-truncated
with a leading ellipsis so the most distinctive tail stays visible.
```

Methods worth reading inside this class:

- `_clean_model(self)`: Strip the provider's registered prefix so the status bar stays compact.

- `get_content_width(self, container: Size, viewport: Size)`: Return the intrinsic width so `width: auto` works.

- `render(self)`: Render the model label with width-aware truncation.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `StatusBar`

Status bar showing mode, auto-approve, cwd, git branch, tokens, and model.

Methods worth reading inside this class:

- `compose(self)`: Compose the status bar layout.

- `on_resize(self, event: events.Resize)`: Manage visibility of status items based on terminal width.

- `on_mount(self)`: Set reactive values after mount to trigger watchers safely.

- `watch_mode(self, mode: str)`: Update mode indicator when mode changes.

- `watch_auto_approve(self, new_value: bool)`: Update auto-approve indicator when state changes.

- `watch_cwd(self, new_value: str)`: Update cwd display when it changes.

- `watch_branch(self, new_value: str)`: Update branch display when it changes.

- `watch_status_message(self, new_value: str)`: Update status message display.

- `_format_cwd(self, cwd_path: str='')`: Format the current working directory for display.

- `set_mode(self, mode: str)`: Set the current input mode.

- `set_auto_approve(self, *, enabled: bool)`: Set the auto-approve state.

- `set_status_message(self, message: str)`: Set the status message.

- `watch_tokens(self, new_value: int)`: Update token display when count changes.

- `_render_tokens(self, count: int, *, approximate: bool=False)`: Render the token count into the display widget.

- `set_tokens(self, count: int, *, approximate: bool=False)`: Set the token count.

- `hide_tokens(self)`: Hide the token display (e.g., during streaming).

- `set_model(self, *, provider: str, model: str)`: Update the model display text.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

## Gotchas

Widget classes are coupled to Textual message names, CSS classes, and screen lifecycle hooks. Changing identifiers here can break bindings in `app.py` or stylesheet selectors even when type checks pass.
