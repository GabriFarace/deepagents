# `libs/cli/deepagents_cli/hooks.py`

## High-Level Purpose

`hooks.py` provides a lightweight event dispatch system that lets external scripts react to CLI lifecycle events. When the CLI starts a session, receives a user message, or finishes an agent response, it fires hooks — external processes that receive a JSON payload on stdin. Hooks are fire-and-forget: they don't block the CLI and their output is not displayed.

---

## Hook Config Format

Hooks are defined in `~/.deepagents/hooks.json`:

```json
{
  "hooks": [
    {
      "command": ["bash", "/path/to/hook.sh"],
      "events": ["session.start", "message.agent"]
    },
    {
      "command": ["python3", "log_to_db.py"],
      "events": ["message.user", "message.agent"]
    }
  ]
}
```

Each hook entry:
- `command` — the subprocess to run (list form, like `subprocess.run`)
- `events` — list of event names that trigger this hook

---

## Event Types

| Event | When fired | Payload |
|---|---|---|
| `session.start` | When TUI mounts and server is ready | `{event, thread_id, agent_name, cwd, git_branch}` |
| `session.end` | When TUI unmounts (user quits) | `{event, thread_id, duration_s}` |
| `message.user` | When user submits a message | `{event, thread_id, message}` |
| `message.agent` | When agent finishes a response | `{event, thread_id, response_text, tool_calls}` |

---

## Key Functions

### `dispatch_hook(event, payload) → None` (async)

Fires all hooks registered for `event`. For each matching hook:
1. Spawns a subprocess with `asyncio.create_subprocess_exec()`
2. Writes `json.dumps(payload)` to its stdin
3. Waits up to 5 seconds for the subprocess to exit
4. Logs a warning if it times out (does not kill the subprocess)

### `dispatch_hook_fire_and_forget(event, payload) → None`

Synchronous wrapper that creates an asyncio task for `dispatch_hook()`. Used from synchronous TUI lifecycle callbacks that can't `await`.

### `load_hooks_config() → HooksConfig`

Reads `~/.deepagents/hooks.json`. Returns an empty config if the file doesn't exist.

---

## Use Cases

**Logging to external systems:** A hook on `message.agent` can POST the conversation to a database, Slack webhook, or analytics service.

**Notifications:** A hook on `session.end` can send a desktop notification when a long-running task completes.

**Audit trail:** A hook on `message.user` and `message.agent` can write an append-only log of all conversation activity.

---

## Architecture Notes

**Fire-and-forget by design:** Hooks are not critical path. If a hook fails or times out, the CLI continues normally. A 5-second timeout prevents slow hooks from blocking session shutdown.

**Security consideration:** Hook commands run with the user's full shell permissions. Hooks defined in `~/.deepagents/hooks.json` are trusted by definition (the user put them there). Project-level hooks, if supported in the future, would need the same trust mechanism as project MCP configs.

---

## See Also

- [config.md](config.md) — config directory paths
- [app.md](app.md) — where `dispatch_hook_fire_and_forget()` is called
