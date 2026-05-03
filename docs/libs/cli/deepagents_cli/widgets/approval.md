# `deepagents_cli/widgets/approval.py`

## High-Level Purpose

`ApprovalMenu` is the Human-in-the-Loop (HITL) modal. It appears whenever the agent wants to call a tool that requires user approval (e.g., `execute`, `write_file`). The user can approve, reject, or edit the tool's arguments before the agent proceeds. This is the primary safety mechanism for the interactive CLI.

---

## Key Class

### `ApprovalMenu(ModalScreen)`

A Textual `ModalScreen` that occupies the full screen. Pushed onto the screen stack by `CLIApp._handle_interrupt()` when the server emits an `__interrupt__` event.

**Displayed information:**
- Tool name and a human-readable description (e.g., "Execute shell command")
- Tool arguments, pretty-printed (JSON, with syntax highlighting)
- For `execute` tool: the shell command highlighted with shell syntax coloring
- For `write_file`/`edit_file`: a diff preview of proposed changes
- Keyboard shortcut hints (A = Approve, R = Reject, E = Edit)

**Actions:**

| Key | Action | Effect |
|---|---|---|
| `a` / Enter | Approve | Returns `{"decision": "approve"}` to `CLIApp` |
| `r` | Reject | Returns `{"decision": "reject"}` to `CLIApp` |
| `e` | Edit | Opens an edit sub-panel with the JSON args pre-filled |

**Edit flow:** When the user presses `e`, `ApprovalMenu` shows an embedded text editor (a Textual `TextArea`) pre-filled with the tool args as JSON. The user can modify the arguments. Pressing Ctrl+S or Enter (in single-line edit) submits the edited args. Returns `{"decision": "approve", "args": <edited_args>}`.

**Result type:**
```python
ApprovalResult = {"decision": "approve" | "reject", "args": dict | None}
```

`CLIApp._handle_interrupt()` awaits the modal result, then calls `remote_agent.resume(thread_id, decision=result)`.

---

## HITL Flow Diagram

```
Agent graph runs
    │
    └─ hits interrupt node (e.g., HumanInTheLoopMiddleware)
            │
            ▼
   Server emits __interrupt__ event (via SSE)
            │
            ▼
   StreamHandler → InterruptAction
            │
            ▼
   CLIApp._handle_interrupt(interrupt_data)
            │
            ▼
   push_screen(ApprovalMenu(interrupt_data))  ← user sees modal
            │
            ▼ (user presses A / R / E)
   approval_result = await modal
            │
            ▼
   remote_agent.resume(thread_id, decision=approval_result)
            │
            ▼
   Server resumes graph run with decision
            │
            ▼
   Tool executes (or is skipped if rejected)
            │
            ▼
   Stream continues → normal response widgets
```

---

## Architecture Notes

**Interrupt payload:** The LangGraph server serializes interrupt data as a dict with keys like `tool_name`, `tool_args`, `description`, `run_id`. `ApprovalMenu` reads all of these to render the display.

**Non-interactive mode:** In `-n` mode, there is no `ApprovalMenu`. Instead, `ShellAllowListMiddleware` checks commands against the allow-list inline and returns a rejection `ToolMessage` for disallowed commands without pausing execution.

**Multiple interrupts:** A single agent turn can trigger multiple interrupts (e.g., three sequential `execute` calls). Each generates a separate `ApprovalMenu` push, and the TUI processes them one at a time.

---

## See Also

- [app.md](../app.md) — `_handle_interrupt()` calls this modal
- [remote_client.md](../remote_client.md) — emits `InterruptAction` from `__interrupt__` SSE event
- [../../deepagents/deepagents/middleware/human_in_the_loop.md](../../deepagents/deepagents/middleware/human_in_the_loop.md) — the middleware that generates interrupts
