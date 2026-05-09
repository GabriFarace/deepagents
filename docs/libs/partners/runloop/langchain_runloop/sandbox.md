# `partners/runloop/langchain_runloop/sandbox.py`

> Runloop sandbox implementation.

## Position in the system

This partner package implements or supports a BackendProtocol-compatible sandbox. The core SDK can use these classes through its backend abstraction without depending directly on the provider.

## Imports and module-level state

This file imports `__future__, typing, deepagents.backends.protocol, deepagents.backends.sandbox`.

## Functions and classes

### `RunloopSandbox`

Sandbox backend that operates on a Runloop devbox. This class inherits from `BaseSandbox` and is the main object for this part of the module.

#### `RunloopSandbox.__init__(self, *, devbox: Devbox)`

Create a sandbox backend connected to an existing Runloop devbox. Key arguments are `devbox`. It mutates `self._devbox`, `self._devbox_id`, `self._default_timeout`. It runs synchronously in the caller and returns directly.

#### `RunloopSandbox.id(self)`

Return the devbox id. It does not keep durable module state; effects come from returned values or delegated calls. It runs synchronously in the caller and returns directly.

#### `RunloopSandbox.execute(self, command: str, *, timeout: int | None=None)`

Execute a shell command inside the devbox. Args: command: Shell command string to execute. timeout: Maximum time in seconds to wait for this command. If None, uses the backend's default timeout. Returns: ExecuteResponse containing output, exit code, and truncation flag. Key arguments are `command`, `timeout`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `exec`, `ExecuteResponse`, `stdout`, `stderr`. It runs synchronously in the caller and returns directly.

#### `RunloopSandbox.download_files(self, paths: list[str])`

Download files from the devbox. Key arguments are `paths`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `download`, `append`, `FileDownloadResponse`. It runs synchronously in the caller and returns directly.

#### `RunloopSandbox.upload_files(self, files: list[tuple[str, bytes]])`

Upload files into the devbox. Key arguments are `files`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `upload`, `append`, `FileUploadResponse`. It runs synchronously in the caller and returns directly.

## Gotchas

Most behavior is delegated through framework protocols, so preserve method signatures when editing even if an argument appears unused locally.
