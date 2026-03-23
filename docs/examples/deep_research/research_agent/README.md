# `deep_research/research_agent/`

## What This Directory Contains

Source code for the research sub-agent used by the deep research orchestrator. Provides the system prompts, web search tool, and strategic reflection tool that define the sub-agent's research behavior.

## Files

| File | Description |
|------|-------------|
| `prompts.py` | All prompt templates: orchestrator workflow, sub-agent instructions, delegation strategy |
| `tools.py` | `tavily_search` (Tavily + full page content) and `think_tool` (chain-of-thought reflection) |
| `__init__.py` | Re-exports prompts and tools for clean imports from the parent module |

## How the Research Sub-Agent Works

1. Receives a focused research topic from the orchestrator.
2. Runs `tavily_search` to discover and fetch full webpage content as markdown.
3. After each search, calls `think_tool` to verbalize current findings, what is missing, and whether to continue.
4. Stops when confident (3+ relevant sources) or at budget limit (5 calls for complex queries).
5. Returns structured findings with inline citations and a `### Sources` section.

## Related Docs

- [`prompts.md`](prompts.md) — `RESEARCH_WORKFLOW_INSTRUCTIONS`, `RESEARCHER_INSTRUCTIONS`, `SUBAGENT_DELEGATION_INSTRUCTIONS`
- [`tools.md`](tools.md) — `tavily_search` and `think_tool`
