# `libs/deepagents/deepagents/profiles/_builtin_profiles.py`

> Lazy bootstrap for built-in and third-party profile registrations.

## Position in the system

Provider and harness registries call `_ensure_builtin_profiles_loaded()` before
public registration or lookup. That one function loads Deep Agents' built-in
profiles directly, then invokes third-party entry points:

```text
get/register profile
  -> _ensure_builtin_profiles_loaded()
      -> built-in provider modules
      -> built-in harness modules
      -> deepagents.provider_profiles entry points
      -> deepagents.harness_profiles entry points
```

The direct imports are intentional: built-ins do not depend on package metadata
being present. Third-party plugins use `importlib.metadata` entry points and
are isolated so one broken plugin cannot prevent the SDK from importing.

## Imports and module-level state

`_PROVIDER_PROFILE_GROUP` and `_HARNESS_PROFILE_GROUP` name the two entry-point
groups. `_loaded`, `_loading_thread_id`, and `_BOOTSTRAP_CONDITION` coordinate
exactly-once lazy bootstrap across threads. `_BOOTSTRAP_HARNESS_KEYS` snapshots
the harness registry after bootstrap so later code can tell bootstrap-provided
defaults from profiles registered explicitly by user code.

The module imports the built-in profile modules but does not ask them to
register until `_ensure_builtin_profiles_loaded()` runs.

## Functions and classes

### `_format_plugin_label(ep: EntryPoint) -> str`

Builds a human-readable label for logging and warnings. It prefers the
entry-point name plus distribution name when `importlib.metadata` exposes one,
because entry-point names can collide across packages. It has no side effects.

### `_ensure_builtin_profiles_loaded() -> None`

Runs the profile bootstrap once per interpreter. It first handles concurrency:
if another thread is already loading, callers wait; if the same thread re-enters
during plugin registration, it returns immediately so registration helpers can
call back into the public registry APIs without deadlocking.

The function snapshots both registries before loading. Built-in provider
profiles (`openai`, `openrouter`) and harness profiles (Claude and Codex
profiles) are registered directly, then plugin groups are invoked. If any
exception escapes this sequence, the registries and bootstrap-harness-key
snapshot are restored in place before the exception is re-raised. Successful
bootstrap marks `_loaded`, clears `_loading_thread_id`, and wakes waiters.

### `_invoke_profile_plugins(group: str) -> None`

Enumerates and invokes registration entry points for a single group. It treats
entry-point enumeration failure as environment-level breakage and logs/warns
once for the whole group. Load failures, non-callable targets, and exceptions
raised by the registration callable are logged and warned per plugin, then the
loop continues.

Ordering is whatever `importlib.metadata.entry_points()` returns. Later
plugins may layer on earlier registrations because the public registration
helpers merge profiles additively.

## Flow walk-through

1. A registry lookup or public registration calls the relevant `_ensure_*`
   helper.
2. `_ensure_builtin_profiles_loaded()` claims the bootstrap condition unless
   it is already loaded or re-entering on the same thread.
3. Built-ins register first, so plugin profiles can layer on top.
4. Third-party entry points are loaded and invoked group by group.
5. The harness registry keys are snapshotted as bootstrap-provided defaults.
6. Any failure restores both registries to their pre-bootstrap state.

## Gotchas

Built-in registration failures are treated as SDK bugs and propagate. Plugin
failures are warnings and do not abort bootstrap. That asymmetry is deliberate.

