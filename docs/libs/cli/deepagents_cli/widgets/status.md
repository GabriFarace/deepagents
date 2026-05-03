# `deepagents_cli/widgets/status.py`

## High-Level Purpose

`StatusBar` is the fixed top bar of the TUI. It provides at-a-glance session information: the active model, current token usage, execution mode, and a spinner that animates while the agent is running. It is reactive — updated by `CLIApp` in response to stream events and state changes.

---

## Key Class

### `StatusBar(Widget)`

**Displayed fields (left to right):**

| Section | Content | Example |
|---|---|---|
| Model | Current model name | `claude-sonnet-4-6` |
| Agent | Active agent name (if custom) | `my-agent` |
| Tokens | `{used}k / {max}k` | `42k / 200k` |
| Mode | Current HITL mode | `auto-approve` or `HITL` |
| Status | Spinner (running) or empty (idle) | `⠋` |

**Reactive properties:**
- `model_name: reactive[str]` — updated when user switches model with `/model`
- `token_count: reactive[int]` — updated after each agent response from `TokenStateMiddleware`
- `is_running: reactive[bool]` — `True` while agent is executing; drives the spinner animation

**Spinner:** A Textual `LoadingIndicator` widget in the right corner. Starts animating when `is_running` becomes `True`, stops when `False`.

---

## Architecture Notes

**Token data source:** `TokenStateMiddleware` in the agent writes current token usage into the LangGraph state after each model call. The TUI reads this from the thread state updates in the SSE stream and calls `status_bar.token_count = new_value`.

**Reactive updates:** All property assignments from the SSE stream handler are done via `app.call_from_thread()` to ensure Textual's event loop owns widget state.

---

## See Also

- [app.md](../app.md) — updates `StatusBar` properties
- [../../agent.md](../agent.md) — `TokenStateMiddleware` feeds token counts
