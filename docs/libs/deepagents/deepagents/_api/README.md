# `libs/deepagents/deepagents/_api/`

> Private API-adapter helpers for the SDK package.

## Position in the system

This package is intentionally internal. It gives Deep Agents a narrow place to
wrap or re-export upstream private helpers so future upstream movement is
localized to one module.

## Module map

| Module | Doc | What to read it for |
|---|---|---|
| `__init__.py` | [`__init__.md`](./__init__.md) | Internal-package marker. |
| `deprecation.py` | [`deprecation.md`](./deprecation.md) | LangChain deprecation adapter and test helper. |

## Gotchas

The package is private. Its APIs may change between minor releases without the
same deprecation guarantees as the public `deepagents` surface.

