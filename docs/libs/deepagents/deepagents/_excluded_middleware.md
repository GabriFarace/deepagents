# `libs/deepagents/deepagents/_excluded_middleware.py`

> Validation and filtering helpers for `HarnessProfile.excluded_middleware`.

## Position in the system

`create_deep_agent()` assembles middleware stacks and uses these helpers to
apply a resolved harness profile's middleware exclusions. The policy for which
middleware is required is owned by `graph.py` and passed in, so the graph
factory remains the source of truth for required scaffolding.

## Imports and module-level state

The module imports only logging at runtime. Type-only imports keep references
to `AgentMiddleware` and `HarnessProfile` out of import-time execution.

## Functions and classes

### `_validate_excluded_middleware_config(profile: HarnessProfile, *, required_classes: frozenset[type[AgentMiddleware]], required_names: frozenset[str]) -> None`

Validates assembly-time invariants before any stack filtering happens. It
splits the profile's exclusion entries into class-form and string-form sets,
then rejects anything that overlaps required scaffolding classes or names.

Grammar checks for string entries happen earlier in `HarnessProfile`; this
function focuses on the graph-level invariant that required filesystem,
subagent, and permission scaffolding cannot be removed.

### `_raise_on_name_collisions(name_matched_types: dict[str, set[type[AgentMiddleware]]]) -> None`

Raises when a single string exclusion matched multiple concrete middleware
classes in one stack. That usually means a user middleware chose the same
`.name` as another middleware. The error asks callers to use class-form runtime
exclusion to disambiguate.

### `_apply_excluded_middleware(stack: list[AgentMiddleware], profile: HarnessProfile, *, matched_classes: set[type[AgentMiddleware]] | None = None, matched_names: set[str] | None = None) -> list[AgentMiddleware]`

Filters one assembled middleware stack according to the profile. Class entries
match exact concrete types, not subclasses. String entries match
`AgentMiddleware.name` exactly. Optional `matched_classes` and `matched_names`
sets let the caller aggregate matches across multiple stacks before coverage
verification.

The function always returns a fresh list. When removals occur, it logs a debug
message naming the excluded classes and names. It also checks for string-name
collisions within the stack before returning.

### `_verify_excluded_middleware_coverage(profile: HarnessProfile, matched_classes: set[type[AgentMiddleware]], matched_names: set[str], *, required_classes: frozenset[type[AgentMiddleware]], required_names: frozenset[str]) -> None`

Runs after every relevant stack has been filtered. It compares the profile's
requested exclusions with the accumulated matches and raises if any legitimate
entry matched nothing anywhere. This catches typos and stale profile entries
without being too strict about middleware that only appears in the main agent
or only in a subagent stack.

Required scaffolding entries are subtracted as defense in depth because they
should already have been rejected by `_validate_excluded_middleware_config()`.

## Flow walk-through

1. `create_deep_agent()` resolves a `HarnessProfile`.
2. Required middleware classes/names are passed to
   `_validate_excluded_middleware_config()`.
3. Each assembled stack is passed through `_apply_excluded_middleware()`, with
   shared match-tracking sets.
4. After all applicable stacks are filtered,
   `_verify_excluded_middleware_coverage()` rejects unmatched exclusions.

## Gotchas

String exclusions are powerful but can be ambiguous. Prefer class-form
exclusions in runtime profiles when the class is importable; reserve string
form for config files and public aliases such as private middleware
implementations with a stable serialized name.

