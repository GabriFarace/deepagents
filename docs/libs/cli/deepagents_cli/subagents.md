# `libs/cli/deepagents_cli/subagents.py`

## High-Level Purpose

`subagents.py` loads custom subagent definitions from the user's agent directory (`~/.deepagents/{agent_name}/agents/`). Each subagent is a Markdown file with YAML frontmatter. This module parses those files into `SubAgent` TypedDicts that are passed to `create_cli_agent()`, enabling the agent to delegate tasks to specialized sub-agents.

---

## Key Function

### `load_subagents(agent_name=None) → list[SubAgent]`

Scans `~/.deepagents/{agent_name}/agents/` (or `~/.deepagents/agents/` if no agent name) for `*.md` files. Parses each as a subagent definition and returns the list.

Returns `[]` if the directory doesn't exist.

---

## Subagent File Format

Each file in `agents/` is a Markdown file with YAML frontmatter:

```markdown
---
name: researcher
description: |
  Deep research agent. Use for multi-step web research tasks.
  Provide a research question; returns a comprehensive report.
model: claude-opus-4-7
tools:
  - web_search
  - fetch_url
---

You are a systematic research agent. Your task is to...
```

**Frontmatter fields:**

| Field | Required | Description |
|---|---|---|
| `name` | Yes | Identifier used in `task(subagent_type="researcher")` |
| `description` | Yes | Used by the parent agent to decide when to delegate |
| `model` | No | Override model for this subagent |
| `tools` | No | Restrict tools (omit to inherit parent's tools) |
| `skills` | No | List of skill names to load |
| `interrupt_on` | No | List of tool names requiring approval |

The file body (below the frontmatter) becomes the subagent's system prompt.

---

## `_parse_subagent_file(path) → SubAgent | None`

Reads a single file, extracts YAML frontmatter and Markdown body, and returns a `SubAgent` dict. Returns `None` if the file is malformed (with a warning log).

---

## Architecture Notes

**General-purpose subagent:** The SDK's `create_deep_agent()` auto-adds a general-purpose subagent unless one with the name `"general-purpose"` is already in the list, or `add_general_purpose_subagent=False` is passed. User-defined subagents from this module are passed alongside the auto-added one.

**File-based discovery:** Subagent files are read at server startup (in `server_graph.py`). Adding a new `agents/*.md` file requires restarting the CLI to take effect.

---

## See Also

- [agent.md](agent.md) — receives the loaded subagent list
- [server_graph.md](server_graph.md) — calls `load_subagents()`
- [../../deepagents/deepagents/middleware/subagents.md](../../deepagents/deepagents/middleware/subagents.md) — SDK `SubAgent` TypedDict
