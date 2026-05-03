# `libs/cli/deepagents_cli/app.py`

## High-Level Purpose

`app.py` is the heart of the interactive TUI. It contains `CLIApp`, a Textual `App` subclass that owns the entire widget tree, the message queue, command routing, and the connection to the LangGraph server. Everything the user sees — the status bar, the scrollable transcript, the input field, every modal — lives inside or is orchestrated by `CLIApp`.

---

## Widget Hierarchy

```
CLIApp
├── StatusBar          (top bar: model name, token count, mode indicator)
├── WelcomeBanner      (initial screen, hidden after first message)
├── VerticalScroll     (scrollable message transcript)
│   └── [dynamic message widgets appended as agent responds]
│       ├── UserMessage
│       ├── AssistantMessage
│       ├── ToolCallMessage
│       ├── DiffMessage
│       └── ErrorMessage
├── ChatInput          (text input at bottom)
└── Modal overlays (pushed/popped on demand):
    ├── ApprovalMenu        (/approve — tool call HITL)
    ├── AskUserMenu         (agent-initiated clarification)
    ├── AgentSelector       (/agents)
    ├── ThreadSelector      (/threads)
    ├── ModelSelector       (/model)
    ├── ThemeSelector       (/theme)
    ├── NotificationCenter  (/notifications)
    ├── MCPViewer           (/mcp)
    ├── HelpScreen          (/help)
    └── UpdateAvailable     (shown on startup if new version exists)
```

---

## Key Classes and Functions

### `CLIApp(App)` — main application class

**State fields:**

| Field | Type | Purpose |
|---|---|---|
| `_agent` | `RemoteAgent \| None` | Client for the LangGraph server; `None` until server is ready |
| `_message_queue` | `deque[QueuedMessage]` | Pending user messages and commands |
| `_busy_state` | `BusyState` | Tracks whether the agent is currently running |
| `_current_thread_id` | `str \| None` | Active LangGraph thread ID |
| `_server_kwargs` | `dict` | Config passed to `start_server_and_get_agent()` |
| `_stream_handler` | `StreamHandler \| None` | Handles SSE chunks from the server |

**Lifecycle methods:**

- `on_mount()` — Called by Textual after widgets are created. Kicks off `_start_server()` as an async task.
- `_start_server()` — Awaits `start_server_and_get_agent()`, then populates `self._agent` and creates a new thread ID. If a resume thread was requested (`-r`), resolves it here.

**Input routing:**

- `on_chat_input_submitted(event)` — Receives user input from `ChatInput`. Classifies the input:
  - Starts with `/` → `_classify_command()` → `_handle_command()`
  - Starts with `/skill:` → `_handle_skill_command()`
  - Normal text → `_enqueue_message()`

### `_classify_command(text) → BypassTier`

Returns a `BypassTier` enum that determines whether a slash command can bypass the queue (execute immediately even while the agent is running) or must wait:

| Tier | Examples | Behavior |
|---|---|---|
| `ALWAYS` | `/quit` | Executes unconditionally |
| `CONNECTING` | `/version` | Only while server is starting |
| `IMMEDIATE_UI` | `/agents`, `/model` | Opens a modal immediately |
| `SIDE_EFFECT_FREE` | `/mcp`, `/trace`, `/changelog` | Side effect fires now; UI update queued |
| `QUEUED` | Most others | Wait until agent is idle |

### `_handle_command(text, tier) → None`

Routes a slash command to its handler:

- `/agents` → push `AgentSelector` modal
- `/model` → push `ModelSelector` modal
- `/threads` → push `ThreadSelector` modal
- `/clear` → queue a "clear" action (starts fresh thread)
- `/theme` → push `ThemeSelector` modal
- `/mcp` → push `MCPViewer` modal
- `/help` → push `HelpScreen`
- `/quit` → `self.exit()`
- `/trace` → print LangSmith trace URL
- `/tokens` → print current token usage
- `/skill:<name> [args]` → `_handle_skill_command()`

### `_process_queue() → None`

Called repeatedly via a Textual `set_interval` timer when the agent is idle. Pops the next `QueuedMessage` and dispatches it by calling `_run_agent()`.

### `_run_agent(message) → None`

The main execution path for a user message:

1. Sets `_busy_state` to `RUNNING`, disables `ChatInput`
2. Creates or reuses a `UserMessage` widget in the transcript
3. Calls `self._agent.astream(message, thread_id=self._current_thread_id)`
4. Iterates the SSE stream, calling `self._stream_handler.handle_chunk(chunk)` for each update
5. On completion (or error), resets `_busy_state` and re-enables `ChatInput`

### `_handle_interrupt(interrupt_data) → dict`

Called when the server streams an `__interrupt__` signal (HITL gate). Pushes an `ApprovalMenu` modal, awaits the user's decision (approve / reject / edit), and returns the decision dict to `_run_agent()` for re-submission to the server.

### `_handle_ask_user(question) → str`

Called when the agent triggers the `ask_user` tool. Pushes an `AskUserMenu` modal and returns the user's text response.

---

## Message Queue Flow

```
User types → ChatInput.Submitted event
    │
    ├─ command? ──→ classify tier
    │                  ├─ ALWAYS / IMMEDIATE_UI → execute now
    │                  └─ QUEUED → enqueue
    │
    └─ normal text → enqueue
                         │
                  _message_queue (deque)
                         │
                  _process_queue() [polling timer, only runs when idle]
                         │
                  _run_agent(message)
                         │
                  RemoteAgent.astream()  → SSE stream → StreamHandler
```

---

## Architecture Notes

**Textual's reactive model:** `CLIApp` uses Textual's `reactive` properties for a few shared states (e.g., the "busy" indicator in the status bar). Widget mutations from the SSE stream handler are always dispatched via `call_from_thread()` or `post_message()` to ensure thread safety, since SSE processing runs in an async task.

**Modal management:** Modals are pushed onto the screen stack with `self.push_screen(modal)`. They return their result via `await`. This lets `_handle_interrupt` and `_handle_ask_user` `await` the modal result cleanly.

**Scroll-to-bottom:** After appending a message widget, the app calls `scroll_to_widget(widget, animate=False)` to keep the transcript pinned to the latest output.

---

## See Also

- [main.md](main.md) — how `run_textual_app()` is called
- [remote_client.md](remote_client.md) — `RemoteAgent` and `StreamHandler`
- [server_manager.md](server_manager.md) — `start_server_and_get_agent()`
- [widgets/README.md](widgets/README.md) — all widget classes
- [command_registry.md](command_registry.md) — slash command definitions
