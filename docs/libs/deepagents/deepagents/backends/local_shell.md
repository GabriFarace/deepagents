# `deepagents/backends/local_shell.py`

## High-Level Purpose

`LocalShellBackend` extends `FilesystemBackend` with unrestricted local shell command execution via `subprocess.run(..., shell=True)`. It combines direct filesystem access with arbitrary shell execution on the host machine — providing no sandboxing, isolation, or resource limits.

> **Security Warning:** This backend runs commands directly on the host system with your user's full permissions. Use only in trusted local development environments with Human-in-the-Loop middleware. Never use in production, multi-tenant systems, or when processing untrusted input.

## Constants

### `DEFAULT_EXECUTE_TIMEOUT`
`int = 120` — Default timeout in seconds for shell command execution. Exported from the module.

## Key Characteristics

- **Inherits `FilesystemBackend`:** All file operations (ls, read, write, edit, grep, glob, upload/download) come from `FilesystemBackend`.
- **Adds `execute()`:** Runs commands via `subprocess.run(shell=True)`.
- **`virtual_mode` does NOT restrict shell:** Commands can access any path on the system regardless of `virtual_mode` or `root_dir` settings.
- **Configurable environment:** Can start with an empty env, inherit from `os.environ`, or specify custom variables.
- **Output truncation:** Combines stdout/stderr with `[stderr]` prefixes; truncates at `max_output_bytes`.

## Dependencies

- `deepagents.backends.filesystem.FilesystemBackend`
- `deepagents.backends.protocol.SandboxBackendProtocol`, `ExecuteResponse`
- `subprocess`, `os`, `uuid`, `warnings` — standard library

## Class: `LocalShellBackend(FilesystemBackend, SandboxBackendProtocol)`

### Constructor

```python
LocalShellBackend(
    root_dir: str | Path | None = None,
    *,
    virtual_mode: bool | None = None,
    timeout: int = DEFAULT_EXECUTE_TIMEOUT,
    max_output_bytes: int = 100_000,
    env: dict[str, str] | None = None,
    inherit_env: bool = False,
)
```

**Parameters:**
- `root_dir` — Working directory for both filesystem ops and shell commands. Defaults to `cwd`.
- `virtual_mode` — Virtual path mode for file operations (see `FilesystemBackend`). Does NOT restrict shell commands.
- `timeout` — Default per-command timeout in seconds. Must be positive; defaults to 120.
- `max_output_bytes` — Maximum bytes of combined output to capture. Excess is truncated. Default: 100,000.
- `env` — Environment variables for shell commands.
- `inherit_env` — If `True`, inherits `os.environ` and applies `env` overrides on top. If `False`, only `env` is available.

**Key Attributes:**
- `self._default_timeout: int`
- `self._max_output_bytes: int`
- `self._env: dict[str, str]`
- `self._sandbox_id: str` — `"local-{random_hex}"`, generated at init.

### Properties

#### `id -> str`
Returns `self._sandbox_id`.

### Methods

#### `execute(command: str, *, timeout: int | None = None) -> ExecuteResponse`

Runs a shell command using `subprocess.run(command, shell=True, cwd=self.cwd, ...)`.

**Parameters:**
- `command` — Shell command string. Passed directly to `/bin/sh` (or OS equivalent).
- `timeout` — Per-command timeout override in seconds. Must be positive if provided.

**Returns:** `ExecuteResponse` with:
- `output` — Combined stdout/stderr. Stderr lines are prefixed with `[stderr] `. Non-zero exit code appends `"\n\nExit code: {code}"`.
- `exit_code` — Process return code. Uses `124` for timeout.
- `truncated` — `True` if output exceeded `max_output_bytes`.

**Error handling:**
- Empty or non-string command → returns `ExecuteResponse(exit_code=1, ...)`
- `subprocess.TimeoutExpired` → returns timeout message with `exit_code=124`
- All other exceptions → returns error message with `exit_code=1`
