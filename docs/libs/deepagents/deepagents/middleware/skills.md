# `deepagents/middleware/skills.py`

## High-Level Purpose

`SkillsMiddleware` implements Anthropic's agent skills pattern with progressive disclosure. It loads skill metadata from backend storage and injects a skills catalog into the system prompt, enabling the agent to discover and use available skills on demand.

## Key Concepts

- **Progressive disclosure:** The agent sees each skill's name and description in the system prompt, but reads the full `SKILL.md` instructions only when needed (via `read_file`).
- **Sources with precedence:** Multiple source directories are loaded in order. Later sources override earlier ones for skills with the same name ("last one wins"), enabling base → user → project → team skill layering.
- **Backend-agnostic:** Works with any backend implementation (filesystem, state, store, etc.).
- **Once per session:** Skills are loaded in `before_agent` and cached in `SkillsState` (private state attribute). Subsequent turns skip the load.

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
---

# Web Research Skill
...
```

## Constants

| Constant | Value | Purpose |
|---|---|---|
| `MAX_SKILL_FILE_SIZE` | `10 * 1024 * 1024` | Max SKILL.md size to prevent DoS (10 MB) |
| `MAX_SKILL_NAME_LENGTH` | `64` | Agent Skills spec constraint |
| `MAX_SKILL_DESCRIPTION_LENGTH` | `1024` | Agent Skills spec constraint |
| `MAX_SKILL_COMPATIBILITY_LENGTH` | `500` | Agent Skills spec constraint |

## TypedDicts

### `SkillMetadata`
Parsed skill metadata per the [Agent Skills specification](https://agentskills.io/specification).

| Field | Type | Description |
|---|---|---|
| `path` | `str` | Path to the SKILL.md file |
| `name` | `str` | Skill identifier (1-64 chars, lowercase alphanumeric + hyphens) |
| `description` | `str` | What the skill does (1-1024 chars) |
| `license` | `str \| None` | License name or reference |
| `compatibility` | `str \| None` | Environment requirements |
| `metadata` | `dict[str, str]` | Arbitrary key-value extension data |
| `allowed_tools` | `list[str]` | Tool names the skill recommends using |

### `SkillsState(AgentState)`
State schema with `skills_metadata: list[SkillMetadata]` as a `PrivateStateAttr` (not propagated to parent agents).

### `SkillsStateUpdate`
State update shape: `{"skills_metadata": list[SkillMetadata]}`.

## Module-Level Functions

### `_validate_skill_name(name, directory_name) -> tuple[bool, str]`
Validates skill name against the Agent Skills spec: lowercase alphanumeric + hyphens, no leading/trailing hyphens or `--`, must match parent directory name. Returns `(is_valid, error_message)`.

### `_parse_skill_metadata(content, skill_path, directory_name) -> SkillMetadata | None`
Parses `SKILL.md` content. Extracts YAML frontmatter between `---` delimiters using `yaml.safe_load`. Validates name and description. Truncates oversized description/compatibility fields with warnings. Returns `None` if parsing fails.

### `_validate_metadata(raw, skill_path) -> dict[str, str]`
Ensures `metadata` field is a `dict[str, str]`. Rejects non-dict values with a warning.

### `_format_skill_annotations(skill) -> str`
Builds a `"License: X, Compatibility: Y"` annotation string from optional skill fields.

### `_skill_metadata_from_response(response, skill_dir_path, skill_md_path) -> SkillMetadata | None`
Decodes a `SKILL.md` download response into `SkillMetadata`. Logs a warning for any unexpected failure (parse error, invalid name, unreadable bytes) so silently dropped skills surface in logs. Returns `None` for `file_not_found` responses (not every subdirectory is a skill) without logging.

### `_list_skills(backend, source_path) -> list[SkillMetadata]`
Synchronous skill discovery. Scans `source_path` via `backend.ls()`, finds subdirectories, downloads their `SKILL.md` files in a batch, parses each one. Backend paths are normalized via `to_posix_path()` before being passed to `PurePosixPath` to handle Windows-native backslash paths (fixes skill name validation and system prompt path rendering on Windows).

### `_alist_skills(backend, source_path) -> list[SkillMetadata]`
Async version using `backend.als()` and `backend.adownload_files()`. Same Windows path normalization as `_list_skills`.

### `SKILLS_SYSTEM_PROMPT`
Template injected into the system prompt. Includes skill locations, skill listing with paths, and detailed instructions for the progressive disclosure workflow (recognize → read full instructions → follow → use helper scripts).

## Class: `SkillsMiddleware(AgentMiddleware[SkillsState, ContextT, ResponseT])`

### Constructor

```python
SkillsMiddleware(
    *,
    backend: BACKEND_TYPES,
    sources: list[str],
)
```

**Parameters:**
- `backend` — Backend instance or factory. Use `lambda rt: StateBackend(rt)` for `StateBackend`.
- `sources` — List of skill directory paths (e.g., `["/skills/user/", "/skills/project/"]`).

### Methods

#### `before_agent(state, runtime, config) -> SkillsStateUpdate | None`
Synchronous. Loads skills from all sources in order (later sources override earlier). Returns `None` if `skills_metadata` is already in state.

#### `abefore_agent(state, runtime, config) -> SkillsStateUpdate | None`
Async version.

#### `modify_request(request) -> ModelRequest`
Formats skill locations and skill list from state, builds the skills section, and appends to the system message.

#### `wrap_model_call` / `awrap_model_call`
Calls `modify_request` and forwards to handler.

#### `_format_skills_locations() -> str`
Formats source paths for display: `"**User Skills**: /skills/user/ (higher priority)"`. Source paths are normalized via `to_posix_path()` before display to handle Windows backslash paths.

#### `_format_skills_list(skills) -> str`
Formats skill metadata as a bullet list with name, description, annotations, and the path to read full instructions.

## Dependencies

- `yaml` — YAML frontmatter parsing
- `pathlib.PurePosixPath` — cross-platform path construction
- `langchain.agents.middleware.types` — `AgentMiddleware`, `PrivateStateAttr`
- `deepagents.backends.protocol` — `LsResult`, `BACKEND_TYPES`, `FileDownloadResponse`, `FILE_NOT_FOUND`
- `deepagents.backends.utils.to_posix_path` — normalizes Windows backslash paths before `PurePosixPath` processing
- `deepagents.middleware._utils.append_to_system_message`
