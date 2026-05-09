# `libs/cli/deepagents_cli/widgets/loading.py`

> Loading widget with animated spinner for agent activity.

## Position in the system

This widget sits on the Textual side of the CLI. `app.py` composes these screens and controls around streamed events from the LangGraph server, while the widget itself owns presentation, local interaction state, and the small event messages it emits upward.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from textual.containers import Horizontal`

- `from textual.content import Content`

- `from textual.widgets import Static`

- `from deepagents_cli.config import get_glyphs`

- `from deepagents_cli.formatting import format_duration`


## Functions and classes

### `Spinner`

Animated spinner using charset-appropriate frames.

Methods worth reading inside this class:

- `frames(self)`: Get spinner frames from glyphs config.

- `next_frame(self)`: Get next animation frame.

- `current_frame(self)`: Get current frame without advancing.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `LoadingWidget`

Animated loading indicator with status text and elapsed time.

Additional notes from the source docstring:

```text
Displays: <spinner> Thinking... (3s, esc to interrupt)
```

Methods worth reading inside this class:

- `compose(self)`: Compose the loading widget layout.

- `on_mount(self)`: Start animation on mount.

- `on_unmount(self)`: Stop the animation timer when the widget leaves the DOM.

- `remove(self)`: Stop animation before delegating DOM removal to Textual.

- `_stop_timer(self)`: Stop the animation timer if it is running.

- `_update_animation(self)`: Update spinner and elapsed time.

- `set_status(self, status: str)`: Update the status text.

- `pause(self, status: str='Awaiting decision')`: Pause the animation and update status.

- `resume(self)`: Resume the animation.

- `stop(self)`: Stop the animation (widget will be removed by caller).


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

## Gotchas

Widget classes are coupled to Textual message names, CSS classes, and screen lifecycle hooks. Changing identifiers here can break bindings in `app.py` or stylesheet selectors even when type checks pass.
