# `libs/deepagents/deepagents/backends/langsmith.py`

> LangSmith sandbox adapter that implements the SDK sandbox backend contract.

## Position in the system

`LangSmithSandbox` wraps a LangSmith `Sandbox` instance and inherits most file
tool behavior from `BaseSandbox`. It overrides read/write/upload/download where
the LangSmith SDK can transfer bytes directly and avoid shell transport limits.

## Imports and module-level state

The module imports `BaseSandbox` plus its read-size constants so SDK-native
read behavior can match command-template behavior. LangSmith classes are
imported only under `TYPE_CHECKING` or inside methods, keeping import-time
dependency requirements light. `logger` records SDK read/upload failures.

## Functions and classes

### `_binary_read_result(file_path, raw)`

This helper returns the same binary read shape used by `BaseSandbox.read()`.
Small binary files become base64 `FileData`; files larger than
`MAX_BINARY_BYTES` return the preview-size error with the same `File '<path>':`
prefix used by the base implementation.

### `LangSmithSandbox(sandbox)`

The constructor stores the provided LangSmith sandbox instance and sets a
30-minute default command timeout. It does not create or manage the sandbox
lifecycle; callers hand in an already-created `Sandbox`.

#### `id`

The property returns `self._sandbox.name`, which is the LangSmith sandbox's
stable label.

#### `execute(command, *, timeout=None)`

`execute()` forwards the command to `sandbox.run()` with either the provided
timeout or the default. It combines stdout and stderr into one output string
and returns `ExecuteResponse` with the sandbox exit code. The method marks
`truncated=False` because it does not perform output truncation itself.

#### `write(file_path, content)`

This override preserves `BaseSandbox.write()`'s create-only preflight, then
uses the LangSmith SDK's `write()` method to send UTF-8 bytes in the request
body. That avoids shell argument-size limits for large content. SDK client
errors become `WriteResult(error=...)`.

#### `read(file_path, offset=0, limit=2000)`

This override reads bytes directly through the SDK, then locally reproduces
`BaseSandbox.read()` semantics. Missing resources become `file_not_found`,
empty files return the standard reminder, non-text extensions and invalid
UTF-8 become capped base64 binary responses, and text files are universal-
newline normalized before line pagination.

The returned text page is capped using `MAX_OUTPUT_BYTES` with
`TRUNCATION_MSG`, matching the command-template implementation while avoiding
large stdout transfer through `execute()`.

#### `download_files(paths)`

`download_files()` reads each absolute path through the SDK and supports
partial success. Non-absolute paths return `"invalid_path"`, missing resources
return `"file_not_found"`, and SDK errors whose message indicates a directory
return `"is_directory"`.

#### `upload_files(files)`

`upload_files()` writes each absolute path through the SDK and preserves input
order in responses. Non-absolute paths return `"invalid_path"`; SDK client
errors are logged at debug level and reported as `"permission_denied"`.

## Gotchas

`LangSmithSandbox` inherits `ls`, `grep`, `glob`, and `edit` from
`BaseSandbox`, so those operations still run commands inside the sandbox. Only
read/write/upload/download use native SDK transfer paths.
