# `libs/cli/deepagents_cli/integrations/__init__.py`

> Integrations for external systems used by the deepagents CLI.

## Position in the system

This file is part of sandbox integration plumbing. The CLI uses these adapters to create or verify backend sandboxes while keeping provider-specific SDK details behind a common interface.

## Functions and classes

This module has no public functions or classes. It exists for package discovery, typing markers, constants, side-effect imports, or re-export behavior described above.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
