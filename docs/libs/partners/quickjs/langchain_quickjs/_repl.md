# `partners/quickjs/langchain_quickjs/_repl.py`

> Thread-keyed QuickJS REPL registry, console bridge, and result formatter.

## Position in the system

This partner package implements or supports a BackendProtocol-compatible sandbox. The core SDK can use these classes through its backend abstraction without depending directly on the provider.

## Imports and module-level state

This file imports `__future__, asyncio, contextlib, hashlib, json, logging, threading, uuid, dataclasses, typing, quickjs_rs, quickjs_rs` and other helpers.
Module constants worth noticing: `_HANDLE_PLACEHOLDER`.

## Functions and classes

### `EvalOutcome`

Normalized result of a single REPL eval. Exactly one of ``result`` / ``error`` is meaningful per call; ``stdout`` is collected from ``console.*`` regardless. This class inherits from `object` and is the main object for this part of the module.

## Gotchas

Several blocks are async and assume they run inside the owning framework event loop. Calling them from sync code needs the package CLI or an explicit async runner.
