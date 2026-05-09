# `libs/deepagents/deepagents/backends/local_shell.py`

> Local host backend that combines direct filesystem access with unrestricted
> shell command execution.

## Position in the system

`LocalShellBackend` is the backend shape expected by local coding-agent
workflows: it inherits all `FilesystemBackend` file methods and implements
`SandboxBackendProtocol.execute()`. `CompositeBackend` can use it as the
default backend when command execution should run in the project directory.

## Imports and module-level state

The module imports `subprocess`, `os.environ`, and `uuid` for local process
execution, environment construction, and per-instance IDs. It re-exports
`DEFAULT_EXECUTE_TIMEOUT = 120` and `LocalShellBackend` through `__all__`.

## Functions and classes

### `LocalShellBackend(root_dir=None, *, virtual_mode=None, timeout=DEFAULT_EXECUTE_TIMEOUT, max_output_bytes=100_000, env=None, inherit_env=False)`

The constructor validates a positive default timeout, handles the deprecated
`virtual_mode=None` default, initializes `FilesystemBackend`, then stores shell
execution settings. `inherit_env=False` means commands run with only the
explicit `env` mapping; `inherit_env=True` copies the parent environment and
then applies any overrides.

It also creates a unique `local-...` ID for protocols and tools that want to
label the execution environment. The filesystem `root_dir` is used as the
working directory for commands, but it does not constrain what shell commands
can access.

#### `id`

The `id` property returns the generated sandbox identifier. It is stable for
the backend instance and has the form `local-{random_hex}`.

#### `execute(command, *, timeout=None)`

`execute()` runs the command with `subprocess.run(shell=True)`, captures stdout
and stderr, combines them into one output string, and returns an
`ExecuteResponse`. Stderr lines are prefixed with `[stderr]`, non-zero exit
codes are appended to the output, and oversized output is truncated according
to `max_output_bytes`.

Empty or non-string commands return a normal `ExecuteResponse` with exit code
1. A per-call timeout overrides the constructor timeout and must be positive.
Timeouts return exit code `124`; other execution exceptions are caught and
returned as error output instead of being raised.

## Gotchas

This backend has no sandboxing. `virtual_mode=True` only affects inherited file
methods; `execute()` can still run arbitrary commands against the host machine
with the configured environment and working directory.
