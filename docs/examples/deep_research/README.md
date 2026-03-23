# `examples/deep_research/`

## What This Example Does

Demonstrates a deep research agent with an orchestrator/sub-agent architecture. The orchestrator plans a research workflow and delegates focused research tasks to parallel sub-agents; each sub-agent uses Tavily web search and a reflection tool to gather comprehensive information, which the orchestrator then synthesizes into a final report.

## Files

| File/Directory | Description |
|----------------|-------------|
| `agent.py` | Orchestrator agent entry point; exports `agent` for LangGraph deployment |
| `research_agent/prompts.py` | Prompt templates for orchestrator and sub-agent |
| `research_agent/tools.py` | `tavily_search` and `think_tool` |
| `research_agent/__init__.py` | Re-exports prompts and tools |
| `research_agent.ipynb` | Jupyter notebook walkthrough (interactive) |
| `pyproject.toml` | Package metadata and dependencies |
| `README.md` | Original user-facing README (quickstart, usage, customization) |

## Architecture

```
User Research Query
    │
    ▼
Orchestrator (Claude Sonnet 4.5)
    ├── Creates todo list with write_todos
    ├── Saves research request to /research_request.md
    ├── Delegates parallel research tasks (max 3 concurrent)
    │
    └── Sub-agents (research-agent, up to 3 parallel, max 3 rounds)
        ├── tools: [tavily_search, think_tool]
        └── model: Claude Sonnet 4.5
    │
    ▼
Orchestrator synthesizes findings
    └── Writes final report to /final_report.md with inline citations
```

## How to Run

**Option 1: LangGraph server**
```bash
cd examples/deep_research
uv sync
langgraph dev
```

**Option 2: Jupyter notebook**
```bash
uv run jupyter notebook research_agent.ipynb
```

## Environment Variables Required

| Variable | Purpose |
|----------|---------|
| `ANTHROPIC_API_KEY` | Claude model API key (required) |
| `TAVILY_API_KEY` | Web search API key (required) |
| `LANGSMITH_API_KEY` | LangSmith tracing (optional) |

## Key Design Decisions

- **Tavily as URL discovery**: Tavily is used only to find relevant URLs; full page content is fetched separately via httpx + markdownify to preserve all information without truncation.
- **`think_tool` as chain-of-thought**: The think tool is a no-op that forces the researcher sub-agent to verbalize its assessment of progress before deciding whether to continue searching.
- **Report structure**: The orchestrator follows specific patterns for comparisons, lists/rankings, and summaries/overviews, with inline `[N]` citations and a deduplicated `### Sources` section.
- **Configurable limits**: `max_concurrent_research_units=3` and `max_researcher_iterations=3` control how aggressively the orchestrator parallelizes and how many rounds it runs.

## Related Docs

- [`agent.md`](agent.md) — Orchestrator configuration and `INSTRUCTIONS` constant
- [`research_agent/README.md`](research_agent/README.md) — Sub-agent package overview
- [`research_agent/prompts.md`](research_agent/prompts.md) — All three prompt template details
- [`research_agent/tools.md`](research_agent/tools.md) — `tavily_search` and `think_tool`
