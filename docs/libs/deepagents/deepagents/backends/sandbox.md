# `libs/deepagents/deepagents/backends/sandbox.py`

> Abstract base class that derives file operations from a sandbox's command
> execution plus upload/download primitives.

## Position in the system

`BaseSandbox` is for remote or isolated execution providers. Subclasses provide
`execute()`, `upload_files()`, `download_files()`, and `id`; the base class
builds `ls`, `read`, `write`, `edit`, `grep`, and `glob` by running Python or
shell snippets inside that sandbox.

```
partner backend
  ├─ execute(command)
  ├─ upload_files(...)
  └─ BaseSandbox helpers
       ├─ read via Python script
       ├─ edit via Python script or temp upload
       └─ grep/glob/ls via shell commands
```

## Imports and module-level state

The module imports `base64`, `json`, `os`, and `shlex` to safely move paths and
payloads through shell commands. The module-level command templates are the
important state: `_READ_COMMAND_TEMPLATE`, `_EDIT_COMMAND_TEMPLATE`,
`_EDIT_TMPFILE_TEMPLATE`, `_WRITE_CHECK_TEMPLATE`, and
`_GLOB_COMMAND_TEMPLATE`. They are not user prompts, but they are remote code
payloads executed inside sandboxes.

Key constants are `MAX_BINARY_BYTES`, `MAX_OUTPUT_BYTES`, `TRUNCATION_MSG`, and
`_EDIT_INLINE_MAX_BYTES`. LangSmith and partner backends can import these to
stay behaviorally aligned with the base implementation.

## Functions and classes

### `BaseSandbox`

`BaseSandbox` implements `SandboxBackendProtocol` for providers whose native
primitive is "run this command in the sandbox." It does not add security beyond
the subclass's sandbox boundary; every helper assumes `execute()` is already an
authorized capability.

#### `execute(command, *, timeout=None)`

This abstract method must run a command inside the sandbox and return
`ExecuteResponse`. All command-derived helpers depend on it, so provider
implementations should make stdout/stderr and exit codes predictable.

#### `ls(path)`

`ls()` base64-encodes the requested path, runs a short Python `os.scandir()`
script in the sandbox, and parses one JSON object per entry. Missing or
permission-denied directories quietly produce an empty list because the script
suppresses those errors.

#### `read(file_path, offset=0, limit=2000)`

`read()` runs `_READ_COMMAND_TEMPLATE` server-side. The script checks for file
existence, handles empty files with the standard reminder, classifies obvious
non-text extensions as base64 binary, detects invalid UTF-8 prefixes, and
paginates text by line before returning JSON.

The returned text page is capped at `MAX_OUTPUT_BYTES`; if the cap is hit,
`TRUNCATION_MSG` is appended with guidance to continue reading via offset or
smaller limit. Binary previews are capped at `MAX_BINARY_BYTES`.

#### `_write_preflight(file_path)`

This helper runs `_WRITE_CHECK_TEMPLATE` in the sandbox. It enforces the
create-only behavior used by `write()` and creates parent directories before
content transfer. Subclasses that override `write()` should call it to preserve
semantics, though there remains a time-of-check/time-of-use window between the
preflight command and the actual upload.

#### `write(file_path, content)`

`write()` calls `_write_preflight()` and then sends UTF-8 bytes through
`upload_files()`. The source content is not interpolated into a shell command,
which avoids most command-size and shell-escaping issues for writes.

#### `edit(file_path, old_string, new_string, replace_all=False)`

`edit()` chooses between two server-side replacement strategies. Small old/new
payloads use `_edit_inline()` with a base64-encoded JSON heredoc. Larger
payloads use `_edit_via_upload()`, which uploads the old and new strings as
temporary files and runs a script that reads them inside the sandbox.

Both scripts preserve line-ending style by trying the supplied string, a CRLF
variant, and an LF-normalized variant. If `replace_all=False` and more than one
match is found, the edit fails instead of guessing.

#### `_edit_inline(file_path, old_string, new_string, replace_all)`

This builds the JSON payload for `_EDIT_COMMAND_TEMPLATE`, executes it, parses
the single-line JSON response, and maps server-side error codes into
`EditResult`. Unexpected output becomes a bounded "unexpected server response"
error to avoid leaking huge command output through tool results.

#### `_edit_via_upload(file_path, old_string, new_string, replace_all)`

This handles large edit payloads. It creates random temp paths under `/tmp`,
uploads old/new bytes, runs `_EDIT_TMPFILE_TEMPLATE`, and parses the JSON
result. If the script does not return JSON, it attempts best-effort cleanup
with `rm -f` and logs cleanup failures.

#### `_map_edit_error(error, file_path, old_string)`

This static helper converts script error codes like `file_not_found`,
`not_a_text_file`, `string_not_found`, and `multiple_occurrences` into the same
user-facing messages other backends return.

#### `grep(pattern, path=None, glob=None)`

`grep()` uses shell `grep -rHnF` for literal search and parses
`path:line:text` output into structured matches. A `glob` argument is forwarded
as `--include=...`. The command ends with `|| true`, so no matches do not
surface as a shell failure.

#### `glob(pattern, path="/")`

`glob()` base64-encodes pattern and path, executes a Python `glob.glob()`
script in the target directory, and parses JSON lines into `FileInfo` objects.
The current result includes path and directory status; size/mtime are produced
inside the script but not copied into the returned `FileInfo`.

#### `id`

Subclasses must expose a stable identifier for the sandbox environment. The
protocol uses it for labeling and for callers that need to correlate file or
execution operations to a backing sandbox instance.

#### `upload_files(files)` and `download_files(paths)`

Subclasses must implement batch upload/download with partial success. They
should catch per-file failures and return `FileUploadResponse` or
`FileDownloadResponse` objects instead of aborting the whole batch.

## Gotchas

The command templates assume `python3`, basic shell quoting, and POSIX-like
tools such as `grep` and `rm`. Providers with non-POSIX environments may need
to override affected methods rather than inherit every helper.
