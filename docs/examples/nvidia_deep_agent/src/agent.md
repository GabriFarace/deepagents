# `nvidia_deep_agent/src/agent.py`

## High-Level Purpose

Main entry point for the NVIDIA Deep Agent. Assembles a multi-model agent architecture with a frontier model as orchestrator, NVIDIA Nemotron Super for research tasks, and a GPU-equipped data processor sub-agent. Exports `agent` as the LangGraph-deployable graph object.

## Module-Level Configuration

| Variable | Value | Description |
|----------|-------|-------------|
| `current_date` | `datetime.now().strftime("%Y-%m-%d")` | Current date injected into prompts |
| `frontier_model` | `init_chat_model(ORCHESTRATOR_MODEL env var, default "anthropic:claude-sonnet-4-6")` | Orchestrator and data processor model |
| `nemotron_super` | `ChatNVIDIA(model="nvidia/nemotron-3-super-120b-a12b")` | NVIDIA NIM model for research tasks |
| `tools` | `[tavily_search]` | Shared tool list for all agents |

## Classes

### `Context`

**Purpose:** TypedDict defining runtime context options passed via `context=` at invoke time.

**Fields:**
- `sandbox_type: Literal["gpu", "cpu"]` (optional) — Controls whether the Modal sandbox uses the RAPIDS GPU image or a CPU-only image. Defaults to `"gpu"` when omitted.

## Module-Level Objects

### `researcher_sub_agent`

A sub-agent configuration dict for web research tasks:

```python
{
    "name": "researcher-agent",
    "description": "...",
    "system_prompt": RESEARCHER_INSTRUCTIONS.format(date=current_date),
    "tools": [tavily_search],
    "model": nemotron_super,
}
```

Uses NVIDIA Nemotron Super as its model rather than the frontier model — optimized for fast information retrieval and synthesis.

### `data_processor_sub_agent`

A sub-agent configuration dict for data analysis, ML, visualization, and document processing:

```python
{
    "name": "data-processor-agent",
    "description": "...",
    "system_prompt": DATA_PROCESSOR_INSTRUCTIONS.format(date=current_date),
    "tools": [tavily_search],
    "model": frontier_model,
    "skills": ["/skills/"],
}
```

Has access to GPU skills (cuDF analytics, cuML ML, data visualization, GPU document processing) loaded from `/skills/` inside the sandbox. Optionally supports HITL via `interrupt_on={"execute": True}` (commented out).

### `agent`

The main LangGraph-deployable orchestrator, created with:
- `model`: frontier model (Claude Sonnet 4.6 by default)
- `tools`: `[tavily_search]`
- `system_prompt`: `ORCHESTRATOR_INSTRUCTIONS` formatted with current date
- `subagents`: `[researcher_sub_agent, data_processor_sub_agent]`
- `memory`: `["/memory/AGENTS.md"]` — loaded from sandbox filesystem
- `backend`: `create_backend` factory function
- `context_schema`: `Context`

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| `create_deep_agent` | `deepagents` | Agent factory |
| `init_chat_model` | `langchain.chat_models` | Model-agnostic LLM initialization |
| `ChatNVIDIA` | `langchain_nvidia_ai_endpoints` | NVIDIA NIM model client |
| `create_backend` | `src.backend` | Modal sandbox factory |
| Prompt constants | `src.prompts` | `ORCHESTRATOR_INSTRUCTIONS`, `RESEARCHER_INSTRUCTIONS`, `DATA_PROCESSOR_INSTRUCTIONS` |
| `tavily_search` | `src.tools` | Web search tool |
