# `skills/load.py`

## High-Level Purpose

This module provides filesystem-based skill discovery for CLI operations (list, create, info, delete). It wraps the prebuilt middleware functionality from `deepagents.middleware.skills` and adapts it for direct filesystem access needed by CLI commands.

For middleware usage within agents, use `deepagents.middleware.skills.SkillsMiddleware` directly.

## Classes

### `ExtendedSkillMetadata`

**Inherits from:** `deepagents.middleware.skills.SkillMetadata`

Extended skill metadata for CLI display. Adds a `source` attribute for tracking where the skill came from.

| Attribute | Type | Description |
|---|---|---|
| `source` | `Literal["built-in", "user", "project", "claude (experimental)"]` | Origin of the skill |

## Functions

### `list_skills(...) -> list[ExtendedSkillMetadata]`

Lists skills from all configured directories. CLI-specific wrapper around the prebuilt middleware's skill loading functionality.

**Parameters (all optional keyword-only):**
| Parameter | Description |
|---|---|
| `built_in_skills_dir` | Path to built-in skills (`<package>/built_in_skills/`) |
| `user_skills_dir` | Path to `~/.deepagents/{agent}/skills/` |
| `project_skills_dir` | Path to `.deepagents/skills/` |
| `user_agent_skills_dir` | Path to `~/.agents/skills/` (alias) |
| `project_agent_skills_dir` | Path to `.agents/skills/` (alias) |
| `user_claude_skills_dir` | Path to `~/.claude/skills/` (experimental) |
| `project_claude_skills_dir` | Path to `.claude/skills/` (experimental) |

**Precedence order (lowest to highest):**
0. `built_in_skills_dir`
1. `user_skills_dir`
2. `user_agent_skills_dir`
3. `project_skills_dir`
4. `project_agent_skills_dir`
5. `user_claude_skills_dir`
6. `project_claude_skills_dir`

Skills from higher-precedence directories override same-name skills from lower-precedence ones.

**Returns:** Merged list of `ExtendedSkillMetadata` instances.

### `load_skill_content(skill_path: Path) -> str`

Reads the full content of a skill file.

**Returns:** File content string.

## Re-exports

| Export | Description |
|---|---|
| `SkillMetadata` | Base skill metadata class from the SDK |
| `list_skills` | The skill discovery function above |
| `load_skill_content` | Skill file content reader |

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `FilesystemBackend` | `deepagents.backends.filesystem` | File system access for the middleware |
| `SkillMetadata`, `_list_skills` | `deepagents.middleware.skills` | SDK skill loading |
| `__version__` | `deepagents_cli._version` | CLI version for metadata |
