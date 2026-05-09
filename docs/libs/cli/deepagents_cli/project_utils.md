# `libs/cli/deepagents_cli/project_utils.py`

> Utilities for project root detection and project-specific configuration.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from deepagents_cli._env_vars import SERVER_ENV_PREFIX`

- `from deepagents_cli._git import find_git_root`


## Functions and classes

### `ProjectContext`

Explicit user/project path context for project-sensitive behavior.

Additional notes from the source docstring:

```text
Attributes:
    user_cwd: Authoritative working directory from the CLI invocation.
    project_root: Resolved project root for `user_cwd`, if one exists.
```

Methods worth reading inside this class:

- `__post_init__(self)`: Validate that path fields are absolute.

- `from_user_cwd(cls, user_cwd: str | Path)`: Build a project context from an explicit user working directory.

- `resolve_user_path(self, path: str | Path)`: Resolve a path relative to the explicit user working directory.

- `project_agent_md_paths(self)`: Return project-level `AGENTS.md` files for this context.

- `project_skills_dir(self)`: Return the project `.deepagents/skills` directory, if any.

- `project_agents_dir(self)`: Return the project `.deepagents/agents` directory, if any.

- `project_agent_skills_dir(self)`: Return the project `.agents/skills` directory, if any.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `get_server_project_context(env: Mapping[str, str] | None=None)`

Read the server project context from environment transport data.

Additional notes from the source docstring:

```text
Args:
    env: Environment mapping to read from.

Returns:
    Reconstructed project context, or `None` if no server context exists.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `find_project_root(start_path: str | Path | None=None)`

Find the project root by looking for git metadata.

Additional notes from the source docstring:

```text
Args:
    start_path: Directory to start searching from.
        Defaults to current working directory.

Returns:
    Path to the project root if found, None otherwise.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `find_project_agent_md(project_root: Path)`

Find project-specific AGENTS.md file(s).

Additional notes from the source docstring:

```text
Checks two locations and returns ALL that exist:
1. project_root/.deepagents/AGENTS.md
2. project_root/AGENTS.md

Both files will be loaded and combined if both exist.

Candidates with symlinked path components are followed only when the
resolved target stays inside `project_root`. The returned `Path` is the
resolved target when any symlink component was traversed, so
`FilesystemBackend.download_files` opens a regular file rather than
tripping `O_NOFOLLOW` on the link itself. Symlinks pointing outside the
project root, symlink loops, and unreadable parents are skipped with a
warning. Broken symlinks are treated as missing files (no warning),
matching the pre-existing behavior for absent candidates.

Why: project AGENTS.md is auto-discovered and loaded into the system
prompt before the first model call. Without the in-tree check, a
malicious clone could ship `AGENTS.md -> ~/.ssh/config` (or any other
locally-readable file) and have its contents injected as agent
instructions on first run.

Args:
    project_root: Path to the project root directory.

Returns:
    Existing AGENTS.md paths, with in-tree symlinked path components
        pre-resolved to their targets. Empty if neither file exists, one
        entry if only one is present, or two entries if both locations
        have the file.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
