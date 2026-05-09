# `libs/cli/deepagents_cli/integrations/sandbox_factory.py`

> Sandbox lifecycle management with provider abstraction.

## Position in the system

This file is part of sandbox integration plumbing. The CLI uses these adapters to create or verify backend sandboxes while keeping provider-specific SDK details behind a common interface.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from rich.markup import escape as escape_markup`

- `from deepagents_cli.config import console, get_glyphs`

- `from deepagents_cli.integrations.sandbox_provider import SandboxNotFoundError, SandboxProvider`


## Functions and classes

### `_run_sandbox_setup(backend: SandboxBackendProtocol, setup_script_path: str)`

Run users setup script in sandbox with env var expansion.

Additional notes from the source docstring:

```text
Args:
    backend: Sandbox backend instance
    setup_script_path: Path to setup script file

Raises:
    FileNotFoundError: If the setup script does not exist.
    RuntimeError: If the setup script fails to execute.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `create_sandbox(provider: str, *, sandbox_id: str | None=None, setup_script_path: str | None=None)`

Create or connect to a sandbox of the specified provider.

Additional notes from the source docstring:

```text
This is the unified interface for sandbox creation using the
provider abstraction.

Args:
    provider: Sandbox provider (`'agentcore'`, `'daytona'`, `'langsmith'`,
        `'modal'`, `'runloop'`)
    sandbox_id: Optional existing sandbox ID to reuse
    setup_script_path: Optional path to setup script to run after sandbox starts

Yields:
    `SandboxBackendProtocol` instance
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_get_available_sandbox_types()`

Get list of available sandbox provider types (internal).

Additional notes from the source docstring:

```text
Returns:
    List of available sandbox provider type names
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `get_default_working_dir(provider: str)`

Get the default working directory for a given sandbox provider.

Additional notes from the source docstring:

```text
Args:
    provider: Sandbox provider name (`'agentcore'`, `'daytona'`, `'langsmith'`,
        `'modal'`, `'runloop'`)

Returns:
    Default working directory path as string

Raises:
    ValueError: If provider is unknown
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_import_provider_module(module_name: str, *, provider: str, package: str)`

Import an optional provider module with a provider-specific error message.

Additional notes from the source docstring:

```text
Args:
    module_name: Python module name to import.
    provider: Sandbox provider name (e.g. `'daytona'`).
    package: PyPI package name exposed by the CLI extra.

Returns:
    The imported module object.

Raises:
    ImportError: If the optional dependency is not installed.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_LangSmithProvider`

LangSmith sandbox provider implementation.

Additional notes from the source docstring:

```text
Manages LangSmith sandbox lifecycle using the LangSmith SDK, booting
sandboxes from snapshots built from a Docker image.
```

Methods worth reading inside this class:

- `get_or_create(self, *, sandbox_id: str | None=None, timeout: int=180, snapshot: str | None=None, snapshot_image: str | None=None, fs_capacity_bytes: int | None=None, **kwargs: Any)`: Get existing or create new LangSmith sandbox.

- `delete(self, *, sandbox_id: str, **kwargs: Any)`: Delete a LangSmith sandbox.

- `_ensure_snapshot(self, snapshot_name: str, image: str, fs_capacity_bytes: int)`: Resolve a snapshot by name, building it from `image` if missing.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `_DaytonaProvider`

Daytona sandbox provider — lifecycle management for Daytona sandboxes.

Methods worth reading inside this class:

- `get_or_create(self, *, sandbox_id: str | None=None, timeout: int=180, **kwargs: Any)`: Get or create a Daytona sandbox.

- `delete(self, *, sandbox_id: str, **kwargs: Any)`: Delete a Daytona sandbox by id.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `_ModalProvider`

Modal sandbox provider — lifecycle management for Modal sandboxes.

Methods worth reading inside this class:

- `get_or_create(self, *, sandbox_id: str | None=None, timeout: int=180, **kwargs: Any)`: Get or create a Modal sandbox.

- `delete(self, *, sandbox_id: str, **kwargs: Any)`: Terminate a Modal sandbox by id.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `_RunloopProvider`

Runloop sandbox provider — lifecycle management for Runloop devboxes.

Methods worth reading inside this class:

- `get_or_create(self, *, sandbox_id: str | None=None, timeout: int=180, **kwargs: Any)`: Get or create a Runloop devbox.

- `delete(self, *, sandbox_id: str, **kwargs: Any)`: Shut down a Runloop devbox by id.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `_AgentCoreProvider`

AgentCore Code Interpreter sandbox provider.

Additional notes from the source docstring:

```text
Manages AgentCore session lifecycle. Sessions cannot be reconnected after
the CLI exits — the `sandbox_id` parameter is not supported.
```

Methods worth reading inside this class:

- `get_or_create(self, *, sandbox_id: str | None=None, **kwargs: Any)`: Create a new AgentCore Code Interpreter session.

- `delete(self, *, sandbox_id: str, **kwargs: Any)`: Stop an AgentCore session.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `_get_provider(provider_name: str)`

Get a `SandboxProvider` instance for the specified provider (internal).

Additional notes from the source docstring:

```text
Args:
    provider_name: Name of the provider (`'agentcore'`, `'daytona'`, `'langsmith'`,
        `'modal'`, `'runloop'`)

Returns:
    `SandboxProvider` instance

Raises:
    ValueError: If `provider_name` is unknown.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `verify_sandbox_deps(provider: str)`

Check that the required packages for a sandbox provider are installed.

Additional notes from the source docstring:

```text
Uses `importlib.util.find_spec` for a lightweight check with no actual
imports. Call this in the CLI process *before* spawning the server
subprocess so users get a clear, actionable error instead of an opaque
server crash.

Args:
    provider: Sandbox provider name (e.g. `'daytona'`).

Raises:
    ImportError: If the provider's backend package is not installed.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
