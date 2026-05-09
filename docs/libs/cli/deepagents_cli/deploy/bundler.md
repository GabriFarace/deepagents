# `libs/cli/deepagents_cli/deploy/bundler.py`

> Bundle a deepagents project for deployment.

## Position in the system

This file belongs to the `deepagents deploy` path. The deploy package reads a project layout, validates `deepagents.toml`, bundles prompts, memories, skills, MCP config, optional frontend assets, and emits files that `langgraph deploy` can run.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from deepagents_cli.deploy.config import AGENTS_MD_FILENAME, MCP_FILENAME, SKILLS_DIRNAME, USER_DIRNAME, DeployConfig, SubAgentProject, load_subagents`

- `from deepagents_cli.deploy.templates import APP_PY_TEMPLATE, AUTH_BLOCKS, AUTH_ON_HANDLER, DEPLOY_GRAPH_TEMPLATE, MCP_TOOLS_TEMPLATE, PYPROJECT_TEMPLATE, SANDBOX_BLOCKS, SYNC_SUBAGENTS_TEMPLATE`


## Functions and classes

### `_build_runtime_config_json(config: DeployConfig)`

Build the JSON value injected into `window.__DEEPAGENTS_CONFIG__`.

Additional notes from the source docstring:

```text
Only reached when `[frontend].enabled` and `[auth]` is set —
validation guarantees both. The `is None` guards below exist so
the optional fields narrow for type-checkers.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_copy_frontend_dist(config: DeployConfig, build_dir: Path)`

Copy the pre-built bundle into build_dir and rewrite the config placeholder.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `bundle(config: DeployConfig, project_root: Path, build_dir: Path)`

Create the full deployment bundle in *build_dir*.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_build_subagent_seed(subagent: SubAgentProject)`

Build the seed entry for a single sync subagent.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_build_seed(project_root: Path, system_prompt: str)`

Build the `_seed.json` payload.

Additional notes from the source docstring:

```text
Layout::

    {
        "memories":       { "/AGENTS.md": "..." },
        "skills":         { "/<skill>/SKILL.md": "...", ... },
        "user_memories":  { "/AGENTS.md": "..." }
    }

`memories` and `skills` are read-only at runtime.
`user_memories` contains a single writable `AGENTS.md` mounted at
`/memories/user/`, namespaced per user_id.  If the project has a
`user/` directory (even if empty), an `AGENTS.md` is always seeded.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_render_deploy_graph(config: DeployConfig, *, mcp_present: bool, has_user_memories: bool=False, has_sync_subagents: bool=False)`

Render the generated `deploy_graph.py`.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_render_auth_py(provider: str)`

Render the generated `auth.py` for the given auth provider.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_render_langgraph_json(*, env_present: bool, auth_present: bool=False, frontend_present: bool=False)`

Render `langgraph.json` — adds `"env"`, `"auth"`, `"http"` when applicable.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_render_pyproject(config: DeployConfig, *, mcp_present: bool, subagent_model_providers: list[str] | None=None, has_subagent_mcp: bool=False)`

Render the deployment package's `pyproject.toml`.

Additional notes from the source docstring:

```text
Deps are inferred — the user never writes them. We add:

- the LangChain partner package matching the model provider prefix
- `langchain-mcp-adapters` if `mcp.json` is present
- the sandbox partner package (daytona/modal/runloop)
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `print_bundle_summary(config: DeployConfig, build_dir: Path)`

Print a human-readable summary of what was bundled.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Deploy helpers usually write files, read environment variables, or shell out through command helpers. Keep validation errors explicit because deployment failures otherwise surface late inside LangGraph Cloud tooling.
