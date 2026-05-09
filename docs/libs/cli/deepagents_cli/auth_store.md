# `libs/cli/deepagents_cli/auth_store.py`

> User-level credential storage for model providers.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Functions and classes

### `ApiKeyCredential`

A persisted API key credential.

Additional notes from the source docstring:

```text
The `type` field is the discriminator that lets `OAuthCredential` (added
later) coexist in the same file without migration.
```

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `OAuthCredential`

A persisted OAuth subscription credential.

Additional notes from the source docstring:

```text
Stub kept here so the `StoredCredential` discriminated union narrows
correctly today and the OAuth implementation lands as a pure addition.
No code path produces or consumes this shape yet.
```

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `WriteOutcome`

Result of a credential write that may have warnings to surface.

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `_auth_path()`

Return `~/.deepagents/.state/auth.json`.

Additional notes from the source docstring:

```text
Resolved at call time (not import time) so tests can redirect storage by
monkeypatching `deepagents_cli.model_config.DEFAULT_STATE_DIR` — same
pattern `mcp_auth._tokens_dir` uses.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_read_raw()`

Read and validate the on-disk auth file.

Additional notes from the source docstring:

```text
Returns:
    The decoded JSON object, or `None` when the file is missing.

Raises:
    RuntimeError: If the file exists but cannot be parsed or has an
        unsupported schema version.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_write_raw(data: dict)`

Atomically write `data` as the new auth file with 0600 perms.

Additional notes from the source docstring:

```text
Mirrors `mcp_auth.FileTokenStorage._write` so the security posture is
consistent across both stores. If you change this, update
`mcp_auth.FileTokenStorage._write` too — they share threat model.

Returns:
    Tuple of warning strings for chmod failures the caller should
    surface to the user. Empty when permissions were locked down
    successfully (or on Windows where POSIX modes don't apply).
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `load_credentials()`

Return all stored credentials keyed by provider name.

Additional notes from the source docstring:

```text
Returns:
    Mapping of provider name to its stored credential. Empty when no
    credentials are persisted yet.

Raises:
    RuntimeError: If the file exists but is corrupt or has an unsupported
        schema version. Caller is expected to surface a remediation hint.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_coerce_credential(raw: Any)`

Validate one raw credential entry, returning `None` on shape mismatch.

Additional notes from the source docstring:

```text
Centralizes the runtime check against the `StoredCredential` union so
`load_credentials` doesn't repeat the per-field guard logic and so a
single helper can grow as new variants are added.

Returns:
    The coerced `StoredCredential`, or `None` when the entry doesn't
    match any known variant's shape.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `get_stored_key(provider: str)`

Return the stored API key for `provider`, or `None` if unset.

Additional notes from the source docstring:

```text
Returns `None` for stored OAuth credentials too — callers that need
OAuth tokens should read `load_credentials()` directly and narrow on
`type`.

Raises:
    RuntimeError: If the credential file is corrupt.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `set_stored_key(provider: str, key: str)`

Persist an API key for `provider`.

Additional notes from the source docstring:

```text
Empty / whitespace-only keys are rejected so callers don't accidentally
write a sentinel that masks a working environment variable (see
`apply_stored_credentials` in `model_config` — a stored empty would
unconditionally overwrite the env var).

Args:
    provider: Provider identifier (e.g., `"anthropic"`).
    key: The API key value. Whitespace is stripped before storage.

Returns:
    A `WriteOutcome` whose `warnings` tuple lists chmod failures the
    caller should surface to the user. Empty on a clean save.

Raises:
    ValueError: If `provider` or the stripped `key` is empty.
    RuntimeError: If the credential file is corrupt and cannot be read.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `delete_stored_key(provider: str)`

Remove a stored credential for `provider`.

Additional notes from the source docstring:

```text
Args:
    provider: Provider identifier.

Returns:
    `True` if a credential was removed, `False` if none was stored.

Raises:
    RuntimeError: If the credential file is corrupt and cannot be read.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `list_configured_providers()`

Return providers that currently have a stored credential, sorted.

Additional notes from the source docstring:

```text
Raises:
    RuntimeError: If the credential file is corrupt.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
