# `partners/modal/langchain_modal/sandbox.py`

> Modal sandbox implementation.

## Position in the system

This partner package implements or supports a BackendProtocol-compatible sandbox. The core SDK can use these classes through its backend abstraction without depending directly on the provider.

## Imports and module-level state

This file imports `__future__, contextlib, modal, deepagents.backends.protocol, deepagents.backends.sandbox`.

## Functions and classes

### `ModalSandbox`

Modal sandbox implementation conforming to SandboxBackendProtocol. This class inherits from `BaseSandbox` and is the main object for this part of the module.

#### `ModalSandbox.__init__(self, *, sandbox: modal.Sandbox)`

Create a backend wrapping an existing Modal sandbox. Key arguments are `sandbox`. It mutates `self._sandbox`, `self._default_timeout`. It runs synchronously in the caller and returns directly.

#### `ModalSandbox.id(self)`

Return the sandbox id. It does not keep durable module state; effects come from returned values or delegated calls. It runs synchronously in the caller and returns directly.

#### `ModalSandbox.execute(self, command: str, *, timeout: int | None=None)`

Execute a shell command inside the sandbox. Args: command: Shell command string to execute. timeout: Maximum time in seconds to wait for this command. If None, uses the backend's default timeout. Note that in Modal's implementation, a timeout of 0 means "wait indefinitely". Returns: ExecuteResponse containing output, exit code, and truncation flag. Key arguments are `command`, `timeout`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `exec`, `wait`, `read`, `ExecuteResponse`. It runs synchronously in the caller and returns directly.

#### `ModalSandbox.download_files(self, paths: list[str])`

Download files from the sandbox. Key arguments are `paths`. It does not keep durable module state; effects come from returned values or delegated calls. It runs synchronously in the caller and returns directly.

#### `ModalSandbox.upload_files(self, files: list[tuple[str, bytes]])`

Upload files into the sandbox. Key arguments are `files`. It does not keep durable module state; effects come from returned values or delegated calls. It runs synchronously in the caller and returns directly.

## Gotchas

Most behavior is delegated through framework protocols, so preserve method signatures when editing even if an argument appears unused locally.
