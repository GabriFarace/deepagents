# `libs/cli/deepagents_cli/command_registry.py`

## High-Level Purpose

`command_registry.py` is the single source of truth for all slash commands. It defines the `SlashCommand` dataclass and the `COMMANDS` tuple that lists every available command with its metadata. The `ChatInput` widget reads `COMMANDS` to build autocomplete suggestions. `CLIApp` reads `BypassTier` to decide whether a command can execute immediately or must wait for the agent to finish.

---

## Key Types

### `BypassTier` (enum)

Controls when a slash command can execute relative to the agent's busy state.

| Value | When it can run | Examples |
|---|---|---|
| `ALWAYS` | Unconditionally, even during agent execution | `/quit` |
| `CONNECTING` | Only while the server is still starting | `/version` (shows during startup) |
| `IMMEDIATE_UI` | Opens a modal immediately, regardless of agent state | `/agents`, `/model`, `/theme` |
| `SIDE_EFFECT_FREE` | Side effect fires immediately, UI update deferred | `/mcp`, `/trace`, `/tokens`, `/changelog` |
| `QUEUED` | Must wait until the agent is idle | `/clear`, `/editor`, `/remember` |

### `SlashCommand` (dataclass)

```python
@dataclass
class SlashCommand:
    name: str                    # "/command" format
    description: str             # shown in autocomplete
    bypass_tier: BypassTier
    aliases: list[str] = field(default_factory=list)   # e.g., ["/q"] for "/quit"
    hidden_keywords: list[str] = field(default_factory=list)  # fuzzy match extras
    argument_hint: str = ""      # e.g., "[thread_id]"
    hidden: bool = False         # if True, not shown in autocomplete
```

---

## Built-in Commands

| Command | Aliases | Tier | Description |
|---|---|---|---|
| `/agents` | — | `IMMEDIATE_UI` | Switch to a different agent |
| `/clear` | — | `QUEUED` | Start a new thread (clears conversation) |
| `/editor` | — | `QUEUED` | Open the current prompt in `$EDITOR` |
| `/help` | — | `IMMEDIATE_UI` | Show help screen |
| `/mcp` | — | `SIDE_EFFECT_FREE` | View MCP server status |
| `/model` | — | `IMMEDIATE_UI` | Switch or configure the model |
| `/notifications` | — | `IMMEDIATE_UI` | View async task notifications |
| `/quit` | `/q`, `/exit` | `ALWAYS` | Exit the CLI |
| `/remember` | — | `QUEUED` | Alias for the built-in `remember` skill |
| `/skill:<name>` | — | `QUEUED` | Run a named skill |
| `/skill-creator` | — | `QUEUED` | Alias for the built-in `skill-creator` skill |
| `/theme` | — | `IMMEDIATE_UI` | Change the color theme |
| `/threads` | — | `IMMEDIATE_UI` | Resume a past conversation thread |
| `/tokens` | — | `SIDE_EFFECT_FREE` | Print token usage for current session |
| `/trace` | — | `SIDE_EFFECT_FREE` | Print LangSmith trace URL |
| `/version` | — | `CONNECTING` | Show CLI version |
| `/changelog` | — | `SIDE_EFFECT_FREE` | Show recent changelog |

---

## Helper Functions

### `get_command(name) → SlashCommand | None`

Looks up a command by its primary name or any alias. Returns `None` if not found.

### `get_all_autocomplete_entries() → list[AutocompleteEntry]`

Returns all non-hidden commands as autocomplete entries for `ChatInput`. Also includes dynamically discovered skill names (e.g., `/skill:web-research`).

### `parse_skill_command(text) → tuple[str, str] | None`

Parses a `/skill:name args` string. Returns `(skill_name, args)` or `None` if the text is not a skill command.

---

## Architecture Notes

**Static registry:** `COMMANDS` is a module-level tuple, not a dynamic registry. New commands require a code change here plus a handler in `app.py`. This keeps autocomplete fast and the command set predictable.

**Skill commands:** `/skill:<name>` is not a fixed entry in `COMMANDS` — it's a prefix convention. The `ChatInput` widget discovers skill names at runtime by scanning the skills directories and appending them to the autocomplete list as `/skill:web-research`, `/skill:code-review`, etc.

**Bypass tier rationale:** `IMMEDIATE_UI` commands (like `/model`) don't need the agent to finish — they just open a modal. `QUEUED` commands (like `/clear`) must wait because they modify agent state (starting a new thread while the agent is running would corrupt the thread).

---

## See Also

- [app.md](app.md) — `_classify_command()` and `_handle_command()` use `COMMANDS`
- [widgets/chat_input.md](widgets/chat_input.md) — `ChatInput` reads `COMMANDS` for autocomplete
- [hooks.md](hooks.md) — commands can trigger hooks
