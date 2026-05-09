# `libs/cli/deepagents_cli/widgets/welcome.py`

> Welcome banner widget for deepagents-cli.

## Position in the system

This widget sits on the Textual side of the CLI. `app.py` composes these screens and controls around streamed events from the LangGraph server, while the widget itself owns presentation, local interaction state, and the small event messages it emits upward.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from textual.color import Color as TColor`

- `from textual.content import Content`

- `from textual.style import Style as TStyle`

- `from textual.widgets import Static`

- `from deepagents_cli import theme`

- `from deepagents_cli._env_vars import DANGEROUSLY_OVERRIDE_STARTUP_SUBHEADER, HIDE_CWD, HIDE_LANGSMITH_TRACING, HIDE_SPLASH_TIPS, HIDE_SPLASH_VERSION, is_env_truthy`

- `from deepagents_cli._version import __version__`

- `from deepagents_cli.config import _get_editable_install_path, _is_editable_install, fetch_langsmith_project_url, get_banner, get_glyphs, get_langsmith_project_name`

- `from deepagents_cli.widgets._links import open_style_link`


## Functions and classes

### `_pick_tip()`

Pick a tip from `_TIPS` weighted by its associated weight.

Additional notes from the source docstring:

```text
Returns:
    A single tip string, selected with probability proportional to its
    weight in `_TIPS`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `WelcomeBanner`

Welcome banner displayed at startup.

Methods worth reading inside this class:

- `on_mount(self)`: Kick off background fetch for LangSmith project URL.

- `_cancel_defer_timer(self)`: Stop and drop the deferred-display timer if it is still pending.

- `_on_defer_timer_fired(self)`: Reveal the connecting footer once the deferral window expires.

- `reveal_connecting_footer(self)`: Stop deferring the "Connecting..." footer and render it now.

- `_on_theme_change(self)`: Re-render the banner when the app theme changes.

- `_fetch_and_update(self)`: Fetch the LangSmith URL in a thread and update the banner.

- `update_thread_id(self, thread_id: str)`: Update the displayed thread ID and re-render the banner.

- `set_connected(self, mcp_tool_count: int=0, *, mcp_unauthenticated: int=0, mcp_errored: int=0)`: Transition from "connecting" to "ready" state.

- `set_connecting(self)`: Transition back to the "connecting" state.

- `set_idle(self)`: Transition to a neutral state with no connecting spinner or footer.

- `on_click(self, event: Click)`: Open style-embedded hyperlinks on single click.

- `_build_banner(self, project_url: str | None=None)`: Build the banner content.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `build_connecting_footer(*, resuming: bool=False, local_server: bool=False)`

Build a footer shown while waiting for the server to connect.

Additional notes from the source docstring:

```text
Args:
    resuming: Show `'Resuming...'` instead of any `'Connecting...'` variant.
    local_server: Qualify the server as "local" in the connecting message.

        Ignored when `resuming` is `True`.

Returns:
    Content with a connecting status message.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `build_welcome_footer(*, primary_color: str=theme.PRIMARY, tip: str | None=None, show_tip: bool | None=None)`

Build the footer shown at the bottom of the welcome banner.

Additional notes from the source docstring:

```text
Includes a tip to help users discover features unless tips are disabled.

Args:
    primary_color: Color string for the ready prompt.

        Defaults to the module-level ANSI `PRIMARY` constant; widget callers
        should pass the active theme's hex value.
    tip: Tip text to display. When `None`, a random tip is selected.

        Pass an explicit value to keep the tip stable across re-renders.
    show_tip: Whether to show the tip. When `None`, the startup splash tips
        env var controls visibility.

Returns:
    Content with the ready prompt and, when enabled, a tip.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Widget classes are coupled to Textual message names, CSS classes, and screen lifecycle hooks. Changing identifiers here can break bindings in `app.py` or stylesheet selectors even when type checks pass.
