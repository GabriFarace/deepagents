# `widgets/approval.py`

## High-Level Purpose

This module defines the `ApprovalMenu` widget for the Human-in-the-Loop (HITL) approval system. It displays tool call details and presents the user with options to approve, auto-approve all future calls, or reject a tool call. It uses a tool renderer pattern to show tool-specific previews (e.g., file diffs, command details).

## Classes

### `ApprovalMenu`

**Inherits from:** `textual.containers.Container`

The main HITL approval widget. Displays tool call information and captures user decisions.

**Design decisions:**
- Uses `Container` base class with `compose()` rather than `Widget`.
- Keybindings for navigation (not `on_key` handlers).
- `can_focus = True`, `can_focus_children = False` to prevent focus theft.
- Tool-specific widgets via the renderer pattern (`tool_renderers.get_renderer`).

**Class Variables:**

| Attribute | Type | Description |
|---|---|---|
| `can_focus` | `bool` | `True` — widget can receive keyboard focus |
| `can_focus_children` | `bool` | `False` — children don't steal focus |
| `BINDINGS` | `list[BindingType]` | See below |
| `_MINIMAL_TOOLS` | `frozenset[str]` | Shell tool names that don't need detailed display |

**Bindings:**
| Key | Action | Description |
|---|---|---|
| `up / k` | `move_up` | Navigate options up |
| `down / j` | `move_down` | Navigate options down |
| `enter` | `select` | Select highlighted option |
| `1 / y` | `select_approve` | Approve this tool call |
| `2 / a` | `select_auto` | Auto-approve all future calls |
| `3 / n` | `select_reject` | Reject this tool call |
| `e` | `toggle_expand` | Toggle expanded command view |

#### Inner Message: `Decided`

Posted when the user makes an approval decision.

| Attribute | Type | Description |
|---|---|---|
| `decision` | `dict[str, str]` | Decision dict with type: `'approve'`, `'reject'`, or `'auto_approve_all'` |

#### Constructor

```python
ApprovalMenu(
    action_requests: list[dict] | dict,
    _assistant_id: str | None = None,
    id: str | None = None,
    **kwargs
)
```

**Parameters:**
- `action_requests`: One or more tool action request dicts (each has `tool_name` and `tool_args`).
- `_assistant_id`: Optional assistant ID for context.

## Display Logic

For each tool call:
1. The tool name and truncated arguments are displayed in the header.
2. For non-shell tools, a tool-specific widget (from the renderer registry) shows a preview (e.g., file diff, write content).
3. Unicode security warnings are displayed if dangerous characters are detected in tool arguments.
4. Three option buttons are shown: Approve, Auto-approve all, Reject.

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `textual` widgets/containers | textual | UI base classes |
| `theme` | `deepagents_cli.theme` | Brand colors |
| `SHELL_TOOL_NAMES`, `get_glyphs`, `is_ascii_mode` | `deepagents_cli.config` | Constants |
| `check_url_safety`, `detect_dangerous_unicode`, etc. | `deepagents_cli.unicode_security` | Security checks |
| `get_renderer` | `deepagents_cli.widgets.tool_renderers` | Tool-specific widget factory |
