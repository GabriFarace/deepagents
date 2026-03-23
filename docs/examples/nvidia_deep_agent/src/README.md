# `nvidia_deep_agent/src/`

## What This Directory Contains

Source code for the NVIDIA Deep Agent — a multi-model agent architecture that combines a frontier LLM orchestrator with NVIDIA Nemotron Super for research and a GPU-equipped data processor sub-agent running on Modal with NVIDIA RAPIDS.

## Files

| File | Description |
|------|-------------|
| `agent.py` | Main entry point; assembles the three-role agent architecture and exports `agent` as the LangGraph-deployable graph |
| `backend.py` | Modal sandbox factory; handles GPU/CPU image selection, sandbox creation/reuse, and seeding with skills and memory files |
| `prompts.py` | System prompt templates for all three agent roles: orchestrator, researcher, data processor |
| `tools.py` | `tavily_search` tool using Tavily + httpx + markdownify for full webpage content retrieval |
| `__init__.py` | Package marker (empty) |

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
            └── skills: ["/skills/"]   (cuDF, cuML, visualization, document processing)
```

## Key Design Decisions

- **Multi-model**: The orchestrator and researcher use different models — the researcher uses Nemotron Super, a fast NVIDIA OSS model optimized for information retrieval.
- **GPU sandbox**: The data processor sub-agent runs inside a Modal sandbox with an NVIDIA A10G GPU and the RAPIDS image (cuDF/cuML) for accelerated data processing.
- **Skill self-improvement**: The data processor is instructed to update SKILL.md files when it discovers API anomalies or workarounds, enabling the agent to improve its own context over time.
- **Runtime context switching**: The `Context` TypedDict allows callers to pass `context={"sandbox_type": "cpu"}` to run without GPU for testing or cost reduction.
- **File-based memory**: Skills and the AGENTS.md persona file are seeded from the local filesystem into the sandbox on first creation; in production, replace with S3 or a database.

## Related Docs

- [`agent.md`](agent.md) — Module-level configuration, sub-agent definitions, `Context` TypedDict
- [`backend.md`](backend.md) — Modal image configuration, sandbox factory, `_seed_sandbox`
- [`prompts.md`](prompts.md) — All three prompt templates with detailed section breakdowns
- [`tools.md`](tools.md) — `tavily_search` and `fetch_webpage_content` tool docs
