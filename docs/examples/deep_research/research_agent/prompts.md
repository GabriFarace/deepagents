# `deep_research/research_agent/prompts.py`

## High-Level Purpose

Contains all prompt templates for the deep research agent system. Defines the orchestrator's research workflow, the sub-agent researcher's behavior, and the delegation strategy for coordinating parallel research tasks.

## Module-Level Constants

### `RESEARCH_WORKFLOW_INSTRUCTIONS`

The orchestrator's 6-step research workflow prompt:
1. Create a todo list with `write_todos`
2. Save the research question to `/research_request.md`
3. Delegate research to sub-agents via the `task()` tool (never research directly)
4. Synthesize sub-agent findings and deduplicate citations
5. Write a comprehensive report to `/final_report.md`
6. Verify the report covers all aspects of the request

Includes detailed report structure patterns (comparisons, lists/rankings, summaries/overviews) and citation format guidelines (inline `[N]` references, `### Sources` section).

---

### `RESEARCHER_INSTRUCTIONS`

System prompt for the research sub-agent. A template with a `{date}` parameter.

Key sections:
- **Task**: Use tools to gather information on the given topic.
- **Available Tools**: `tavily_search` (web search) and `think_tool` (reflection).
- **Instructions**: Think like a time-constrained human researcher — broad searches first, narrow down, stop when confident.
- **Hard Limits**: 2-3 calls for simple queries, 5 max for complex; stop when you have 3+ relevant sources or last 2 searches returned similar info.
- **Show Your Thinking**: Use `think_tool` after each search.
- **Final Response Format**: Structure findings with headings, inline citations, and a `### Sources` section.

---

### `TASK_DESCRIPTION_PREFIX`

Template prefix for the task tool description, formatted with `{other_agents}`.

---

### `SUBAGENT_DELEGATION_INSTRUCTIONS`

Orchestrator instructions for delegating to sub-agents. Template with `{max_concurrent_research_units}` and `{max_researcher_iterations}`.

Key principles:
- Default: use 1 sub-agent unless the query explicitly requires comparison or independent aspects.
- Use multiple sub-agents only for explicit comparisons (e.g., "Compare X vs Y") or clearly separated aspects.
- Limit: at most `max_concurrent_research_units` parallel delegations per iteration.
- Stop after `max_researcher_iterations` rounds.
