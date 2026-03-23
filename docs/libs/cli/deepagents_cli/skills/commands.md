# `skills/commands.py`

## High-Level Purpose

This module implements the CLI commands for skill management. It provides argument parsing setup and execution for four skill management subcommands: `list`, `create`, `info`, and `delete`.

## Module-Level Constants

| Constant | Value | Description |
|---|---|---|
| `MAX_SKILL_NAME_LENGTH` | `64` | Maximum allowed length for skill names |

## Functions

### `_validate_name(name: str) -> tuple[bool, str]`

Validates a skill name per the Agent Skills specification.

**Requirements:**
- Max 64 characters
- Unicode lowercase alphanumeric characters and single hyphens only
- Cannot start or end with a hyphen
- No consecutive hyphens (`--`)
- No path traversal sequences (`..`, `/`, `\`)

Unicode lowercase alphanumeric includes any character where `c.isalpha() and c.islower()` or `c.isdigit()` returns `True` — covering accented Latin characters and other scripts.

**Parameters:**
- `name`: The name to validate.

**Returns:** `(is_valid, error_message)`. If valid, error_message is empty string.

### `setup_skills_parser(subparsers, make_help_action, add_output_args) -> None`

Registers the `skills` subparser and its subcommands into the main argparse parser.

**Registered subcommands:**

| Subcommand | Description |
|---|---|
| `skills list` | List all available skills |
| `skills create <name>` | Create a new skill |
| `skills info <name>` | Show information about a skill |
| `skills delete <name>` | Delete a skill |

**Parameters:**
- `subparsers`: The argparse subparsers action from `main.py`.
- `make_help_action`: Factory function for Rich-formatted help actions.
- `add_output_args`: Function to add `--json` output flag.

### `execute_skills_command(args: argparse.Namespace) -> None`

Executes the skills subcommand determined by `args.skills_command`.

**Routing:**
- `"list"` or `None` → `_list_skills_command(args)`
- `"create"` → `_create_skill_command(args)`
- `"info"` → `_info_skill_command(args)`
- `"delete"` → `_delete_skill_command(args)`

### `_list_skills_command(args) -> None`

Lists all available skills from built-in, user, and project directories. Outputs a Rich table or JSON depending on `args.output_format`.

**Columns:** name, description, source (built-in/user/project), location path.

### `_create_skill_command(args) -> None`

Creates a new skill directory and `skill.md` file. Validates the name, checks for conflicts, creates the skill template, and optionally opens the editor.

**Skill template structure:**
```markdown
---
name: <skill-name>
description: Brief description of what this skill does
---

# Skill instructions here
```

### `_info_skill_command(args) -> None`

Shows detailed information about a named skill: metadata from frontmatter, content preview, and file location.

### `_delete_skill_command(args) -> None`

Deletes a named skill directory (with confirmation prompt unless `--yes` is passed).

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `argparse` | stdlib | Argument parsing |
| `shutil`, `pathlib.Path` | stdlib | File operations |
| `theme` | `deepagents_cli.theme` | Brand colors for output |
| `ExtendedSkillMetadata`, `list_skills` | `skills.load` | Skill discovery |
