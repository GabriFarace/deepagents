# `app.py`

## High-Level Purpose

This module defines the main Textual UI application (`DeepAgentsApp`) for `deepagents-cli`. It is a Textual `App` subclass that provides the full interactive terminal chat interface including:

- Chat message display and history
- Chat input with slash-command support
- Status bar with mode, model, token counts, and git branch
- Welcome banner
- Approval menus for Human-in-the-Loop (HITL) tool confirmations
- Model selector, thread selector, theme selector, and MCP viewer modals
- Token tracking and display
- Deferred server startup with background workers
- Session state management (thread IDs, auto-approve)
- iTerm2 cursor guide workaround for visual compatibility

## Classes

### `QueuedMessage`

**Type:** `dataclass(frozen=True, slots=True)`

Represents a user message awaiting processing.

| Attribute | Type | Description |
|---|---|---|
| `text` | `str` | The message text content |
| `mode` | `InputMode` | Input mode determining message routing (`'normal'`, `'shell'`, `'command'`) |

### `DeferredAction`

**Type:** `dataclass(frozen=True, slots=True, kw_only=True)`

An action (model switch, thread switch, chat output) deferred until the current busy state resolves.

| Attribute | Type | Description |
|---|---|---|
| `kind` | `DeferredActionKind` | Identity key for deduplication |
| `execute` | `Callable[[], Awaitable[None]]` | Async callable that performs the actual work |

`DeferredActionKind` is a `Literal` of `"model_switch"`, `"thread_switch"`, `"chat_output"`.

### `TextualTokenTracker`

Token counter that updates the status bar.

**Methods:**

- `__init__(update_callback, hide_callback=None)` — Initialize with callbacks to update the display.
- `add(total_tokens, _output_tokens=0)` — Update token count from a response.
- `reset()` — Reset token count to 0.
- `hide()` — Hide the token display (e.g., during streaming).
- `show()` — Show the token display with current value.

### `TextualSessionState`

Session state for the Textual app: manages thread ID and auto-approve flag.

**Methods:**

- `__init__(*, auto_approve=False, thread_id=None)` — Initialize session state; generates a UUID7 if no `thread_id` is provided.
- `reset_thread() -> str` — Generate a new UUID7 thread ID, store it, and return it.

### `DeepAgentsApp`

**Inherits from:** `textual.app.App`

The main Textual application. Manages the full TUI lifecycle.

**Class Variables:**
| Attribute | Value | Description |
|---|---|---|
| `TITLE` | `"Deep Agents"` | Textual window title |
| `CSS_PATH` | `"app.tcss"` | Path to the Textual CSS stylesheet |
| `ENABLE_COMMAND_PALETTE` | `False` | Disables Textual's built-in command palette |
| `SCROLL_SENSITIVITY_Y` | `1.0` | Vertical scroll speed |
| `BINDINGS` | `[...]` | App-level keybindings (see below) |

**Key Bindings:**
| Key | Action | Description |
|---|---|---|
| `escape` | `interrupt` | Interrupt current agent turn |
| `ctrl+c` | `quit_or_interrupt` | Quit or interrupt |
| `ctrl+d` | `quit_app` | Quit the app |
| `ctrl+t` / `shift+tab` | `toggle_auto_approve` | Toggle auto-approve mode |
| `ctrl+o` | `toggle_tool_output` | Toggle tool output display |
| `ctrl+x` | `open_editor` | Open external editor for prompt composition |
| `up/k`, `down/j`, `enter` | `approval_up/down/select` | Navigate approval menu |
| `y/1`, `a/2`, `n/3` | `approval_yes/auto/no` | Approve/auto/reject in approval menu |

**Inner Messages:**
- `ServerReady` — Posted when the background server-startup worker succeeds. Contains `agent`, `server_proc`, and `mcp_server_info`.
- `ServerStartFailed` — Posted when the background server-startup worker fails. Contains `error`.

**Constructor Parameters:**

| Parameter | Type | Description |
|---|---|---|
| `agent` | `Pregel \| None` | Pre-configured LangGraph agent |
| `assistant_id` | `str \| None` | Agent identifier for memory storage |
| `backend` | `CompositeBackend \| None` | Backend for file operations |
| `auto_approve` | `bool` | Start with auto-approve enabled |
| `cwd` | `str \| Path \| None` | Working directory to display |
| `thread_id` | `str \| None` | Thread ID for the session |
| `resume_thread` | `str \| None` | Resume intent from `-r` flag |
| `initial_prompt` | `str \| None` | Optional prompt to auto-submit at start |
| `mcp_server_info` | `list[MCPServerInfo] \| None` | MCP server metadata |
| `profile_override` | `dict[str, Any] \| None` | Extra profile fields from `--profile-override` |
| `server_proc` | `ServerProcess \| None` | LangGraph server process |
| `server_kwargs` | `dict[str, Any] \| None` | Kwargs for deferred server startup |
| `mcp_preload_kwargs` | `dict[str, Any] \| None` | Kwargs for `_preload_session_mcp_server_info` |
| `model_kwargs` | `dict[str, Any] \| None` | Kwargs for deferred `create_model()` |

## Module-Level Functions

### `_load_theme_preference() -> str`

Reads the saved theme name from `~/.deepagents/config.toml` under `[ui].theme`. Falls back to `theme.DEFAULT_THEME` if the file is missing, corrupt, or the stored name is unknown.

**Returns:** A Textual theme name string.

### `save_theme_preference(name: str) -> bool`

Persists the theme preference to `~/.deepagents/config.toml`. Uses an atomic write (temp file + rename) to avoid corruption.

**Parameters:**
- `name`: Textual theme name to save.

**Returns:** `True` if saved successfully, `False` on any error.

### `_extract_model_params_flag(raw_arg: str) -> tuple[str, dict[str, Any] | None]`

Extracts the `--model-params` flag and its JSON value from a `/model` command argument string. Handles quoted and bare JSON objects.

**Returns:** `(remaining_args, parsed_dict | None)`.

## Module-Level Constants

| Constant | Type | Description |
|---|---|---|
| `_IS_ITERM` | `bool` | Whether running inside iTerm2 |
| `_ITERM_CURSOR_GUIDE_OFF/ON` | `str` | iTerm2 OSC 1337 escape sequences |
| `_TYPING_IDLE_THRESHOLD_SECONDS` | `float` | 2.0s — idle threshold for showing deferred approval |
| `_DEFERRED_APPROVAL_TIMEOUT_SECONDS` | `float` | 30.0s — max wait for deferred approval |
| `_COMMAND_URLS` | `dict[str, str]` | Slash-command to URL mapping for browser-opening commands |

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `textual.app.App` | textual | Base Textual application |
| `textual.binding.Binding` | textual | Key bindings |
| `textual.screen.ModalScreen` | textual | Modal screens (model selector, etc.) |
| `DeepAgentsApp` from theme, config | local | Brand colors, settings |
| `ChatInput` | `widgets.chat_input` | Chat input widget |
| `LoadingWidget` | `widgets.loading` | Animated spinner |
| `MessageStore`, `MessageData` | `widgets.message_store` | Virtualized message store |
| `StatusBar` | `widgets.status` | Bottom status bar |
| `WelcomeBanner` | `widgets.welcome` | Startup banner |
| `SessionStats`, `SpinnerStatus` | `_session_stats` | Token tracking |
| `CLIContext` | `_cli_context` | Runtime context type |
