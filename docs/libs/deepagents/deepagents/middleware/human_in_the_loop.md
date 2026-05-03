# `deepagents/middleware/human_in_the_loop.md`

## High-Level Purpose

`HumanInTheLoopMiddleware` (HITL) adds approval gates before specified tool calls. When the agent tries to call a tool in the `interrupt_on` list, the graph pauses execution and emits an `__interrupt__` event. The CLI's `ApprovalMenu` catches this event, asks the user to approve/reject/edit, and resumes the graph with the decision.

---

## Key Class

### `HumanInTheLoopMiddleware(AgentMiddleware)`

**Constructor parameters:**

| Parameter | Type | Description |
|---|---|---|
| `interrupt_on` | `list[str]` | Tool names that trigger approval |

---

## How It Works

LangGraph supports "interrupts" — points where the graph pauses and returns control to the caller. `HumanInTheLoopMiddleware` adds a LangGraph `interrupt()` call before each tool execution if the tool name is in `interrupt_on`.

**Interrupt payload sent to CLI:**
```python
{
    "tool_name": "execute",
    "tool_args": {"command": "rm -rf /build"},
    "description": "Execute shell command",
    "run_id": "abc123",
}
```

**Resume with decision:**
```python
# Approve (possibly with edited args):
graph.astream(None, config={"configurable": {"decisions": [{"decision": "approve", "args": {...}}]}})

# Reject:
graph.astream(None, config={"configurable": {"decisions": [{"decision": "reject"}]}})
```

When rejected, the tool receives a rejection `ToolMessage` ("Tool call rejected by user") and the agent decides how to proceed (usually by skipping the step or asking for an alternative).

---

## Common `interrupt_on` Lists

**CLI default (full HITL):**
```python
interrupt_on=["execute", "write_file", "edit_file", "web_search", "fetch_url", "task", "start_async_task"]
```

**Shell-only (approve shell commands, not file writes):**
```python
interrupt_on=["execute"]
```

**Disabled (auto-approve):**
```python
interrupt_on=[]
```

---

## Architecture Notes

**Positioning in stack:** `HumanInTheLoopMiddleware` is the outermost middleware (last in the list, first to run). This ensures it sees tool calls before any other middleware can process them — interrupts happen at the graph level, not inside tool execution.

**Non-interactive mode:** In `-n` (non-interactive) mode, `interrupt_on` is set to `[]`. Instead, `ShellAllowListMiddleware` validates commands inline using the allow-list. There is no human to approve anything.

---

## See Also

- [README.md](README.md) — middleware stack
- [../graph.md](../graph.md) — `interrupt_on` parameter
- [../../../../cli/deepagents_cli/widgets/approval.md](../../../../cli/deepagents_cli/widgets/approval.md) — TUI approval modal
- [../../../../cli/deepagents_cli/remote_client.md](../../../../cli/deepagents_cli/remote_client.md) — handles `__interrupt__` SSE events
