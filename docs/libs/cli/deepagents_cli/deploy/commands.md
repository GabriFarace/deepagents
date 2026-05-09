# `libs/cli/deepagents_cli/deploy/commands.py`

> CLI commands for `deepagents init`, `dev`, and `deploy`.

## Position in the system

This file belongs to the `deepagents deploy` path. The deploy package reads a project layout, validates `deepagents.toml`, bundles prompts, memories, skills, MCP config, optional frontend assets, and emits files that `langgraph deploy` can run.

## Functions and classes

### `setup_deploy_parsers(subparsers: Any, *, make_help_action: Callable[[Callable[[], None]], type[argparse.Action]])`

Register the top-level `init`, `dev`, and `deploy` subparsers.

Additional notes from the source docstring:

```text
The three commands used to live under `deepagents deploy {init,dev}`
but are now flat: `deepagents init`, `deepagents dev`, and
`deepagents deploy`. This function registers all three on the root
subparsers object.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `execute_init_command(args: argparse.Namespace)`

Execute the `deepagents init` command.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `execute_dev_command(args: argparse.Namespace)`

Execute the `deepagents dev` command.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `execute_deploy_command(args: argparse.Namespace)`

Execute the `deepagents deploy` command.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_init_project(*, name: str, force: bool=False)`

Scaffold a deploy project folder.

Additional notes from the source docstring:

```text
Creates `name/` with the canonical layout:

```txt
<name>/
    deepagents.toml
    AGENTS.md
    .env
    mcp.json
    skills/
```

Args:
    name: Project folder name (created under cwd).
    force: Overwrite existing files if `True`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_deploy(config_path: str | None=None, dry_run: bool=False)`

Bundle and deploy the agent.

Additional notes from the source docstring:

```text
Args:
    config_path: Path to config file, or `None` for default.
    dry_run: If `True`, generate artifacts but don't deploy.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_dev(*, config_path: str | None, port: int, allow_blocking: bool)`

Bundle the project and run a local `langgraph dev` server.

Additional notes from the source docstring:

```text
The bundle is identical to what `deepagents deploy` would ship, just
served locally instead of pushed to LangGraph Platform. Hot-reloading
is provided by `langgraph dev` itself watching the build directory;
edits to the source project (`deepagents.toml`, skills, `AGENTS.md`)
require re-running `deepagents dev` to re-bundle.

Args:
    config_path: Path to `deepagents.toml`, or `None` for default.
    port: Local port for the dev server.
    allow_blocking: Pass `--allow-blocking` to `langgraph dev` so
        sync HTTP calls inside the graph (e.g. the LangSmith sandbox
        client) don't trigger blockbuster errors.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_seed_hub_repo(config: DeployConfig, build_dir: Path)`

Eagerly create the LangSmith Hub agent repo at bundle time.

Additional notes from the source docstring:

```text
Mirrors the per-(process, assistant_id) seeding that the generated
`deploy_graph.py` performs on first invocation, but runs it from the
CLI so the repo exists in LangSmith Hub the moment `deepagents deploy`
(or `deepagents dev`) returns from bundling. The runtime seed path
stays in place as a defensive no-op: it short-circuits via
`has_prior_commits()` once this seed has run.

Per-user hub repos (`{identifier}-user-{slug}`) are intentionally
*not* seeded here — user identities aren't known until authenticated
requests arrive at runtime.

Args:
    config: Loaded `DeployConfig`. Only invoked for hub-backed deploys;
        no-op otherwise.
    build_dir: Directory containing `_seed.json` written by `bundle()`.

Raises:
    SystemExit: If the hub commit fails or returns per-file errors —
        fail fast at bundle time rather than at first invocation.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_run_langgraph_deploy(build_dir: Path, *, name: str)`

Shell out to `langgraph deploy` in the build directory.

Additional notes from the source docstring:

```text
Args:
    build_dir: Directory containing generated deployment artifacts.
    name: Deployment name (passed as `--name` to avoid interactive prompt).

Raises:
    SystemExit: If `langgraph` CLI is not installed or deployment fails.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_resolve_langsmith_api_key()`

This resolver hides lookup order and fallback behavior behind one call.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_resolve_langsmith_endpoint()`

This resolver hides lookup order and fallback behavior behind one call.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_resolve_tracer_session_id_by_project_name(*, project_name: str, api_key: str)`

Resolve tracing project id (session id) by name.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_upsert_issues_board_config(*, session_id: str, api_key: str, context_hub_repo_handle: str)`

Best-effort create or patch board config for a deployed agent.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_resolve_context_hub_identifier(config: DeployConfig)`

This resolver hides lookup order and fallback behavior behind one call.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_resolve_context_hub_repo_handle_for_issues_board(config: DeployConfig)`

This resolver hides lookup order and fallback behavior behind one call.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_auto_wire_issues_board_if_hub(config: DeployConfig)`

Best-effort issues-board wiring after deploy for hub-backed memories.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Deploy helpers usually write files, read environment variables, or shell out through command helpers. Keep validation errors explicit because deployment failures otherwise surface late inside LangGraph Cloud tooling.
