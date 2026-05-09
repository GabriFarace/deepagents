# `libs/cli/deepagents_cli/skills/commands.py`

> CLI commands for skill management.

## Position in the system

This file supports the CLI skill system: discovering `SKILL.md` files, presenting them in slash/CLI commands, and wrapping a selected skill body into the first user message so the agent follows those instructions.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from deepagents_cli import theme`


## Functions and classes

### `_validate_name(name: str)`

Validate name per Agent Skills spec.

Additional notes from the source docstring:

```text
Requirements (https://agentskills.io/specification):
- Max 64 characters
- Unicode lowercase alphanumeric and hyphens only
- Cannot start or end with hyphen
- No consecutive hyphens
- No path traversal sequences

Unicode lowercase alphanumeric means any character where
`c.isalpha() and c.islower()` or `c.isdigit()` returns `True`,
which covers accented Latin characters (e.g., `'cafe'`,
`'uber-tool'`) and other scripts.  This matches the SDK's
`_validate_skill_name` implementation.

Args:
    name: The name to validate.

Returns:
    Tuple of (is_valid, error_message). If valid, error_message is empty.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_validate_skill_path(skill_dir: Path, base_dir: Path)`

Validate that the resolved skill directory is within the base directory.

Additional notes from the source docstring:

```text
Args:
    skill_dir: The skill directory path to validate
    base_dir: The base skills directory that should contain skill_dir

Returns:
    Tuple of (is_valid, error_message). If valid, error_message is empty.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_format_info_fields(skill: SkillMetadata)`

Extract non-empty optional metadata fields for display.

Additional notes from the source docstring:

```text
The upstream `_parse_skill_metadata` normalises empty/whitespace license
and compatibility values to `None`, so the truthy checks below are
sufficient.

Args:
    skill: Skill metadata to extract display fields from.

Returns:
    Ordered list of (label, value) tuples for non-empty fields.
        Fields appear in order: License, Compatibility, Allowed Tools,
        Metadata.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_list(agent: str, *, project: bool=False, output_format: OutputFormat='text')`

List all available skills for the specified agent.

Additional notes from the source docstring:

```text
Args:
    agent: Agent identifier for skills (default: agent).
    project: If True, show only project skills.
        If False, show all skills (user + project).
    output_format: Output format — `'text'` (Rich) or `'json'`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_generate_template(skill_name: str)`

Generate a `SKILL.md` template for a new skill.

Additional notes from the source docstring:

```text
The template follows the Agent Skills spec
(https://agentskills.io/specification) and the skill-creator guidance:

- Description includes "when to use" trigger information (not the body)
- Body contains only instructions loaded after the skill triggers

Args:
    skill_name: Name of the skill (used in frontmatter and heading).

Returns:
    Complete `SKILL.md` content with YAML frontmatter and markdown body.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_create(skill_name: str, agent: str, project: bool=False, *, output_format: OutputFormat='text')`

Create a new skill with a template SKILL.md file.

Additional notes from the source docstring:

```text
Args:
    skill_name: Name of the skill to create.
    agent: Agent identifier for skills
    project: If True, create in project skills directory.
        If False, create in user skills directory.
    output_format: Output format — `'text'` (Rich) or `'json'`.

Raises:
    SystemExit: If the skill name is invalid or the directory cannot be created.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_info(skill_name: str, *, agent: str='agent', project: bool=False, output_format: OutputFormat='text')`

Show detailed information about a specific skill.

Additional notes from the source docstring:

```text
Args:
    skill_name: Name of the skill to show info for.
    agent: Agent identifier for skills (default: agent).
    project: If True, only search in project skills.
        If False, search in both user and project skills.
    output_format: Output format — `'text'` (Rich) or `'json'`.

Raises:
    SystemExit: If the skill is not found or not in a project directory.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_delete(skill_name: str, *, agent: str='agent', project: bool=False, force: bool=False, dry_run: bool=False, output_format: OutputFormat='text')`

Delete a skill directory after validation and optional user confirmation.

Additional notes from the source docstring:

```text
Validates the skill name, locates the skill in user or project directories,
confirms the deletion with the user (unless `force` is `True`), and
recursively removes the skill directory.

Args:
    skill_name: Name of the skill to delete.
    agent: Agent identifier for skills.
    project: If `True`, only search in project skills.

        If `False`, search in both user and project skills.
    force: If `True`, skip confirmation prompt.
    dry_run: If `True`, print what would be removed without deleting.
    output_format: Output format — `'text'` (Rich) or `'json'`.

Raises:
    SystemExit: If the deletion fails or a safety check is violated.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `setup_skills_parser(subparsers: Any, *, make_help_action: Callable[[Callable[[], None]], type[argparse.Action]], add_output_args: Callable[[argparse.ArgumentParser], None] | None=None)`

Setup the skills subcommand parser with all its subcommands.

Additional notes from the source docstring:

```text
Each subcommand gets a dedicated help screen so that
`deepagents skills -h` shows skills-specific help, not the
global help.

Args:
    subparsers: The parent subparsers object to add the skills parser to.
    make_help_action: Factory that accepts a zero-argument help
        callable and returns an argparse Action class wired to it.
    add_output_args: Optional hook to add a shared `--json` flag.

Returns:
    The skills subparser for argument handling.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `execute_skills_command(args: argparse.Namespace)`

Execute skills subcommands based on parsed arguments.

Additional notes from the source docstring:

```text
Args:
    args: Parsed command line arguments with skills_command attribute

Raises:
    SystemExit: If the agent name is invalid.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
