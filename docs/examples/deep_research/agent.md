# `deep_research/agent.py`

## High-Level Purpose

The main entry point for the deep research agent deployed with LangGraph. Assembles the orchestrator agent with a research sub-agent, model configuration, and combined system prompt instructions. This module exports `agent` as the LangGraph-deployable graph object.

## Module-Level Configuration

| Variable | Value | Description |
|----------|-------|-------------|
| `max_concurrent_research_units` | `3` | Maximum parallel sub-agent delegations per iteration |
| `max_researcher_iterations` | `3` | Maximum delegation rounds before stopping |
| `current_date` | `datetime.now().strftime("%Y-%m-%d")` | Current date injected into prompts |

## Module-Level Objects

### `INSTRUCTIONS`

The combined orchestrator system prompt, assembled by concatenating:
1. `RESEARCH_WORKFLOW_INSTRUCTIONS` (todo list, save request, delegate, synthesize, write report, verify)
2. `SUBAGENT_DELEGATION_INSTRUCTIONS` (formatted with `max_concurrent_research_units` and `max_researcher_iterations`)

### `research_sub_agent`

A sub-agent configuration dict:

```python
{
    "name": "research-agent",
    "description": "Delegate research to the sub-agent researcher...",
    "system_prompt": RESEARCHER_INSTRUCTIONS.format(date=current_date),
    "tools": [tavily_search, think_tool],
}
```

### `model`

`init_chat_model("anthropic:claude-sonnet-4-5-20250929", temperature=0.0)`

### `agent`

The main LangGraph-deployable agent, created with:
- `model`: Claude Sonnet 4.5
- `tools`: `[tavily_search, think_tool]`
- `system_prompt`: Combined orchestrator instructions
- `subagents`: `[research_sub_agent]`

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| `create_deep_agent` | `deepagents` | Agent factory |
| `init_chat_model` | `langchain.chat_models` | Model-agnostic LLM initialization |
| `research_agent.prompts` | local | Prompt templates |
| `research_agent.tools` | local | `tavily_search`, `think_tool` |
