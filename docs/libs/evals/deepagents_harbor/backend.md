# `evals/deepagents_harbor/backend.py`

> BackendProtocol adapter over Harbor environments, giving deepagents shell and filesystem access through Harbor actions.

## Position in the system

This module belongs to the Harbor integration layer. It adapts Harbor environments and LangSmith metadata into interfaces that the deepagents SDK and eval tooling can consume.

## Imports and module-level state

This file imports `asyncio, logging, shlex, tempfile, pathlib, deepagents.backends.filesystem, deepagents.backends.protocol, deepagents.backends.utils, harbor.environments.base`.
Module constants worth noticing: `_SYNC_NOT_SUPPORTED`, `DEFAULT_COMMAND_TIMEOUT_SEC`, `_PIPE_FIELD_COUNT`, `_GREP_FIELD_COUNT`, `_COMMAND_PREVIEW_CHAR_LIMIT`.

## Functions and classes

### `HarborSandbox`

A sandbox implementation using Harbor environments. Write and edit use Harbor's native file transfer (upload/download) for data content. Read, ls, grep, and glob execute shell commands in the environment. This class inherits from `SandboxBackendProtocol` and is the main object for this part of the module.

#### `HarborSandbox.__init__(self, environment: BaseEnvironment)`

Initialize HarborSandbox with the given environment. Key arguments are `environment`. It mutates `self.environment`. It runs synchronously in the caller and returns directly.

#### `HarborSandbox.aexecute(self, command: str, *, timeout: int | None=None)`

Execute a bash command in the task environment. Args: command: Shell command string to execute. timeout: Maximum time in seconds to wait for the command to complete. If None, uses the environment's default timeout. Key arguments are `command`, `timeout`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `strip`, `ExecuteResponse`, `join`, `append`, `replace`, `wait_for`. This is asynchronous and awaits I/O or framework operations before returning.

#### `HarborSandbox.execute(self, command: str, *, timeout: int | None=None)`

Execute a bash command in the task environment. Key arguments are `command`, `timeout`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `NotImplementedError`. It runs synchronously in the caller and returns directly.

#### `HarborSandbox.id(self)`

Unique identifier for the sandbox backend. It does not keep durable module state; effects come from returned values or delegated calls. It runs synchronously in the caller and returns directly.

#### `HarborSandbox.aread(self, file_path: str, offset: int=0, limit: int=2000)`

Read raw file content for the requested line range. Line-number formatting is applied by the middleware, so this method returns unformatted text matching the contract of `ReadResult`. Key arguments are `file_path`, `offset`, `limit`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `quote`, `rstrip`, `check_empty_content`, `ReadResult`, `aexecute`, `FileData`. This is asynchronous and awaits I/O or framework operations before returning.

#### `HarborSandbox.read(self, file_path: str, offset: int=0, limit: int=2000)`

Read file content with line numbers using shell commands. Key arguments are `file_path`, `offset`, `limit`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `NotImplementedError`. It runs synchronously in the caller and returns directly.

#### `HarborSandbox.awrite(self, file_path: str, content: str)`

Create a new file using Harbor's native file transfer. Uses `environment.upload_file()` instead of embedding content in the command string, which avoids OS ARG_MAX limits on large payloads. Key arguments are `file_path`, `content`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `quote`, `WriteResult`, `aexecute`, `strip`, `NamedTemporaryFile`, `write`. This is asynchronous and awaits I/O or framework operations before returning.

#### `HarborSandbox.write(self, file_path: str, content: str)`

Create a new file (sync). Not supported; use `awrite`. Key arguments are `file_path`, `content`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `NotImplementedError`. It runs synchronously in the caller and returns directly.

#### `HarborSandbox.aedit(self, file_path: str, old_string: str, new_string: str, replace_all: bool=False)`

Edit a file by replacing string occurrences. Downloads the file via Harbor's native file transfer, performs the replacement locally, and re-uploads. This keeps arbitrarily large payloads off the command line, avoiding OS `ARG_MAX` limits. Key arguments are `file_path`, `old_string`, `new_string`, `replace_all`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `EditResult`, `TemporaryDirectory`, `count`, `write_bytes`, `Path`, `decode`. This is asynchronous and awaits I/O or framework operations before returning.

#### `HarborSandbox.edit(self, file_path: str, old_string: str, new_string: str, replace_all: bool=False)`

Edit a file by replacing string occurrences (sync). Not supported; use `aedit`. Key arguments are `file_path`, `old_string`, `new_string`, `replace_all`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `NotImplementedError`. It runs synchronously in the caller and returns directly.

#### `HarborSandbox.als(self, path: str)`

List directory contents with metadata using shell commands. Key arguments are `path`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `quote`, `split`, `LsResult`, `aexecute`, `strip`, `len`. This is asynchronous and awaits I/O or framework operations before returning.

#### `HarborSandbox.ls(self, path: str)`

List directory contents with metadata using shell commands. Key arguments are `path`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `NotImplementedError`. It runs synchronously in the caller and returns directly.

#### `HarborSandbox.agrep(self, pattern: str, path: str | None=None, glob: str | None=None)`

Search for a literal string in files using `grep -F`. Key arguments are `pattern`, `path`, `glob`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `quote`, `rstrip`, `split`, `GrepResult`, `aexecute`, `strip`. This is asynchronous and awaits I/O or framework operations before returning.

#### `HarborSandbox.grep(self, pattern: str, path: str | None=None, glob: str | None=None)`

Search for a literal string in files using `grep -F` (sync). Not supported; use `agrep`. Key arguments are `pattern`, `path`, `glob`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `NotImplementedError`. It runs synchronously in the caller and returns directly.

#### `HarborSandbox.aglob(self, pattern: str, path: str='/')`

Find files matching glob pattern using shell commands. Please note that this implementation does not currently support all glob patterns. Key arguments are `pattern`, `path`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `quote`, `strip`, `split`, `GlobResult`, `aexecute`, `len`. This is asynchronous and awaits I/O or framework operations before returning.

#### `HarborSandbox.glob(self, pattern: str, path: str='/')`

Find files matching glob pattern using shell commands. Key arguments are `pattern`, `path`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `NotImplementedError`. It runs synchronously in the caller and returns directly.

#### `HarborSandbox.aupload_files(self, files: list[tuple[str, bytes]])`

Upload files using Harbor's native file transfer. Key arguments are `files`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `append`, `NamedTemporaryFile`, `write`, `Path`, `warning`, `upload_file`. This is asynchronous and awaits I/O or framework operations before returning.

#### `HarborSandbox.upload_files(self, files: list[tuple[str, bytes]])`

Upload files (sync). Not supported; use `aupload_files`. Key arguments are `files`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `NotImplementedError`. It runs synchronously in the caller and returns directly.

#### `HarborSandbox.adownload_files(self, paths: list[str])`

Download files using Harbor's native file transfer. Key arguments are `paths`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `TemporaryDirectory`, `Path`, `read_bytes`, `append`, `download_file`, `FileDownloadResponse`. This is asynchronous and awaits I/O or framework operations before returning.

#### `HarborSandbox.download_files(self, paths: list[str])`

Download files (sync). Not supported; use `adownload_files`. Key arguments are `paths`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `NotImplementedError`. It runs synchronously in the caller and returns directly.

## Gotchas

Several blocks are async and assume they run inside the owning framework event loop. Calling them from sync code needs the package CLI or an explicit async runner.
