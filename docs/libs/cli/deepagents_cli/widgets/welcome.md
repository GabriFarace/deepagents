# `deepagents_cli/widgets/welcome.py`

## High-Level Purpose

`WelcomeBanner` is the initial screen shown when the TUI starts, before the user has sent any message. It fills the transcript area with onboarding information. As soon as the user submits their first message, the banner is unmounted and replaced by the message transcript.

---

## Key Class

### `WelcomeBanner(Widget)`

**Content displayed:**

- **Project context** — current working directory and git branch
- **Active agent** — agent name (or "default agent" if none selected)
- **Model** — resolved model name and provider
- **MCP servers** — list of configured MCP servers with their status (connected / error)
- **Tips** — 2-3 contextual hints (e.g., "Type `/agents` to switch agents", "Use `@file` to attach files")
- **Available slash commands** — abbreviated list with descriptions
- **Version** — CLI version number

**Visibility logic:** `CLIApp` adds `WelcomeBanner` to the `VerticalScroll` on mount. When the first `UserMessage` is about to be appended, `CLIApp` calls `welcome_banner.remove()` first.

---

## Architecture Notes

**MCP server status:** `WelcomeBanner` reads `MCPServerInfo` objects passed in from `server_kwargs`. If servers are listed but connection failed (status `"error"`), the banner shows the error message in red to alert the user before they start chatting.

**Async data:** The banner renders synchronously at mount time with whatever data is available. If MCP loading is still in progress (rare, since server startup is awaited), it shows "loading…" and updates via a reactive property once the data arrives.

---

## See Also

- [app.md](../app.md) — mounts and unmounts the banner
- [mcp_tools.md](../mcp_tools.md) — provides `MCPServerInfo` list
