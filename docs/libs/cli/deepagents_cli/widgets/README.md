# `widgets/` — Textual UI Widgets

This directory contains all Textual widget components that compose the `deepagents-cli` interactive terminal interface (`DeepAgentsApp`).

## Overview

The widgets are built on the [Textual](https://textual.textualize.io/) framework and follow its patterns: `Widget`/`Container` subclasses that `compose()` child widgets, use `reactive` attributes for auto-updating state, post `Message` objects for cross-widget communication, and use CSS for styling.

## Files and Relationships

### Core Application Widgets

| File | Widget | Role |
|---|---|---|
| `chat_input.py` | `ChatInput` | Primary text input with autocomplete and history |
| `messages.py` | `UserMessage`, `AssistantMessage`, `ToolCallMessage`, etc. | Individual message renderers |
| `message_store.py` | `MessageStore`, `MessageData` | Virtualized message history data store |
| `status.py` | `StatusBar`, `ModelLabel` | Bottom status bar with mode/model/token display |
| `welcome.py` | `WelcomeBanner` | Startup banner with tips and session info |
| `loading.py` | `LoadingWidget`, `Spinner` | Animated thinking/loading indicator |

### Modal Screens

| File | Widget | Activated By |
|---|---|---|
| `model_selector.py` | `ModelSelectorScreen` | `/model` slash command |
| `thread_selector.py` | `ThreadSelectorScreen` | `/threads` slash command |
| `theme_selector.py` | `ThemeSelectorScreen` | `/theme` slash command |
| `mcp_viewer.py` | `MCPViewerScreen` | `/mcp` slash command |
| `agent_selector.py` | `AgentSelectorScreen` | `/agents` slash command |
| `ask_user.py` | `AskUserMenu` | `ask_user` tool during agent execution |
| `approval.py` | `ApprovalMenu` | Tool calls requiring HITL approval |

### Support Widgets and Utilities

| File | Purpose |
|---|---|
| `autocomplete.py` | `CompletionController` protocol, `SlashCommandController`, `FuzzyFileController`, `MultiCompletionManager` |
| `history.py` | `HistoryManager` — chat input history with JSON-lines persistence |
| `diff.py` | `compose_diff_lines` — per-line diff widget generation |
| `tool_widgets.py` | Tool-specific approval preview widgets (`WriteFileApprovalWidget`, `EditFileApprovalWidget`, etc.) |
| `tool_renderers.py` | Registry pattern mapping tool names to approval widget classes |
| `_links.py` | `open_style_link` — clickable URL helper for Textual content |
| `__init__.py` | Package docstring only — import from submodules directly |

## Data Flow

```
User types → ChatInput (autocomplete, history)
          → on submit → DeepAgentsApp.handle_submit()
                     → agent processes → TextualUIAdapter
                                       → AssistantMessage (streaming)
                                       → ToolCallMessage (with ApprovalMenu if HITL needed)
```

## Key Design Patterns

- **Virtualized history:** `MessageStore` stores all messages as `MessageData` dataclasses; only a window of widgets is in the DOM.
- **Modal screens:** `ModalScreen[T]` subclasses return typed values via `dismiss(value)`.
- **Approval flow:** `ApprovalMenu.Decided` message is posted when the user approves/rejects a tool call.
- **Autocomplete:** `MultiCompletionManager` delegates key events to the first active controller (`SlashCommandController` or `FuzzyFileController`).
- **Theme-aware diffs:** Diff line CSS classes (`.diff-line-added`, `.diff-line-removed`) use CSS variables so they update automatically on theme change.
