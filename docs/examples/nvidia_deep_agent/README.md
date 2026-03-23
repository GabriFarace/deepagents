# `examples/nvidia_deep_agent/`

## What This Example Does

Demonstrates a multi-model agent architecture with GPU code execution. A frontier LLM orchestrates tasks, delegating research to NVIDIA Nemotron Super (via NIM) and data analysis/ML/visualization to a GPU-equipped data processor sub-agent running on Modal with NVIDIA RAPIDS.

## Files

| File/Directory | Description |
|----------------|-------------|
| `src/agent.py` | Main entry point; assembles the three-role agent and exports `agent` |
| `src/backend.py` | Modal sandbox factory; handles GPU/CPU image selection and file seeding |
| `src/prompts.py` | System prompt templates for orchestrator, researcher, and data processor |
| `src/tools.py` | `tavily_search` tool for web research |
| `src/AGENTS.md` | Persistent agent persona/memory file (seeded into sandbox) |
| `src/__init__.py` | Package marker |
| `skills/cudf-analytics/SKILL.md` | GPU data analysis skill (cuDF) |
| `skills/cuml-machine-learning/SKILL.md` | GPU ML skill (cuML) |
| `skills/data-visualization/SKILL.md` | Chart generation skill (matplotlib/seaborn) |
| `skills/gpu-document-processing/SKILL.md` | Large document processing skill |
| `pyproject.toml` | Package metadata and dependencies |
| `README.md` | Original user-facing README (quickstart, examples, architecture) |

## Architecture

```
User Prompt
    │
    ▼
Orchestrator (frontier_model: Claude Sonnet 4.6 by default)
    ├── memory=["/memory/AGENTS.md"]  (loaded from Modal sandbox)
    ├── tools=[tavily_search]
    ├── backend=create_backend        (Modal sandbox factory)
    └── Subagents:
        ├── researcher-agent
        │   ├── model: NVIDIA Nemotron Super (via NIM)
        │   └── tools: [tavily_search]
        └── data-processor-agent
            ├── model: frontier_model
            ├── tools: [tavily_search]
            └── skills: ["/skills/"]
```

## How to Run

```bash
uv run langgraph dev --allow-blocking
```

Then open LangSmith Studio and submit a query.

## Environment Variables Required

| Variable | Purpose |
|----------|---------|
| `ANTHROPIC_API_KEY` | Frontier model API key |
| `NVIDIA_API_KEY` | NVIDIA NIM API key (for Nemotron Super) |
| `TAVILY_API_KEY` | Web search API key |
| `LANGSMITH_API_KEY` | LangSmith tracing (optional) |
| `MODAL_TOKEN_ID` / `MODAL_TOKEN_SECRET` | Modal authentication (or use `modal setup`) |

## Related Docs

- [`src/README.md`](src/README.md) — Source directory overview
- [`src/agent.md`](src/agent.md) — Agent assembly and model configuration
- [`src/backend.md`](src/backend.md) — Modal sandbox setup and seeding
- [`src/prompts.md`](src/prompts.md) — All prompt templates
- [`src/tools.md`](src/tools.md) — Web search tool
