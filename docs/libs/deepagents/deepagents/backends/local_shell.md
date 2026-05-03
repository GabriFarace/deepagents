# `deepagents/backends/local_shell.py`

## High-Level Purpose

`LocalShellBackend` gives the agent direct access to the local filesystem and a local shell. It is the backend used by the CLI in interactive mode. Unlike sandbox backends, it runs commands as the current user with no isolation — commands execute directly on the user's machine.

---

## Key Class

### `LocalShellBackend`

Implements `SandboxBackendProtocol` (filesystem + `execute`).

**Constructor parameters:**

| Parameter | Default | Description |
|---|---|---|
| `root_dir` | `os.getcwd()` | Working directory for relative paths; shell commands run here |
| `allowed_paths` | `None` | Optional list of absolute paths the backend may access |

**File operations:** Delegates to `FilesystemBackend` under the hood (reads/writes actual files on disk).

**Shell execution:** `execute(command, env)` runs via `asyncio.create_subprocess_shell`. Returns `ExecuteResult` with `stdout`, `stderr`, and `exit_code`.

---

## Safety Considerations

`LocalShellBackend` is explicitly **not sandboxed**. The HITL interrupt system (`HumanInTheLoopMiddleware` + `ApprovalMenu`) is the primary guardrail in interactive CLI use. In non-interactive mode, `ShellAllowListMiddleware` provides the only protection.

For untrusted code, use a sandbox partner backend instead.

---

## See Also

- [README.md](README.md) — backend comparison
- [sandbox.md](sandbox.md) — sandboxed alternatives
- [../../cli/deepagents_cli/agent.md](../../cli/deepagents_cli/agent.md) — CLI always uses `LocalShellBackend`
