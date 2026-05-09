# `libs/cli/deepagents_cli/_git.py`

> Lightweight git metadata helpers for CLI state detection.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Functions and classes

### `_abbreviate_git_ref(ref: str)`

Convert a full git ref into a short display name.

Additional notes from the source docstring:

```text
Args:
    ref: Full git ref name from repository metadata.

Returns:
    The abbreviated ref name suitable for display.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_parse_git_dir_pointer(git_entry: Path)`

Resolve a `.git` file containing a `gitdir:` pointer.

Additional notes from the source docstring:

```text
Args:
    git_entry: `.git` file to parse.

Returns:
    The resolved git directory path, or `None` if the file is not a valid
    gitdir pointer.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_normalize_lookup_path(path: str | Path)`

Normalize a lookup path for git metadata discovery.

Additional notes from the source docstring:

```text
Args:
    path: Directory or file path inside a repository.

Returns:
    A normalized absolute path when possible, or the expanded path if full
    resolution fails.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_find_git_dir_uncached(path: Path)`

Locate the effective git metadata directory without using caches.

Additional notes from the source docstring:

```text
Args:
    path: Normalized directory or file path inside a repository.

Returns:
    The git metadata directory for the repository containing `path`, or
    `None` when no repository can be identified.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `find_git_dir(path: str | Path)`

Locate the effective git metadata directory for a path.

Additional notes from the source docstring:

```text
Args:
    path: Directory or file path inside a repository.

Returns:
    The git metadata directory for the repository containing `path`, or
    `None` when no repository can be identified.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `find_git_root(path: str | Path)`

Locate the repository root for a path.

Additional notes from the source docstring:

```text
Args:
    path: Directory or file path inside a repository.

Returns:
    The repository root containing `path`, or `None` when no repository can
    be identified.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `read_git_branch_from_filesystem(path: str | Path)`

Read the current git branch from repository metadata.

Additional notes from the source docstring:

```text
Args:
    path: Directory or file path inside a repository.

Returns:
    The abbreviated branch name, `HEAD` for detached HEAD, an empty string
    when `path` is not inside a git repository, or `None` when metadata
    exists but cannot be parsed confidently.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `read_git_branch_via_subprocess(path: str | Path)`

Fall back to `git rev-parse` for unusual repository layouts.

Additional notes from the source docstring:

```text
Args:
    path: Directory or file path inside a repository.

Returns:
    The branch name reported by git, or an empty string on failure.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `resolve_git_branch(path: str | Path)`

Resolve the current git branch with a filesystem-first strategy.

Additional notes from the source docstring:

```text
Args:
    path: Directory or file path inside a repository.

Returns:
    The current branch name, `HEAD` for detached HEAD, or an empty string
    when no branch can be determined.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
