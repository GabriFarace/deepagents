# `libs/cli/deepagents_cli/built_in_skills/__init__.py`

> Built-in skills that ship with the Deep Agents CLI.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Functions and classes

This module has no public functions or classes. It exists for package discovery, typing markers, constants, side-effect imports, or re-export behavior described above.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
