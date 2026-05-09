# `libs/deepagents/deepagents/backends/__init__.py`

> Public re-export module for the SDK backend package.

## Position in the system

This file is the import convenience layer for callers who want backend classes
from `deepagents.backends` rather than from implementation modules. It does not
participate in backend execution itself; `FilesystemMiddleware` and agent setup
code import these names so users can configure storage and execution backends
with a small public surface.

```
user code / graph setup
  │
  └─ from deepagents.backends import StateBackend, StoreBackend, ...
       └─ implementation modules
```

## Imports and module-level state

The module imports the concrete implementations (`CompositeBackend`,
`FilesystemBackend`, `LangSmithSandbox`, `LocalShellBackend`, `StateBackend`,
`StoreBackend`) plus the common protocol and store namespace helper types. Its
only module-level state is `__all__`, which defines the intended public API.

## Functions and classes

This module defines no functions or classes. Its public block is the export
list: `DEFAULT_EXECUTE_TIMEOUT`, `BackendContext`, `BackendProtocol`,
`CompositeBackend`, `FilesystemBackend`, `LangSmithSandbox`,
`LocalShellBackend`, `NamespaceFactory`, `StateBackend`, and `StoreBackend`.

## Gotchas

`LangSmithSandbox` is exported here even though it depends on the optional
LangSmith sandbox SDK at runtime. Importing the name is cheap because the
implementation only imports LangSmith-specific classes under `TYPE_CHECKING` or
inside methods.
