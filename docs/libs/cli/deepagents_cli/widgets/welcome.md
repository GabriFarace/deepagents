# `widgets/welcome.py`

## High-Level Purpose

This module defines the `WelcomeBanner` widget, which is displayed at the top of the chat when the CLI starts. It shows the Deep Agents ASCII art logo, version information, the current thread ID, connected LangSmith project, loaded MCP tool count, and a rotating tip.

## Module-Level Constants

| Constant | Description |
|---|---|
| `_TIPS` | List of 12 rotating tips shown in the welcome footer (one picked per session) |

**Tips include:** using `@` for file references, `/threads` for resuming conversations, `/offload` for long conversations, `/mcp` for tool listing, `/model` for model switching, `ctrl+x` for external editor, etc.

## Classes

### `WelcomeBanner`

**Inherits from:** `textual.widgets.Static`

The startup welcome banner.

**Class Variables:**
- `auto_links = False` — Disabled to prevent a flicker cycle caused by Textual's auto-link ID randomization.

**Constructor:**

```python
WelcomeBanner(
    thread_id: str | None = None,
    mcp_tool_count: int = 0,
    *,
    connecting: bool = False,
    resuming: bool = False,
    local_server: bool = False,
    **kwargs
)
```

**Parameters:**
| Parameter | Type | Description |
|---|---|---|
| `thread_id` | `str \| None` | Optional thread ID to display |
| `mcp_tool_count` | `int` | Number of MCP tools loaded at startup |
| `connecting` | `bool` | Show "Connecting..." state instead of thread info |
| `resuming` | `bool` | Show "Resuming..." state |
| `local_server` | `bool` | Whether running in local LangGraph server mode |

**Key Methods:**

- `render() -> RenderResult` — Renders the welcome banner with logo, version, thread ID, LangSmith project link (if configured), MCP tool count, and a randomly selected tip.
- `on_click(event: Click) -> None` — Opens the LangSmith project URL in the browser when the project link is clicked.

**Content sections:**
1. ASCII art logo (from `config.get_banner()`)
2. Version and thread ID (or connecting/resuming state)
3. LangSmith project link (if `LANGSMITH_API_KEY` is set)
4. MCP tool count (if any MCP tools are loaded)
5. Randomly selected tip from `_TIPS`

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `random` | stdlib | Random tip selection |
| `textual.widgets.Static` | textual | Base static widget |
| `theme` | `deepagents_cli.theme` | Brand colors |
| `__version__` | `deepagents_cli._version` | Version string |
| `get_banner`, `get_glyphs`, `fetch_langsmith_project_url` | `deepagents_cli.config` | Display helpers |
| `open_style_link` | `widgets._links` | Clickable link helper |
