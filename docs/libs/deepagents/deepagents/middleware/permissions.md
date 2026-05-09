# `libs/deepagents/deepagents/middleware/permissions.py`

> Backward-compatible permissions facade. It re-exports
> `FilesystemPermission` from the filesystem middleware.

## Position in the system

Older callers may import filesystem permission rules from
`deepagents.middleware.permissions`. The implementation now lives in
`middleware/filesystem.py`, but this file preserves the old import path.

```
deepagents.middleware.permissions.FilesystemPermission
  └─ same object as deepagents.middleware.filesystem.FilesystemPermission
```

## Imports and module-level state

The only import is `FilesystemPermission`. `__all__` contains that one name.
There is no independent permission logic in this module.

## Functions and classes

### `FilesystemPermission`

This is a re-export, not a new class. See
`docs/libs/deepagents/deepagents/middleware/filesystem.md` for the rule schema,
validation behavior, and how the filesystem tools apply permission checks.

## Gotchas

Do not add new permission behavior here. The source of truth is
`filesystem.py`; changing this file would only affect import compatibility.
