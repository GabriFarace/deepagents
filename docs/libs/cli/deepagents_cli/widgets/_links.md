# `libs/cli/deepagents_cli/widgets/_links.py`

> Shared link-click handling for Textual widgets.

## Position in the system

This widget sits on the Textual side of the CLI. `app.py` composes these screens and controls around streamed events from the LangGraph server, while the widget itself owns presentation, local interaction state, and the small event messages it emits upward.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from deepagents_cli.unicode_security import check_url_safety, strip_dangerous_unicode`


## Functions and classes

### `open_url_async(url: str, *, app: App)`

Open url in a browser and toast on failure.

Additional notes from the source docstring:

```text
Runs `webbrowser.open` in a thread, catches the platform errors
that can arise when no browser backend is available, and posts a
warning toast containing the URL so the user can copy it manually
instead of the failure vanishing into a background worker log.

Args:
    url: The URL to open.
    app: App used to post the failure toast.

Returns:
    `True` when the browser accepted the URL; `False` otherwise
        (in which case a warning toast has already been posted).
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `open_style_link(event: Click)`

Open the URL from a Rich link style on click, if present.

Additional notes from the source docstring:

```text
Rich `Style(link=...)` embeds OSC 8 terminal hyperlinks, but Textual's
mouse capture intercepts normal clicks before the terminal can act on them.
By handling the Textual click event directly we open the URL with a single
click, matching the behavior of links in the Markdown widget.

URLs that fail the safety check (e.g. containing hidden Unicode or
homograph domains) are blocked and not opened; the event bubbles and a
warning is logged and displayed as a Textual notification.

On success the event is stopped so it does not bubble further. On failure
(e.g. no browser available in a headless environment) the error is logged at
debug level and the event bubbles normally.

Args:
    event: The Textual click event to inspect.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Widget classes are coupled to Textual message names, CSS classes, and screen lifecycle hooks. Changing identifiers here can break bindings in `app.py` or stylesheet selectors even when type checks pass.
