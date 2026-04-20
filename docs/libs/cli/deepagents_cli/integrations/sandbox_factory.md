# `integrations/sandbox_factory.py`

## High-Level Purpose

This module handles sandbox lifecycle management with provider abstraction. It is responsible for:

- Mapping sandbox type names to their default working directories
- Running user-provided setup scripts inside sandboxes
- Loading the correct provider module dynamically based on the `--sandbox` argument
- Creating, setting up, and cleaning up sandbox backends throughout the CLI session

## Module-Level Constants

### `_PROVIDER_TO_WORKING_DIR`

Maps sandbox provider names to their default working directories:

| Provider | Working Directory |
|---|---|
| `agentcore` | `/tmp` |
| `daytona` | `/home/daytona` |
| `langsmith` | `/tmp` |
| `modal` | `/workspace` |
| `runloop` | `/home/user` |

### LangSmith snapshot constants

| Constant | Value | Description |
|---|---|---|
| `_LANGSMITH_DEFAULT_SNAPSHOT` | `"deepagents-cli"` | Default snapshot name when none is specified |
| `_LANGSMITH_DEFAULT_IMAGE` | *(Docker image string)* | Default Docker image for building snapshots |
| `_LANGSMITH_DEFAULT_FS_CAPACITY_BYTES` | `16 GiB` | Default filesystem capacity for new snapshots |

## Functions

### `get_default_working_dir(sandbox_type: str) -> str | None`

Returns the default working directory for a given sandbox type.

**Parameters:**
- `sandbox_type`: One of the supported sandbox types.

**Returns:** Working directory path string, or `None` for unknown types.

### `_run_sandbox_setup(backend: SandboxBackendProtocol, setup_script_path: str) -> None`

Runs a user-provided setup script inside the sandbox after creation.

**Parameters:**
- `backend`: The sandbox backend instance.
- `setup_script_path`: Path to the setup script file.

**Process:**
1. Reads the script content.
2. Expands `${VAR}` syntax using `string.Template.safe_substitute(os.environ)`.
3. Executes the expanded script via `bash -c` inside the sandbox.
4. Raises `RuntimeError` if the exit code is non-zero.

**Raises:**
- `FileNotFoundError`: If the setup script file doesn't exist.
- `RuntimeError`: If the setup script fails (non-zero exit code).

### `create_sandbox(sandbox_type: str, *, sandbox_id: str | None = None, setup_script: str | None = None, **provider_kwargs) -> tuple[SandboxBackendProtocol, str | None]`

Creates or retrieves a sandbox backend and optionally runs a setup script.

**Parameters:**
- `sandbox_type`: Type of sandbox (`"none"`, `"agentcore"`, `"modal"`, `"daytona"`, `"runloop"`, `"langsmith"`).
- `sandbox_id`: Existing sandbox ID to reuse.
- `setup_script`: Path to an optional setup script to run after creation.
- `**provider_kwargs`: Additional kwargs passed to the provider.

**Returns:** `(backend, sandbox_id)` where `sandbox_id` is the ID of the created/retrieved sandbox (or `None` for local sandboxes).

**Provider loading:** Uses `importlib.import_module` to load provider-specific modules dynamically, enabling optional extras without hard dependencies (e.g., `langchain-agentcore-codeinterpreter` only needed for `agentcore`).

**LangSmith provider — snapshot-based API:** The `_LangSmithProvider` now uses the snapshot-based LangSmith SDK API (`create_snapshot` / `create_sandbox(snapshot_id=...)`) instead of the old template API. The `get_or_create` flow:
1. Resolves a snapshot ID via env var `LANGSMITH_SANDBOX_SNAPSHOT_ID` (highest priority, skips name lookup).
2. Falls back to `LANGSMITH_SANDBOX_SNAPSHOT_NAME` env var, then the `snapshot` kwarg, then `_LANGSMITH_DEFAULT_SNAPSHOT`.
3. Calls `_ensure_snapshot(name, image, capacity)` to list existing snapshots by name. Only `status == "ready"` snapshots are accepted; a matching-name snapshot in a non-ready state raises rather than triggering a duplicate build.
4. If no matching snapshot exists, calls `create_snapshot` with the Docker image and filesystem capacity, blocking until the snapshot is ready.
5. Boots the sandbox from the resolved snapshot ID.

### `cleanup_sandbox(sandbox_type: str, sandbox_id: str, was_existing: bool = False) -> None`

Cleans up a sandbox after the session ends.

**Parameters:**
- `sandbox_type`: The sandbox type.
- `sandbox_id`: ID of the sandbox to clean up.
- `was_existing`: If `True`, skip cleanup (sandbox was pre-existing and should persist).

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `importlib` | stdlib | Dynamic provider module loading |
| `string.Template` | stdlib | `${VAR}` expansion in setup scripts |
| `rich.markup.escape` | rich | Markup escaping for console output |
| `console`, `get_glyphs` | `deepagents_cli.config` | Console output and glyphs |
| `SandboxProvider`, `SandboxNotFoundError` | `integrations.sandbox_provider` | Provider interface |
| `SandboxBackendProtocol` | `deepagents.backends.protocol` | Backend type hint |
