# `libs/cli/deepagents_cli/deploy/config.py`

> Deploy configuration parsing and validation.

## Position in the system

This file belongs to the `deepagents deploy` path. The deploy package reads a project layout, validates `deepagents.toml`, bundles prompts, memories, skills, MCP config, optional frontend assets, and emits files that `langgraph deploy` can run.

## Functions and classes

### `AgentConfig`

`[agent]` section — core agent identity.

Methods worth reading inside this class:

- `__post_init__(self)`: This helper performs one narrow step for the surrounding module while keeping normalization, error handling, or side-effect policy in one place.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `SubAgentConfig`

Parsed from a subagent's deepagents.toml.

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `SubAgentProject`

A discovered subagent directory with its parsed config.

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `SandboxConfig`

`[sandbox]` section — sandbox provider settings.

Additional notes from the source docstring:

```text
The whole section is optional. When omitted (or `provider = "none"`)
the runtime falls back to an in-process `StateBackend` and tools
like `execute` become no-ops.
```

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `AuthConfig`

`[auth]` section — authentication provider settings.

Additional notes from the source docstring:

```text
The whole section is optional. When omitted, no `auth.py` is
generated and LangSmith Cloud's default `x-api-key` auth applies
(callers still need a LangSmith API key to reach the deployment).
To make the API genuinely open — e.g., to expose the bundled
`[frontend]` without sign-in — set `provider = "anonymous"`
explicitly.
```

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `MemoriesConfig`

`[memories]` section — backing store for `/memories/`.

Additional notes from the source docstring:

```text
`backend = "hub"` (default) routes `/memories/` through a
`ContextHubBackend` bound to a LangSmith Hub agent repo, giving
persistent, git-like storage. `backend = "store"` routes it through a
`StoreBackend` against the LangGraph runtime store.

`identifier` overrides the Hub agent repo. When omitted, it defaults
to `-/{agent.name}` at bundle time.

`agent_writable` controls whether the agent can write to `/memories/`.
When `False` (default), agent memory is read-only and only user memories
under `/memories/user/` are writable. When `True`, the agent can write
anywhere under `/memories/`.
```

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `FrontendConfig`

`[frontend]` section — bundled default frontend settings.

Additional notes from the source docstring:

```text
When `enabled = True`, `deepagent deploy` copies a pre-built React
chat UI into the deployment alongside the agent. An `[auth]`
section is required in this case — pick `"supabase"` or `"clerk"`
for real per-user auth, or set `provider = "anonymous"` explicitly
to ship the UI with an open API.
```

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `DeployConfig`

Top-level deploy configuration parsed from `deepagents.toml`.

Methods worth reading inside this class:

- `validate(self, project_root: Path)`: Validate config against the filesystem.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `_validate_mcp_for_deploy(mcp_path: Path)`

Validate that MCP config only uses http/sse transports (no stdio).

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_parse_subagent_config(data: dict[str, Any], subagent_dir: Path)`

Parse a subagent's deepagents.toml into a `SubAgentConfig`.

Additional notes from the source docstring:

```text
Raises:
    ValueError: If the config has disallowed sections, missing required
        fields, or unknown keys.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `load_subagents(project_root: Path)`

Discover and load subagent projects from `subagents/`.

Additional notes from the source docstring:

```text
Returns a dict keyed by subagent name. If the `subagents/` directory
does not exist or is empty, returns an empty dict.

Raises:
    ValueError: On any structural or config validation error.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `load_config(config_path: Path)`

Load and parse a `deepagents.toml` file.

Additional notes from the source docstring:

```text
Raises:
    FileNotFoundError: If the config file does not exist.
    ValueError: If the config is missing required fields or has an
        unknown top-level section.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_parse_config(data: dict[str, Any])`

Parse raw TOML dict into a `DeployConfig`.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_validate_model_credentials(model: str)`

Check that the API key env var is set for the model provider.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_validate_sandbox_credentials(provider: str)`

Check that at least one required API key env var is set for the provider.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_validate_auth_credentials(provider: str)`

Check that all required env vars are set for the auth provider.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_validate_hub_credentials()`

Check that a LangSmith key is set when `[memories].backend = 'hub'`.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_validate_frontend_credentials(provider: str)`

Check that all extra env vars are set for the frontend bundle.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `find_config(start_path: Path | None=None)`

Find `deepagents.toml` in *start_path* (or cwd if not given).

Additional notes from the source docstring:

```text
Only checks the single directory — does not walk parent directories.

Returns the path if found, or `None` otherwise.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `generate_starter_config()`

Generate a starter `deepagents.toml` template.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `generate_starter_agents_md()`

Generate a starter `AGENTS.md` template.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `generate_starter_env()`

Generate a starter `.env` template.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `generate_starter_mcp_json()`

Generate a starter `mcp.json` template.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `generate_starter_skill_md()`

Generate a starter `skills/review/SKILL.md` for code review.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Deploy helpers usually write files, read environment variables, or shell out through command helpers. Keep validation errors explicit because deployment failures otherwise surface late inside LangGraph Cloud tooling.
