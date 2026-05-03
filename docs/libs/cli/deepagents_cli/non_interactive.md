# `libs/cli/deepagents_cli/non_interactive.py`

## High-Level Purpose

`non_interactive.py` provides headless execution for the agent. When the user passes `-n "prompt"` or pipes input to the CLI, `run_non_interactive()` runs the agent without a TUI, streaming output directly to stdout. This is used for scripting, CI pipelines, and automation.

---

## Key Function

### `run_non_interactive(args) → int`

Runs the agent headlessly. Returns exit code 0 on success, 1 on error.

**Steps:**

1. **Bootstrap** — loads config and resolves the model spec
2. **Agent creation** — calls `create_cli_agent()` directly (no server subprocess). In non-interactive mode, the agent graph runs in-process rather than via a LangGraph server.
3. **Checkpointer** — uses an `InMemorySaver` for the session (no persistence by default). If `-r` is passed, uses `AsyncSqliteSaver` to resume a previous thread.
4. **Tool approval** — no HITL is possible (no user). Tool calls are filtered by `ShellAllowListMiddleware` — only commands in `--shell-allow-list` are allowed. Disallowed commands receive a rejection `ToolMessage` instead of executing.
5. **Streaming** — calls `graph.astream_events()` and prints tokens to stdout as they arrive (unless `--no-stream`)
6. **Quiet mode** — if `--quiet`, agent output goes to stdout only; tool call progress and status go to stderr

---

## Key Parameters (from `args`)

| Flag | Effect |
|---|---|
| `-n "prompt"` | The prompt to send |
| `--max-turns N` | Abort after N agentic steps (prevents infinite loops) |
| `--no-stream` | Buffer full response, print at end |
| `--quiet` | Only final response to stdout; rest to stderr |
| `--shell-allow-list "cmd1,cmd2"` | Comma-separated allowed shell commands |
| `-r [thread_id]` | Resume a previous thread |

---

## Output Format

By default (no `--quiet`):
```
[tool call: execute("ls -la")]
[tool result: total 48\n...]
Final response text here...
```

With `--quiet`:
- stderr: tool call progress
- stdout: only the final response text

---

## Architecture Notes

**In-process vs subprocess:** Non-interactive mode runs the agent graph directly in the CLI process (no `langgraph dev`). This is faster (no subprocess startup) but means the graph doesn't persist automatically — each run starts fresh unless `-r` is passed.

**Shell allow-list as safety net:** Without HITL, the agent could run arbitrary shell commands. The allow-list is the only guardrail. In CI environments, consider passing an explicit `--shell-allow-list` or `--auto-approve=False` to prevent unintended actions.

**Max turns:** The `--max-turns` limit prevents infinite agentic loops in unattended execution. The default is a conservative cap; increase it for complex tasks.

---

## See Also

- [main.md](main.md) — dispatches to `run_non_interactive()`
- [agent.md](agent.md) — `create_cli_agent()` called here
- [../../deepagents/deepagents/middleware/README.md](../../deepagents/deepagents/middleware/README.md) — `ShellAllowListMiddleware`
