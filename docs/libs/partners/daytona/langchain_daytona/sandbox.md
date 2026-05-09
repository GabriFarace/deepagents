# `partners/daytona/langchain_daytona/sandbox.py`

> Daytona sandbox backend implementation.

## Position in the system

This partner package implements or supports a BackendProtocol-compatible sandbox. The core SDK can use these classes through its backend abstraction without depending directly on the provider.

## Imports and module-level state

This file imports `__future__, time, collections.abc, typing, uuid, daytona, daytona, deepagents.backends.protocol, deepagents.backends.sandbox`.

## Functions and classes

### `DaytonaSandbox`

Daytona sandbox implementation conforming to SandboxBackendProtocol. This implementation inherits all file operation methods from BaseSandbox and only implements the execute() method using Daytona's API. This class inherits from `BaseSandbox` and is the main object for this part of the module.

#### `DaytonaSandbox.__init__(self, *, sandbox: daytona.Sandbox, timeout: int=30 * 60, sync_polling_interval: SyncPollingInterval=0.1)`

Create a backend wrapping an existing Daytona sandbox. Args: sandbox: Existing Daytona sandbox instance to wrap. timeout: Default command timeout in seconds used when `execute()` is called without an explicit `timeout`. sync_polling_interval: Delay in seconds between polling Daytona for command completion on the sync execution path, or a callable that receives elapsed execution time in seconds and returns the next polling delay. This will eventually only appear on the sync path once an optimized async implementation is available. Key arguments are `sandbox`, `timeout`, `sync_polling_interval`. It mutates `self._sandbox`, `self._default_timeout`, `self._sync_polling_interval`. Internally it delegates to `callable`, `cast`. It runs synchronously in the caller and returns directly.

#### `DaytonaSandbox.id(self)`

Return the Daytona sandbox id. It does not keep durable module state; effects come from returned values or delegated calls. It runs synchronously in the caller and returns directly.

#### `DaytonaSandbox.execute(self, command: str, *, timeout: int | None=None)`

Execute a shell command inside the sandbox. Args: command: Shell command string to execute. timeout: Maximum time in seconds to wait for the command to complete. If None, uses the backend's default timeout. Note that in Daytona's implementation, a timeout of 0 means "wait indefinitely". Key arguments are `command`, `timeout`. It does not keep durable module state; effects come from returned values or delegated calls. It runs synchronously in the caller and returns directly.

#### `DaytonaSandbox.download_files(self, paths: list[str])`

Download files from the sandbox. Key arguments are `paths`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `download_files`, `iter`, `enumerate`, `append`, `next`, `startswith`. It runs synchronously in the caller and returns directly.

#### `DaytonaSandbox.upload_files(self, files: list[tuple[str, bytes]])`

Upload files into the sandbox. Key arguments are `files`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `append`, `upload_files`, `startswith`, `FileUpload`, `FileUploadResponse`. It runs synchronously in the caller and returns directly.

## Gotchas

Most behavior is delegated through framework protocols, so preserve method signatures when editing even if an argument appears unused locally.
