# `libs/cli/deepagents_cli/state_migration.py`

> One-time migration of legacy CLI state files into `~/.deepagents/.state/`.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from deepagents_cli.model_config import DEFAULT_CONFIG_DIR, DEFAULT_STATE_DIR`

- `from deepagents_cli.onboarding import ONBOARDING_MARKER_FILENAME`


## Functions and classes

### `_iter_migrations(config_dir: Path, state_dir: Path, names: Iterable[str])`

This helper performs one narrow step for the surrounding module while keeping normalization, error handling, or side-effect policy in one place.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `migrate_legacy_state(*, config_dir: Path=DEFAULT_CONFIG_DIR, state_dir: Path=DEFAULT_STATE_DIR)`

Move legacy state entries from `config_dir` into `state_dir`.

Additional notes from the source docstring:

```text
Idempotent: each entry is skipped when the destination already exists
(the migration ran on a prior invocation) or when the source does not
exist (nothing to move). Errors on individual entries are logged and
swallowed so a single unmovable file does not block the rest.

Args:
    config_dir: Directory holding legacy state. Defaults to
        `~/.deepagents/`.
    state_dir: Destination directory for state files. Defaults to
        `~/.deepagents/.state/`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
