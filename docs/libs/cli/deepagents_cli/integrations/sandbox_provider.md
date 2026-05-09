# `libs/cli/deepagents_cli/integrations/sandbox_provider.py`

> Sandbox provider interface used by the deepagents CLI.

## Position in the system

This file is part of sandbox integration plumbing. The CLI uses these adapters to create or verify backend sandboxes while keeping provider-specific SDK details behind a common interface.

## Functions and classes

### `SandboxError`

Base error for sandbox provider operations.

Methods worth reading inside this class:

- `original_exc(self)`: Return the original exception that caused this error, if any.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `SandboxNotFoundError`

Raised when the requested sandbox cannot be found.

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `SandboxProvider`

Interface for creating and deleting sandbox backends.

Methods worth reading inside this class:

- `get_or_create(self, *, sandbox_id: str | None=None, **kwargs: Any)`: Get an existing sandbox, or create one if needed.

- `delete(self, *, sandbox_id: str, **kwargs: Any)`: Delete a sandbox by id.

- `aget_or_create(self, *, sandbox_id: str | None=None, **kwargs: Any)`: Async wrapper around get_or_create.

- `adelete(self, *, sandbox_id: str, **kwargs: Any)`: Async wrapper around delete.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
