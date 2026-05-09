# `libs/cli/deepagents_cli/skills/load.py`

> Skill loader for CLI commands.

## Position in the system

This file supports the CLI skill system: discovering `SKILL.md` files, presenting them in slash/CLI commands, and wrapping a selected skill body into the first user message so the agent follows those instructions.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from deepagents.backends.filesystem import FilesystemBackend`

- `from deepagents.middleware.skills import SkillMetadata, _list_skills as list_skills_from_backend`

- `from deepagents_cli._version import __version__ as _cli_version`


## Functions and classes

### `ExtendedSkillMetadata`

Extended skill metadata for CLI display, adds source tracking.

Additional notes from the source docstring:

```text
Attributes:
    source: Origin of the skill. One of `'built-in'`, `'user'`, `'project'`,
        or `'claude (experimental)'`.
```

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `list_skills(*, built_in_skills_dir: Path | None=None, user_skills_dir: Path | None=None, project_skills_dir: Path | None=None, user_agent_skills_dir: Path | None=None, project_agent_skills_dir: Path | None=None, user_claude_skills_dir: Path | None=None, project_claude_skills_dir: Path | None=None)`

List skills from built-in, user, and/or project directories.

Additional notes from the source docstring:

```text
This is a CLI-specific wrapper around the prebuilt middleware's skill loading
functionality. It uses FilesystemBackend to load skills from local directories.

Precedence order (lowest to highest):
0. `built_in_skills_dir` (`<package>/built_in_skills/`)
1. `user_skills_dir` (`~/.deepagents/{agent}/skills/`)
2. `user_agent_skills_dir` (`~/.agents/skills/`)
3. `project_skills_dir` (`.deepagents/skills/`)
4. `project_agent_skills_dir` (`.agents/skills/`)
5. `user_claude_skills_dir` (`~/.claude/skills/`, experimental)
6. `project_claude_skills_dir` (`.claude/skills/`, experimental)

Skills from higher-precedence directories override those with the same name.

Args:
    built_in_skills_dir: Path to built-in skills shipped with the package.
    user_skills_dir: Path to `~/.deepagents/{agent}/skills/`.
    project_skills_dir: Path to `.deepagents/skills/`.
    user_agent_skills_dir: Path to `~/.agents/skills/` (alias).
    project_agent_skills_dir: Path to `.agents/skills/` (alias).
    user_claude_skills_dir: Path to `~/.claude/skills/` (experimental).
    project_claude_skills_dir: Path to `.claude/skills/` (experimental).

Returns:
    Merged list of skill metadata from all sources, with higher-precedence
        directories taking priority when names conflict.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `load_skill_content(skill_path: str, *, allowed_roots: Sequence[Path]=())`

Read the full raw SKILL.md content for a skill.

Additional notes from the source docstring:

```text
Returns the complete file content including any YAML frontmatter.
Callers are responsible for parsing or stripping frontmatter if needed.

When `allowed_roots` is provided, the resolved path must fall within at
least one root directory. This prevents symlink traversal from reading files
outside known skill directories.

Args:
    skill_path: Path to the SKILL.md file (from `SkillMetadata['path']`).
    allowed_roots: Skill root directories the resolved path must be
        contained within.

        Callers must pre-resolve these via `Path.resolve()` — the resolved
        skill path is compared directly, so un-resolved roots cause false
        containment failures.

        If empty, containment is not checked.

Returns:
    Full text content of the SKILL.md file, or `None` on read failure.

Raises:
    PermissionError: If the resolved path is outside all `allowed_roots`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
