# `libs/cli/deepagents_cli/widgets/history.py`

> Command history manager for input persistence.

## Position in the system

This widget sits on the Textual side of the CLI. `app.py` composes these screens and controls around streamed events from the LangGraph server, while the widget itself owns presentation, local interaction state, and the small event messages it emits upward.

## Functions and classes

### `HistoryManager`

Manages command history with file persistence.

Additional notes from the source docstring:

```text
Uses append-only writes for concurrent safety. Multiple agents can
safely write to the same history file without corruption.
```

Methods worth reading inside this class:

- `_load_history(self)`: Load history from file.

- `_append_to_file(self, text: str)`: Append a single entry to history file (concurrent-safe).

- `_compact_history(self)`: Rewrite history file to remove old entries.

- `add(self, text: str)`: Add a command to history.

- `get_previous(self, current_input: str, *, query: str='')`: Get the previous history entry matching a substring query.

- `get_next(self)`: Get the next history entry matching the stored query.

- `in_history(self)`: Whether currently navigating history entries.

- `reset_navigation(self)`: Reset navigation state.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

## Gotchas

Widget classes are coupled to Textual message names, CSS classes, and screen lifecycle hooks. Changing identifiers here can break bindings in `app.py` or stylesheet selectors even when type checks pass.
