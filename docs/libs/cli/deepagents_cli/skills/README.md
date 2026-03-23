# `skills/` — Skill Management

This directory implements the skill management CLI commands (`deepagents skills list/create/info/delete`) and the skill discovery system for loading skills from multiple filesystem locations.

## Overview

**Skills** are reusable, composable agent capabilities defined as Markdown files with YAML frontmatter. They can be built-in (shipped with the package), user-level (`~/.deepagents/{agent}/skills/`), or project-level (`.deepagents/skills/`). Skills are loaded into the agent via `SkillsMiddleware` at startup.

## Files

| File | Purpose |
|---|---|
| `__init__.py` | Public API: re-exports `execute_skills_command` and `setup_skills_parser` |
| `commands.py` | Argparse setup and command execution for `skills list/create/info/delete` |
| `load.py` | `list_skills()` — multi-directory skill discovery with precedence ordering |

## Precedence Order

Skills are loaded from multiple directories in increasing precedence. When two skills have the same name, the higher-precedence one wins:

| Priority | Directory | Source Label |
|---|---|---|
| 0 (lowest) | `<package>/built_in_skills/` | `built-in` |
| 1 | `~/.deepagents/{agent}/skills/` | `user` |
| 2 | `~/.agents/skills/` | `user` (alias) |
| 3 | `.deepagents/skills/` | `project` |
| 4 | `.agents/skills/` | `project` (alias) |
| 5 | `~/.claude/skills/` | `claude (experimental)` |
| 6 (highest) | `.claude/skills/` | `claude (experimental)` |

## Skill File Format

```markdown
---
name: my-skill
description: Brief description
---

# Skill system prompt content here
```

## Relationship to SDK

`load.py` wraps `deepagents.middleware.skills._list_skills` with a `FilesystemBackend` to perform the actual file reading. The `ExtendedSkillMetadata` class extends the SDK's `SkillMetadata` with a `source` field for CLI display.

For agent-time skill execution, use `deepagents.middleware.skills.SkillsMiddleware` directly.
