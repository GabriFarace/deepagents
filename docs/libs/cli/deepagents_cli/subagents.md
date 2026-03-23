# `subagents.py`

## High-Level Purpose

This module loads custom subagent definitions from the filesystem. Subagents are defined as Markdown files with YAML frontmatter in an `agents/` directory structure. They allow users to extend the main agent with specialized sub-agents that can be delegated tasks.

## Directory Structure

```
~/.deepagents/agents/{subagent_name}/AGENTS.md   (user-level)
.deepagents/agents/{subagent_name}/AGENTS.md      (project-level)
```

## File Format

```markdown
---
name: researcher
description: Research topics on the web before writing content
model: anthropic:claude-haiku-4-5-20251001
---

You are a research assistant with access to web search.

## Your Process
1. Search for relevant information
2. Summarize findings clearly
```

- `name` (required): Unique identifier used with the task tool.
- `description` (required): What this subagent does. The main agent uses this to decide when to delegate.
- `model` (optional): Model override in `provider:model-name` format.
- Body: Becomes the `system_prompt`.

## Classes

### `SubagentMetadata`

**Type:** `TypedDict`

Metadata for a custom subagent loaded from the filesystem.

| Key | Type | Description |
|---|---|---|
| `name` | `str` | Unique identifier for the subagent |
| `description` | `str` | What this subagent does |
| `system_prompt` | `str` | Instructions for the subagent (markdown body) |
| `model` | `str \| None` | Optional model override in `provider:model` format |
| `source` | `str` | Where this subagent was loaded from (`'user'` or `'project'`) |
| `path` | `str` | Absolute path to the subagent definition file |

## Functions

### `_parse_subagent_file(file_path: Path) -> SubagentMetadata | None`

Parses a single subagent Markdown file with YAML frontmatter.

**Parameters:**
- `file_path`: Path to the Markdown file.

**Returns:** `SubagentMetadata` if parsing succeeds and all required fields are valid, `None` otherwise.

**Validation:**
- File must have `---` delimited YAML frontmatter.
- `name` and `description` must be non-empty strings.
- `model` must be a string if present.

### `_load_subagents_from_dir(agents_dir: Path, source: str) -> dict[str, SubagentMetadata]`

Loads all subagents from a directory. Expects structure: `agents_dir/{subagent_name}/AGENTS.md`.

**Parameters:**
- `agents_dir`: Directory containing subagent subdirectories.
- `source`: Source identifier (`'user'` or `'project'`).

**Returns:** Dict mapping subagent name to metadata.

### `list_subagents(*, user_agents_dir=None, project_agents_dir=None) -> list[SubagentMetadata]`

Loads and merges subagents from user-level and project-level directories. Project-level subagents override user-level subagents with the same name.

**Parameters:**
- `user_agents_dir`: Path to `~/.deepagents/agents/`. If `None`, uses default.
- `project_agents_dir`: Path to `.deepagents/agents/`. If `None`, uses project discovery.

**Returns:** Merged list of `SubagentMetadata` dicts. Empty if no agents found.

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `re` | stdlib | YAML frontmatter extraction via regex |
| `yaml` | `pyyaml` | YAML frontmatter parsing |
