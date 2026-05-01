# `deepagents/middleware/skills.py`

## High-Level Purpose

`SkillsMiddleware` implements the Agent Skills pattern with progressive disclosure. It loads skill metadata from backend storage and injects a skills catalog into the system prompt, enabling the agent to discover and use available skills on demand.

## Key Concepts

- **Progressive disclosure:** The agent sees each skill's name and description in the system prompt, but reads the full `SKILL.md` instructions only when needed (via `read_file`).
- **Labelled sources with precedence:** Multiple source directories are loaded in order. Later sources override earlier ones for skills with the same name ("last one wins"), enabling base → user → project → team skill layering. Sources can be bare path strings or `(path, label)` tuples.
- **Backend-agnostic:** Works with any backend implementation (filesystem, state, store, etc.).
- **Once per session:** Skills are loaded in `before_agent` and cached in `SkillsState` (private state attribute). Subsequent turns skip the load.
- **Module field (JS integration):** The optional `module` field in `SKILL.md` frontmatter names a JS/TS entrypoint. `REPLMiddleware` (QuickJS) consumes this field to make skills dynamically importable as ES modules from JavaScript.

## Skill File Structure

```
/skills/user/web-research/
├── SKILL.md          # Required: YAML frontmatter + markdown instructions
└── helper.py         # Optional: supporting files
```

`SKILL.md` format:
```markdown
---
name: web-research
description: Structured approach to conducting thorough web research
license: MIT
module: index.ts
allowed-tools: read_file, grep
---

# Web Research Skill
...
```

## Constants

| Constant | Value | Purpose |
|---|---|---|
| `MAX_SKILL_FILE_SIZE` | `10 * 1024 * 1024` | Max SKILL.md size (10 MB) |
| `MAX_SKILL_NAME_LENGTH` | `64` | Agent Skills spec constraint |
| `MAX_SKILL_DESCRIPTION_LENGTH` | `1024` | Agent Skills spec constraint |
| `MAX_SKILL_COMPATIBILITY_LENGTH` | `500` | Agent Skills spec constraint |

## TypedDicts

### `SkillMetadata`

Parsed skill metadata per the [Agent Skills specification](https://agentskills.io/specification).

| Field | Type | Description |
|---|---|---|
| `path` | `str` | Path to the SKILL.md file |
| `name` | `str` | Skill identifier (1–64 chars, lowercase alphanumeric + hyphens) |
| `description` | `str` | What the skill does (1–1024 chars) |
| `license` | `str \| None` | License name or reference |
| `compatibility` | `str \| None` | Environment requirements (1–500 chars) |
| `metadata` | `dict[str, str]` | Arbitrary key-value extension data |
| `allowed_tools` | `list[str]` | Tool names the skill recommends using |
| `module` | `str \| None` | **New.** POSIX path to a JS/TS entrypoint (e.g., `"index.ts"`, `"lib/util.js"`). Must be relative, no `..` escapes, must end in `.js`, `.mjs`, `.cjs`, `.ts`, `.mts`, `.cts`, `.jsx`, or `.tsx`. Consumed by `REPLMiddleware` for dynamic ES module imports. |

### `SkillsState(AgentState)`
State schema with `skills_metadata: list[SkillMetadata]` as a `PrivateStateAttr` (not propagated to parent agents).

### `SkillsStateUpdate`
State update shape: `{"skills_metadata": list[SkillMetadata]}`.

## Module-Level Functions

### `_validate_skill_name(name, directory_name) -> tuple[bool, str]`
Validates skill name against the Agent Skills spec: lowercase alphanumeric + hyphens, no leading/trailing hyphens or `--`, must match parent directory name. Returns `(is_valid, error_message)`.

### `_validate_module_path(module) -> str | None`
Validates the `module` field from frontmatter. Rejects absolute paths, `..` traversals, and non-JS extensions. Returns the normalized path or `None` if invalid (with a warning log).

### `_parse_skill_metadata(content, skill_path, directory_name) -> SkillMetadata | None`
Parses `SKILL.md` content. Extracts YAML frontmatter between `---` delimiters. Validates name and description. Truncates oversized fields with warnings. Returns `None` if parsing fails.

### `_validate_metadata(raw, skill_path) -> dict[str, str]`
Ensures the `metadata` field is a `dict[str, str]`. Rejects non-dict values with a warning.

### `_format_skill_annotations(skill) -> str`
Builds a `"License: X, Compatibility: Y"` annotation string from optional skill fields.

### `_skill_metadata_from_response(response, skill_dir_path, skill_md_path) -> SkillMetadata | None`
Decodes a `SKILL.md` download response into `SkillMetadata`. Logs a warning for any unexpected failure so silently dropped skills surface in logs. Returns `None` for `file_not_found` (not every subdirectory is a skill).

### `_list_skills(backend, source_path) -> list[SkillMetadata]`
Synchronous skill discovery. Scans `source_path` via `backend.ls()`, finds subdirectories, downloads their `SKILL.md` files in a batch, parses each one. Paths normalized via `to_posix_path()` for Windows compatibility.

### `_alist_skills(backend, source_path) -> list[SkillMetadata]`
Async version using `backend.als()` and `backend.adownload_files()`.

### `SKILLS_SYSTEM_PROMPT`
Template injected into the system prompt. Includes skill locations (with labels), skill listing with paths, and detailed instructions for the progressive disclosure workflow.

## Class: `SkillsMiddleware(AgentMiddleware[SkillsState, ContextT, ResponseT])`

### Constructor

```python
SkillsMiddleware(
    *,
    backend: BACKEND_TYPES,
    sources: Sequence[str | tuple[str, str]],
)
```

**Parameters:**
- `backend` — Backend instance or factory. Use `lambda rt: StateBackend(rt)` for `StateBackend`.
- `sources` — Skill source paths. Each entry is either:
  - A bare path string: `"/skills/user/"` (label auto-derived from the path)
  - A `(path, label)` tuple: `("/skills/user/", "My Skills")` (explicit label)

**Attributes:**
- `sources` — `list[str]` view of paths only (backward-compatible).
- `source_labels` — `list[str]` of display labels aligned 1:1 with `sources`.

### Source Label Derivation (for bare paths)

| Path pattern | Derived label |
|---|---|
| `built_in_skills` | `Built-in` |
| `~/.claude/skills` or similar | Normalized parent (e.g., `Claude`) |
| `/skills/user/` | `User` (capitalize final component) |
| Root or empty | `Unnamed` |

### Methods

#### `before_agent(state, runtime, config) -> SkillsStateUpdate | None`
Loads skills from all sources in order (later sources override earlier). Returns `None` if `skills_metadata` is already in state.

#### `abefore_agent(state, runtime, config) -> SkillsStateUpdate | None`
Async version.

#### `modify_request(request) -> ModelRequest`
Formats skill locations (with labels) and skill list from state, builds the skills section, and appends to the system message.

#### `_format_skills_locations() -> str`
Formats source paths for display with their labels:
`"**User Skills**: /skills/user/ (higher priority)"`.

#### `_format_skills_list(skills) -> str`
Formats skill metadata as a bullet list with name, description, annotations, and path to full instructions.

## Dependencies

- `yaml` — YAML frontmatter parsing
- `pathlib.PurePosixPath` — cross-platform path construction
- `langchain.agents.middleware.types` — `AgentMiddleware`, `PrivateStateAttr`
- `deepagents.backends.protocol` — `LsResult`, `BACKEND_TYPES`, `FileDownloadResponse`, `FILE_NOT_FOUND`
- `deepagents.backends.utils.to_posix_path` — normalizes Windows backslash paths
- `deepagents.middleware._utils.append_to_system_message`
