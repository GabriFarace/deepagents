# `libs/cli/deepagents_cli/subagents.py`

> Subagent loader for CLI.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Functions and classes

### `SubagentMetadata`

Metadata for a custom subagent loaded from filesystem.

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `_parse_subagent_file(file_path: Path)`

Parse a subagent markdown file with YAML frontmatter.

Additional notes from the source docstring:

```text
The file must have YAML frontmatter (delimited by ---) containing at minimum
'name' and 'description' fields. The body of the file becomes the system_prompt.

Args:
    file_path: Path to the markdown file.

Returns:
    SubagentMetadata if parsing succeeds, None otherwise.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_load_subagents_from_dir(agents_dir: Path, source: str)`

Load subagents from a directory.

Additional notes from the source docstring:

```text
Expects structure: agents_dir/{subagent_name}/AGENTS.md

Args:
    agents_dir: Directory containing subagent folders.
    source: Source identifier ('user' or 'project').

Returns:
    Dict mapping subagent name to metadata.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `list_subagents(*, user_agents_dir: Path | None=None, project_agents_dir: Path | None=None)`

List subagents from user and/or project directories.

Additional notes from the source docstring:

```text
Scans for subagent definitions in the provided directories.
Project subagents override user subagents with the same name.

Args:
    user_agents_dir: Path to user-level agents directory.
    project_agents_dir: Path to project-level agents directory.

Returns:
    List of subagent metadata, with project subagents taking precedence.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
