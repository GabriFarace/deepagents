# `libs/cli/deepagents_cli/update_check.py`

> Update lifecycle for `deepagents-cli`.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from deepagents_cli._version import PYPI_URL, SDK_PYPI_URL, USER_AGENT, __version__`

- `from deepagents_cli.model_config import DEFAULT_CONFIG_PATH, DEFAULT_STATE_DIR`


## Functions and classes

### `_parse_version(v: str)`

Parse a PEP 440 version string into a comparable `Version` object.

Additional notes from the source docstring:

```text
Supports stable (`1.2.3`) and pre-release (`1.2.3a1`, `1.2.3rc2`) versions.

Args:
    v: Version string like `'1.2.3'` or `'1.2.3a1'`.

Returns:
    A `packaging.version.Version` instance.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_latest_from_releases(releases: dict[str, list[object]], *, include_prereleases: bool)`

Pick the newest version from a PyPI `releases` mapping.

Additional notes from the source docstring:

```text
Skips versions with no uploaded files (empty entries) and, when
*include_prereleases* is `False`, skips pre-release versions.

Args:
    releases: The `releases` dict from the PyPI JSON API.
    include_prereleases: Whether to consider pre-release versions.

Returns:
    The highest matching version string, or `None` if none qualify.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `get_latest_version(*, bypass_cache: bool=False, include_prereleases: bool=False)`

Fetch the latest deepagents-cli version from PyPI, with caching.

Additional notes from the source docstring:

```text
Results are cached to `CACHE_FILE` to avoid repeated network calls.
The cache stores both the latest stable and pre-release versions so a
single PyPI request serves both code paths.

Args:
    bypass_cache: Skip the cache and always hit PyPI.
    include_prereleases: When `True`, consider pre-release versions
        (alpha, beta, rc). Stable users should leave this `False`.

Returns:
    The latest version string, or `None` on any failure.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_extract_release_times(payload: dict[str, Any], *, stable: str, prerelease: str | None)`

Pull `upload_time_iso_8601` for the given versions out of a PyPI payload.

Additional notes from the source docstring:

```text
PyPI lists per-file uploads; the first file's timestamp is used as a
stand-in for the release's publish time (files typically land within
seconds of each other). Looks up both versions under `releases[ver]`
rather than `payload["urls"]`, which reflects the project's
`info.version` and may not match `stable` when the latest on PyPI is
a pre-release.

Args:
    payload: Parsed PyPI JSON response.
    stable: Latest stable version string.
    prerelease: Latest pre-release version string, if any.

Returns:
    Mapping of version string to ISO-8601 upload time. Silently drops
    versions whose timestamp is missing or malformed.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_upload_time(file_entry: object)`

Return `upload_time_iso_8601` from a PyPI file entry, or `None`.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `get_release_time(version: str | None)`

Return the cached ISO-8601 upload time for `version`, or `None`.

Additional notes from the source docstring:

```text
Only versions captured during a prior `get_latest_version` call are
available; unknown versions, or a `None` input, return `None`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_format_age_from_iso(iso: str | None)`

Return `'released Nd ago'` for an ISO-8601 timestamp, or `""` on failure.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `format_release_age(version: str | None)`

Return a human-readable age for `version` (e.g., `'released 3d ago'`).

Additional notes from the source docstring:

```text
Returns an empty string when the upload time is unknown (cache entry
lacks `release_times` for this version, or a `None` version) so callers
can concatenate unconditionally.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `format_age_suffix(version: str | None)`

Return `", released Nd ago"` for `version`, or `""` when unknown.

Additional notes from the source docstring:

```text
The `", "` separator is included so callers can splice the age into a
parenthetical unconditionally — if the age is unknown, the empty
string collapses cleanly into the surrounding text.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `get_sdk_release_time(version: str | None, *, bypass_cache: bool=False)`

Return the ISO-8601 upload time for `deepagents` SDK `version`.

Additional notes from the source docstring:

```text
Reads from `CACHE_FILE` under `sdk_release_times`, falling back to a
single PyPI fetch on cache miss and writing the result back so
subsequent calls stay local.

Args:
    version: Installed SDK version string.
    bypass_cache: Skip the cache read and always hit PyPI.

        The result is still written back to the cache.

Returns:
    The ISO-8601 upload timestamp, or `None` on any failure (missing
        version, unresolvable on PyPI, `requests` unavailable, or
        network error).
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_write_sdk_release_time(version: str, iso: str)`

Merge a single SDK release timestamp into `CACHE_FILE`.

Additional notes from the source docstring:

```text
A corrupt existing cache is overwritten rather than propagating the
decode error — otherwise every caller would keep paying the PyPI
round-trip because the write never succeeds.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `format_sdk_release_age(version: str | None)`

Return a human-readable age for SDK `version` (e.g., `'released 3d ago'`).

Additional notes from the source docstring:

```text
May trigger a single PyPI fetch on cache miss (3s timeout). Returns an
empty string on any failure so callers can concatenate unconditionally.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `format_sdk_age_suffix(version: str | None)`

Return `", released Nd ago"` for SDK `version`, or `""` when unknown.

Additional notes from the source docstring:

```text
The `", "` separator is included so callers can splice the age into a
line unconditionally — if the age is unknown, the empty string
collapses cleanly into the surrounding text. May trigger a single
PyPI fetch on cache miss.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_read_update_state()`

Read the shared update state file.

Additional notes from the source docstring:

```text
Returns:
    Parsed dict, or empty dict on missing/corrupt file.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_write_update_state(patch: dict[str, object], *, remove_keys: tuple[str, ...]=())`

Merge *patch* into the shared update state file and drop *remove_keys*.

Additional notes from the source docstring:

```text
Args:
    patch: Keys to merge into the existing state.
    remove_keys: Keys to drop from the existing state before writing.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `should_notify_update(latest: str)`

Return whether the user should be notified about version *latest*.

Additional notes from the source docstring:

```text
Throttles notifications to at most once per `CACHE_TTL` period for a
given version, preventing repeated banners every session.

Args:
    latest: The version string to check against.

Returns:
    `True` if the user should see the update banner, `False` if the
        notification was already shown within the `CACHE_TTL` window.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `mark_update_notified(latest: str)`

Record that the user was notified about version *latest*.

Additional notes from the source docstring:

```text
Writes into the shared update state file so a subsequent
`should_notify_update` call can suppress duplicate banners.

Args:
    latest: The version string that was shown.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `clear_update_notified()`

Clear the "already notified" marker so the update modal re-opens next launch.

Additional notes from the source docstring:

```text
Removes both `notified_at` and `notified_version` from the shared
update state file.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `is_update_available(*, bypass_cache: bool=False)`

Check whether a newer version of deepagents-cli is available.

Additional notes from the source docstring:

```text
When the installed version is a pre-release (e.g. `0.0.35a1`),
pre-release versions on PyPI are included in the comparison so alpha
testers are notified of newer alphas and the eventual stable release.
Stable installs only compare against stable PyPI releases.

Args:
    bypass_cache: Skip the cache and always hit PyPI.

Returns:
    A `(available, latest)` tuple.

        `latest` is the PyPI version string when it was fetched and parsed
        successfully, or `None` when the PyPI check itself fails (network
        error, unparseable response, or non-PEP 440 installed version).
        `available` is `True` only when `latest` is strictly newer than
        the installed version. Callers can therefore distinguish "already
        up to date" (`(False, "1.2.3")`) from "could not reach PyPI"
        (`(False, None)`).
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `detect_install_method()`

Detect how `deepagents-cli` was installed.

Additional notes from the source docstring:

```text
Checks `sys.prefix` against known paths for uv and Homebrew.

Returns:
    The detected install method: `'uv'`, `'brew'`, `'pip'`, or `'unknown'`
        (editable/dev installs).
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `upgrade_command(method: InstallMethod | None=None)`

Return the shell command to upgrade `deepagents-cli`.

Additional notes from the source docstring:

```text
Falls back to the pip command for unrecognized install methods.

Args:
    method: Install method override.

        Auto-detected if `None`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `perform_upgrade()`

Attempt to upgrade `deepagents-cli` using the detected install method.

Additional notes from the source docstring:

```text
Only tries the detected method — does not fall back to other package
managers to avoid cross-environment contamination.

Returns:
    `(success, output)` — *output* is the combined stdout/stderr.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `is_update_check_enabled()`

Return whether update checks are enabled.

Additional notes from the source docstring:

```text
Checks `DEEPAGENTS_CLI_NO_UPDATE_CHECK` env var and the `[update].check` key
in `config.toml`.

Defaults to enabled.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `is_auto_update_enabled()`

Return whether auto-update is enabled.

Additional notes from the source docstring:

```text
Opt-in via `DEEPAGENTS_CLI_AUTO_UPDATE=1` env var or
`[update].auto_update = true` in `config.toml`.

Defaults to `False`.

Always disabled for editable installs.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `set_auto_update(enabled: bool)`

Persist the auto-update preference to `config.toml`.

Additional notes from the source docstring:

```text
Writes `[update].auto_update` so the setting survives across sessions.

Args:
    enabled: Whether auto-update should be enabled.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_read_update_config()`

Read `[update]` section from `config.toml`.

Additional notes from the source docstring:

```text
Returns:
    A dict of boolean config values, empty on missing/unreadable file.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `get_seen_version()`

Return the last version the user saw the "what's new" banner for.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `mark_version_seen(version: str)`

Record that the user has seen the "what's new" banner for *version*.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `should_show_whats_new()`

Return `True` if this is the first launch on a newer version.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
