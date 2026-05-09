# `libs/cli/deepagents_cli/onboarding.py`

> First-run onboarding state for the interactive CLI.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from deepagents_cli._env_vars import DEBUG_ONBOARDING, is_env_truthy`

- `from deepagents_cli.model_config import DEFAULT_STATE_DIR`


## Functions and classes

### `onboarding_marker_path(state_dir: Path | None=None)`

Return the first-run onboarding marker path.

Additional notes from the source docstring:

```text
Args:
    state_dir: Optional state directory override for tests.

Returns:
    Path to the onboarding completion marker.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `has_completed_onboarding(state_dir: Path | None=None)`

Return whether the user has completed onboarding.

Additional notes from the source docstring:

```text
Args:
    state_dir: Optional state directory override for tests.

Returns:
    `True` when the onboarding marker exists, otherwise `False`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `mark_onboarding_complete(state_dir: Path | None=None)`

Persist that onboarding has completed.

Additional notes from the source docstring:

```text
Args:
    state_dir: Optional state directory override for tests.

Returns:
    `True` when the marker was written, otherwise `False`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `write_onboarding_name_memory(name: str, assistant_id: str, *, memory_path: Path | None=None)`

Persist the optional onboarding name into user agent memory.

Additional notes from the source docstring:

```text
Empty or whitespace-only names are skipped (no file is written).

Args:
    name: Submitted user name.
    assistant_id: Agent identifier whose user memory should be updated.
    memory_path: Optional memory file override for tests.

Returns:
    `True` when memory was written, otherwise `False`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_normalize_memory_name(name: str)`

Normalize whitespace in a name before writing it to memory.

Additional notes from the source docstring:

```text
Returns:
    Name with leading/trailing whitespace stripped and internal runs
        collapsed to single spaces.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_onboarding_name_memory_block(name: str)`

Return the managed memory block for an onboarding name.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_upsert_onboarding_name_memory(existing: str, block: str)`

Insert or replace the managed onboarding name memory block.

Additional notes from the source docstring:

```text
Returns:
    Updated memory file content.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `should_run_onboarding(state_dir: Path | None=None)`

Return whether onboarding should open at interactive startup.

Additional notes from the source docstring:

```text
Args:
    state_dir: Optional state directory override for tests.

Returns:
    `True` when the debug override is enabled or no completion marker exists.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
