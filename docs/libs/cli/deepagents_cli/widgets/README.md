# `deepagents_cli/widgets/` — TUI Widgets

All Textual widget classes used in the CLI live here. Every visible element of the TUI is a widget or composed from widgets.

---

## Widget Inventory

| File | Widget | Role |
|---|---|---|
| [`chat_input.py`](chat_input.md) | `ChatInput` | Multiline text input with slash-command autocomplete |
| [`messages.py`](messages.md) | `UserMessage`, `AssistantMessage`, `ToolCallMessage`, `DiffMessage`, `ErrorMessage` | Transcript message types |
| [`message_store.py`](message_store.md) | `MessageStore` | In-memory registry of all displayed messages |
| [`approval.py`](approval.md) | `ApprovalMenu` | Modal for HITL tool-call approval |
| [`ask_user.py`](ask_user.md) | `AskUserMenu` | Modal for agent-initiated questions |
| [`status.py`](status.md) | `StatusBar` | Top bar (model, tokens, mode, spinner) |
| [`welcome.py`](welcome.md) | `WelcomeBanner` | Initial onboarding screen |
| [`agent_selector.py`](README.md) | `AgentSelector` | `/agents` modal |
| [`model_selector.py`](README.md) | `ModelSelector` | `/model` modal |
| [`thread_selector.py`](README.md) | `ThreadSelector` | `/threads` modal |
| [`theme_selector.py`](README.md) | `ThemeSelector` | `/theme` modal |
| [`mcp_viewer.py`](README.md) | `MCPViewer` | `/mcp` modal — MCP server status |
| [`notification_center.py`](README.md) | `NotificationCenter` | `/notifications` modal — async task updates |
| [`update_available.py`](README.md) | `UpdateAvailable` | Startup banner when a new CLI version is out |
| [`loading.py`](README.md) | `LoadingWidget` | Animated spinner during agent execution |
| [`tool_renderers.py`](README.md) | Various renderers | Rich formatting for specific tool outputs |
| [`tool_widgets.py`](README.md) | `ToolCallDisplay` | Reusable display for tool call + result |
| [`autocomplete.py`](README.md) | `AutocompleteOverlay` | Floating autocomplete dropdown for `ChatInput` |
| [`diff.py`](README.md) | `DiffView` | Side-by-side or unified diff rendering |
| [`history.py`](README.md) | `HistoryBrowser` | Input history navigation |

---

## Widget Hierarchy (as rendered in CLIApp)

```
Screen
└── CLIApp
    ├── StatusBar                      (fixed top)
    ├── WelcomeBanner                  (hidden after first message)
    ├── VerticalScroll
    │   ├── UserMessage
    │   ├── AssistantMessage
    │   │   └── (streaming text via Markdown widget)
    │   ├── ToolCallMessage
    │   │   ├── ToolCallDisplay (call side)
    │   │   └── ToolCallDisplay (result side)
    │   └── DiffMessage / ErrorMessage
    ├── ChatInput
    │   └── AutocompleteOverlay        (floating, shown on "/" prefix)
    └── [Modal layer — push_screen()]
        ├── ApprovalMenu
        ├── AskUserMenu
        ├── AgentSelector
        ├── ModelSelector
        ├── ThreadSelector
        ├── ThemeSelector
        ├── MCPViewer
        └── NotificationCenter
```

---

## Choosing Between Selector Modals

All selector modals (`AgentSelector`, `ModelSelector`, `ThreadSelector`, `ThemeSelector`) follow the same pattern:
- Shown via `app.push_screen(modal)` 
- Return their selection (or `None` for cancel) via Textual's screen result mechanism
- Use a `ListView` or `DataTable` for item selection with keyboard navigation (Up/Down arrows, Enter to confirm, Escape to cancel)

---

## See Also

- [app.md](../app.md) — creates and uses all modals
- [remote_client.md](../remote_client.md) — produces `UIAction`s that mutate message widgets
- [approval.md](approval.md) — the most complex modal (HITL flow)
