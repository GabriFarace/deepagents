# `libs/deepagents/deepagents/__init__.py`

> Public package-root re-exports for the Deep Agents SDK.

## Position in the system

Most user code imports from this package root. The file re-exports the primary
agent factory, version, core middleware/subagent types, filesystem permission
types, and the beta profile APIs.

## Functions and classes

This file defines no functions or classes. It is a concise re-export module and
is covered by the package [`README.md`](./README.md) plus the docs for the
individual implementation files.

## Gotchas

Adding a public symbol here expands the SDK import surface. Keep the
implementation details in their own modules and use this file only for
intentional public API.

