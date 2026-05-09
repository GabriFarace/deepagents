# `libs/cli/deepagents_cli/widgets/__init__.py`

> Textual widgets for deepagents-cli.

## Position in the system

This widget sits on the Textual side of the CLI. `app.py` composes these screens and controls around streamed events from the LangGraph server, while the widget itself owns presentation, local interaction state, and the small event messages it emits upward.

## Functions and classes

This module has no public functions or classes. It exists for package discovery, typing markers, constants, side-effect imports, or re-export behavior described above.

## Gotchas

Widget classes are coupled to Textual message names, CSS classes, and screen lifecycle hooks. Changing identifiers here can break bindings in `app.py` or stylesheet selectors even when type checks pass.
