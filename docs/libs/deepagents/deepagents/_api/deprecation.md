# `libs/deepagents/deepagents/_api/deprecation.py`

> Adapter around LangChain Core deprecation helpers.

## Position in the system

Deep Agents uses LangChain's deprecation machinery, but this module centralizes
the import surface and adds a wrapper for warning stack attribution. If
LangChain moves the private helper module, Deep Agents should only need to
change this file.

## Imports and module-level state

The module re-exports `deprecated`,
`suppress_langchain_deprecation_warning`, and
`LangChainDeprecationWarning` from `langchain_core._api.deprecation`. It imports
the upstream `warn_deprecated` as `_lc_warn_deprecated` so the local
`warn_deprecated()` wrapper can capture and re-emit its formatted warning.

## Functions and classes

### `warn_deprecated(since: str, *, message: str = "", name: str = "", alternative: str = "", alternative_import: str = "", pending: bool = False, obj_type: str = "", addendum: str = "", removal: str = "", package: str = "", stacklevel: int = 2) -> None`

Formats a deprecation warning through LangChain Core, captures the emitted
warning, then re-emits it with caller-controlled `stacklevel`. This fixes the
case where the upstream helper's hardcoded stacklevel points one frame too high
when called directly from inside a deprecated method body.

The function mutates no SDK state. It temporarily changes warning filters
inside `warnings.catch_warnings(record=True)` and then emits the captured
warning message/category with `warnings.warn()`.

### `reset_deprecation_dedupe(*targets: object) -> None`

Testing helper that resets the closure-bound `warned` flag used by LangChain's
`@deprecated` decorator. It accepts decorated callables, decorated methods, and
`property` objects wrapping decorated getters. Targets without the expected
closure variable are skipped silently, so tests can call it defensively.

The function mutates closure cell contents for matching decorated targets. This
is intentionally test-oriented: it makes per-call deprecation assertions stable
even though LangChain's decorator normally warns only once per process.

## Gotchas

`reset_deprecation_dedupe()` reaches into closure internals. Keep it scoped to
tests or test-support code, not runtime behavior.

