# `libs/cli/deepagents_cli/extras_info.py`

> Inspect optional-dependency install status for the running distribution.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Functions and classes

### `ExtraDependencyStatus`

Install status for one optional dependency extra.

Methods worth reading inside this class:

- `ready(self)`: Return whether all declared packages for this extra are installed.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `_extract_extra_name(marker_str: str)`

Pull the extra name out of a marker like `extra == "anthropic"`.

Additional notes from the source docstring:

```text
Args:
    marker_str: String form of a `packaging.markers.Marker`.

Returns:
    The quoted extra name, or `None` when the marker does not carry an
        `extra == "..."` clause.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `get_extras_status(distribution_name: str='deepagents-cli')`

Return installed optional dependencies grouped by extra.

Additional notes from the source docstring:

```text
Reads `Requires-Dist` metadata from the named distribution, groups the
entries gated by `extra == "..."` markers under their extra name, and
resolves each package's installed version via `importlib.metadata`.
Packages that are not installed are omitted; extras whose entire
package list is absent are dropped.

Composite meta-extras that only bundle other extras (see
`_COMPOSITE_EXTRAS`) and self-references to the distribution itself
are skipped — their components already appear under their own extras.

Args:
    distribution_name: Name of the installed distribution to inspect.

Returns:
    Mapping from extra name to a sorted list of `(package, version)`
        tuples for packages that are currently installed. An empty
        mapping is returned when the distribution itself is not found.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `get_optional_dependency_status(distribution_name: str='deepagents-cli')`

Return installed and missing optional dependencies grouped by extra.

Additional notes from the source docstring:

```text
Args:
    distribution_name: Name of the installed distribution to inspect.

Returns:
    Sorted tuple of optional extra statuses. An empty tuple is returned
        when the distribution itself is not found.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `format_extras_status_plain(status: ExtrasStatus)`

Render an `ExtrasStatus` mapping as column-aligned plain text.

Additional notes from the source docstring:

```text
Suitable for stdout in non-interactive contexts (e.g. the `--version`
CLI flag) where a markdown renderer is unavailable.

Args:
    status: Mapping returned by `get_extras_status`.

Returns:
    Multi-line string with a heading and one `extra  package  version`
        row per installed package.

        Returns an empty string when `status` is empty.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `format_extras_status(status: ExtrasStatus)`

Render an `ExtrasStatus` mapping as a markdown fragment.

Additional notes from the source docstring:

```text
Args:
    status: Mapping returned by `get_extras_status`.

Returns:
    Multi-line markdown string containing a heading and a pipe table
        with `Extra`, `Package`, and `Version` columns, suitable for
        rendering via a markdown widget.

        Returns an empty string when `status` is empty.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
