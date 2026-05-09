# `libs/deepagents/deepagents/middleware/skills.py`

> Skills middleware. It scans backend skill directories for `SKILL.md`
> metadata, stores a private catalog in state, and injects a progressive
> disclosure skills guide into the system prompt.

## Position in the system

Skills are optional, reusable instructions stored as directories with a
`SKILL.md` file. This middleware lists configured backend sources before the
agent run, parses each skill's YAML frontmatter, and exposes only metadata to
the model. The model then reads the full skill file with filesystem tools when
the skill applies.

```
source dirs
  └─ skill-name/SKILL.md
       └─ before_agent(): parse metadata
            └─ state["skills_metadata"]
                 └─ wrap_model_call(): append skills list + usage workflow
```

Later sources override earlier sources by skill name, enabling base/user/team
or project layering.

## Imports and module-level state

The module uses `yaml.safe_load()` for frontmatter, `PurePosixPath` and
`to_posix_path()` for backend path normalization, `html`/`json` for safe
warning rendering, and backend result types for listing/downloading skill
files.

Important constants include max `SKILL.md` file size, warning count/length
limits, and Agent Skills metadata constraints for name, description, and
compatibility fields. `_MODULE_EXTENSIONS` lists supported JavaScript/TypeScript
entrypoint extensions for experimental module metadata.

## Functions and classes

### `SkillSource`

Type alias for either a bare source path or `(path, label)`. Bare paths derive
a display label from the path. Tuple labels are rendered in the prompt as
`**{label} Skills**`.

### `_validate_tuple_source(source)`

Raises `TypeError` unless a tuple source is exactly `(str, str)`. This catches
bad constructor inputs early instead of allowing confusing prompt output or
later index errors.

### `_source_path(source)`

Returns the path portion of a source. Bare strings are returned directly;
tuples are validated and return element zero.

### `_truncate_skill_load_warning(error)`

Caps a warning string before it is inserted into the model prompt. Overlong
warnings are cut and suffixed with `"... [truncated]"`.

### `_derive_source_label(source)`

Returns the human-readable source label. Tuples provide their label directly.
Bare paths use the final path component with special handling for
`built_in_skills` (`Built-in`) and `.../skills` paths, where the parent name is
used so `.claude/skills` renders as `Claude` rather than `Skills Skills`.

### `SkillMetadata`

TypedDict for parsed skill metadata. Required fields include `path`, `name`,
`description`, `license`, `compatibility`, `metadata`, and `allowed_tools`.
`module` is optional and experimental; consumers such as QuickJS middleware may
use it to install a JS/TS module, but this middleware only validates and stores
the path.

### `SkillsState`

Extends `AgentState` with private `skills_metadata` and `skills_load_errors`.
Private state keeps the catalog and diagnostics out of public final output
while keeping them available for prompt rendering.

### `SkillsStateUpdate`

State update returned by sync and async loading hooks. It always includes
`skills_metadata`; it includes `skills_load_errors` only when one or more
source-level loading errors occurred.

### `_validate_skill_name(name, directory_name)`

Checks the Agent Skills name constraints: non-empty, at most 64 characters,
lowercase alphanumeric or single hyphens, no leading/trailing/consecutive
hyphens, and equal to the parent directory name. It returns
`(is_valid, error_message)` and supports Unicode lowercase letters via Python's
`isalpha()`/`islower()` checks.

Invalid names currently warn but do not prevent loading, preserving backward
compatibility.

### `_parse_skill_metadata(content, skill_path, directory_name)`

Parses and validates the YAML frontmatter at the top of a `SKILL.md` file. It
rejects files over 10 MB, missing frontmatter, invalid YAML, non-mapping
frontmatter, and missing required `name`/`description`.

The function truncates long descriptions and compatibility strings, parses
`allowed-tools` from a space-delimited string with comma tolerance, validates
optional `module`, normalizes optional `metadata`, and returns a
`SkillMetadata` dict. Parse failures return `None` after logging warnings.

### `_validate_module_path(raw, skill_path)`

Validates the optional `module` frontmatter key. It accepts non-empty relative
paths ending in known JS/TS extensions, strips a leading `./`, and rejects
absolute paths or paths that escape the skill directory. Invalid module values
are warnings, not fatal parse errors.

### `_validate_metadata(raw, skill_path)`

Normalizes the optional metadata field to `dict[str, str]`. Non-dict metadata
is ignored with a warning when truthy; dict keys and values are coerced to
strings.

### `_format_skill_annotations(skill)`

Builds the parenthetical prompt annotation from optional `license` and
`compatibility` fields. If neither is set, returns an empty string.

### `_skill_metadata_from_response(response, skill_dir_path, skill_md_path)`

Converts a backend `FileDownloadResponse` into `SkillMetadata`. Expected
`file_not_found` misses are silently skipped because not every subdirectory is
a skill. Other backend errors, missing content, non-UTF-8 content, and parse
failures are logged and return `None`.

### `_format_skills_source_error(source_path, error)`

Formats a recoverable source-level loading error as a prompt/log string:
`Cannot load skills from '{source_path}': {error}`.

### `_list_skills_with_errors(backend, source_path)`

Lists a source directory, selects child directories, downloads each
`SKILL.md`, parses metadata, and returns `(skills, source_error)`. If listing
the source returns an `LsResult` with an error, the error is logged and returned
but any returned entries are still processed.

### `_list_skills(backend, source_path)`

Compatibility helper that returns only the skills list from
`_list_skills_with_errors()`, discarding the source error.

### `_alist_skills_with_errors(backend, source_path)` and `_alist_skills(backend, source_path)`

Async versions of the source scanner. They use `backend.als()` and
`backend.adownload_files()` but otherwise share the same parsing and error
semantics.

### `SkillsMiddleware`

Loads skill metadata from backend sources and injects the model-visible skills
catalog. The constructor stores the backend, creates a paths-only
`self.sources` list for backward compatibility, creates aligned
`self.source_labels`, and stores `SKILLS_SYSTEM_PROMPT` as the prompt template.

#### `_get_backend(state, runtime, config)`

Resolves either a direct backend instance or a backend factory. Factory
resolution uses a synthetic `ToolRuntime`, matching other middleware factory
patterns. A `None` backend result raises `AssertionError`.

#### `_format_skills_locations()`

Renders configured source locations for the system prompt. The final source is
marked `(higher priority)` because later sources override earlier sources when
skill names collide.

#### `_format_skills_list(skills)`

Renders the available skills list. Each skill shows name, description,
optional license/compatibility annotations, optional allowed tools, and the
absolute path to read for full instructions. If there are no skills, the prompt
tells the model where skills could be created.

#### `_format_skills_load_warnings(errors)`

Renders recoverable source loading errors inside `<skill_load_warnings>` tags.
Warnings are JSON-encoded and HTML-escaped so diagnostics are explicitly
untrusted text, not instructions. At most 20 warnings are shown; extras are
summarized.

#### `modify_request(request)`

Reads `skills_metadata` and `skills_load_errors` from state, renders
locations/list/warnings into `SKILLS_SYSTEM_PROMPT`, appends the resulting
section to the system message, and returns an overridden request.

#### `before_agent(state, runtime, config)` and `abefore_agent(state, runtime, config)`

Load skills once per state. If `skills_metadata` is already present, they
return `None`. Otherwise they scan sources in order, collect source errors,
and insert each parsed skill into a dict keyed by skill name, so later sources
win. They return `SkillsStateUpdate` with the final list and any source errors.

#### `wrap_model_call(request, handler)` and `awrap_model_call(request, handler)`

Inject the formatted skills section into the system prompt and delegate to the
next model-call handler. These hooks do not re-scan sources; scanning happens
only in the before-agent hooks.

## System prompts and tool descriptions

`SKILLS_SYSTEM_PROMPT`:

```text

## Skills System

You have access to a skills library that provides specialized capabilities and domain knowledge.

{skills_locations}{skills_load_warnings}

**Available Skills:**

{skills_list}

**How to Use Skills (Progressive Disclosure):**

Skills follow a **progressive disclosure** pattern - you see their name and description above, but only read full instructions when needed:

1. **Recognize when a skill applies**: Check if the user's task matches a skill's description
2. **Read the skill's full instructions**: Use `read_file` on the path shown in the skill list above.
   Pass `limit=1000` since the default of 100 lines is too small for most skill files.
3. **Follow the skill's instructions**: SKILL.md contains step-by-step workflows, best practices, and examples
4. **Access supporting files**: Skills may include helper scripts, configs, or reference docs - use absolute paths

**When to Use Skills:**
- User's request matches a skill's domain (e.g., "research X" -> web-research skill)
- You need specialized knowledge or structured workflows
- A skill provides proven patterns for complex tasks

**Executing Skill Scripts:**
Skills may contain Python scripts or other executable files. Always use absolute paths from the skill list.

**Example Workflow:**

User: "Can you research the latest developments in quantum computing?"

1. Check available skills -> See "web-research" skill with its path
2. Read the full skill file: `read_file(path, limit=1000)`
3. Follow the skill's research workflow (search -> organize -> synthesize)
4. Use any helper scripts with absolute paths

Remember: Skills make you more capable and consistent. When in doubt, check if a skill exists for the task!
```

This middleware exposes no tools of its own. The prompt assumes `read_file` is
available through filesystem middleware.

## Flow walk-through

1. `before_agent()` resolves the backend and scans each configured source.
2. Each child directory is probed for `SKILL.md`; parseable files produce
   `SkillMetadata`.
3. Metadata is stored privately in state, with later sources replacing earlier
   skills by name.
4. `wrap_model_call()` renders source locations, warnings, and metadata into
   the system prompt.
5. The model reads a full skill file only when its description matches the task.

## Gotchas

Invalid skill names warn but still load. This keeps older skill directories
working but means prompt-visible skill names may not be spec-clean.

`allowed_tools` is informational here. This middleware parses and displays it;
it does not enforce tool access.
