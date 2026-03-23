# `deepagents_harbor/backend.py`

## High-Level Purpose

Provides `HarborSandbox`, a `SandboxBackendProtocol` implementation backed by Harbor environments. All file and command operations are executed via shell commands inside the Harbor environment. This class is used by `DeepAgentsWrapper` to give the agent access to the Harbor task sandbox.

**Note:** This backend only supports async execution. All sync methods raise `NotImplementedError`.

## Module-Level Constants

| Constant | Value | Description |
|----------|-------|-------------|
| `DEFAULT_COMMAND_TIMEOUT_SEC` | `300` | Default per-command timeout (5 minutes) |
| `_PIPE_FIELD_COUNT` | `2` | Expected fields in `ls`/`glob` pipe-separated output |
| `_GREP_FIELD_COUNT` | `3` | Minimum fields in `grep` colon-separated output |
| `_COMMAND_PREVIEW_CHAR_LIMIT` | `200` | Max chars shown in timeout error previews |
| `_EXIT_NOT_FOUND` | `1` | Shell exit code: old_string not found during edit |
| `_EXIT_MULTIPLE_MATCHES` | `2` | Shell exit code: multiple matches during edit |
| `_EXIT_FILE_MISSING` | `3` | Shell exit code: file not found during edit |
| `_EXIT_DECODE_FAILED` | `4` | Shell exit code: base64 decode failure during edit |

## Classes

### `HarborSandbox`

**Purpose:** Implements the full `SandboxBackendProtocol` using shell commands executed in a Harbor environment. Each file operation composes a shell script and runs it via `aexecute`.

**Inherits from:** `deepagents.backends.protocol.SandboxBackendProtocol`

#### `__init__(environment: BaseEnvironment) -> None`
Stores the Harbor `BaseEnvironment` instance.

#### `id -> str` (property)
Returns `environment.session_id`.

#### `aexecute(command: str, *, timeout: int | None = None) -> ExecuteResponse`

**Purpose:** Execute a shell command in the Harbor environment.

**Key Logic:**
- Wraps `environment.exec(command)` in `asyncio.wait_for()` when `timeout > 0`.
- On `TimeoutError`: returns `ExecuteResponse(output="ERROR: Command timed out...", exit_code=124)`.
- Filters harmless "bash: no job control" and "cannot set terminal process group" messages from both stdout and stderr.
- Appends stderr to output with a `"\n\n stderr: "` separator if stderr is non-empty.

#### `aread(file_path: str, offset: int = 0, limit: int = 2000) -> ReadResult`

Uses `awk` to extract lines `[offset+1, offset+limit]` from the file. Returns `ReadResult(error=...)` if the file does not exist or the command fails.

#### `awrite(file_path: str, content: str) -> WriteResult`

Creates a new file using base64-encoded content piped via heredoc to avoid argument-length limits. Fails if the file already exists.

#### `aedit(file_path: str, old_string: str, new_string: str, replace_all: bool = False) -> EditResult`

Replaces string occurrences in a file using a combination of Python 3 (for JSON parsing), `grep` (for counting occurrences), and `perl` (for the actual substitution). The old/new strings are JSON-encoded and base64-encoded to safely transfer through the shell heredoc. Returns specific error objects for each exit code scenario.

#### `als(path: str) -> LsResult`

Lists directory contents using a shell loop that outputs `name|is_dir` pairs. Parses the `|`-separated output into `FileInfo` dicts.

#### `agrep(pattern: str, path: str | None = None, glob: str | None = None) -> GrepResult`

Runs `grep -rHn` to search for patterns. Parses `file:line:text` output into `GrepMatch` dicts. Returns `GrepResult(matches=[])` when nothing is found, and `GrepResult(error=...)` on grep failure (exit code ≥ 2).

#### `aglob(pattern: str, path: str = "/") -> GlobResult`

Finds files matching a shell glob pattern using a `for file in <pattern>` loop. Returns `FileInfo` dicts.

All sync variants (`execute`, `read`, `write`, `edit`, `ls`, `grep`, `glob`) raise `NotImplementedError`.

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| `SandboxBackendProtocol` and result types | `deepagents.backends.protocol` | Interface and response types |
| `check_empty_content`, `create_file_data` | `deepagents.backends.utils` | Helpers for read result formatting |
| `BaseEnvironment` | `harbor.environments.base` | Harbor environment interface |
| `asyncio`, `base64`, `json`, `shlex`, `logging` | stdlib | Command execution and encoding |
