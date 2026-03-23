# `nvidia_deep_agent/src/prompts.py`

## High-Level Purpose

Contains system prompt templates for the three roles in the NVIDIA Deep Agent: orchestrator, researcher sub-agent, and data processor sub-agent. Adapted from NVIDIA's AIQ Blueprint (`orchestrator.j2`, `researcher.j2`) and the LangChain deep_research example prompts.

## Module-Level Constants

### `ORCHESTRATOR_INSTRUCTIONS`

System prompt for the top-level orchestrator agent. Template with a `{date}` parameter.

**Content:** Minimal; positions the agent as a "Deep Agent" handling research, data analysis, and optimization tasks. Instructs it to produce thorough, well-structured outputs tailored to the user's request. Delegates research to the researcher sub-agent and data work to the data processor sub-agent.

---

### `RESEARCHER_INSTRUCTIONS`

System prompt for the researcher sub-agent. Template with a `{date}` parameter.

**Key sections:**
- **Research Protocol**: 5-step process — read the question, start broad, reflect after each search, narrow searches, stop when confident.
- **Guidelines**: Cross-reference sources, seek "why/how" explanations, synthesize across sources.
- **Depth Requirements**: Include specific facts/figures/dates, explain concepts thoroughly, retain richness and detail.
- **Tool Call Budget**: 2-3 calls for simple queries, 5-8 for complex. Stop immediately if: question can be answered comprehensively, 3+ relevant sources found, or last 2 searches returned similar info.
- **Output Format**: Structured with "Query Topic", "Research Notes" (subsections with inline citations), and "Sources". Output is written to `/shared/[query_topic].txt` via `write_file`.

---

### `DATA_PROCESSOR_INSTRUCTIONS`

System prompt for the data processor sub-agent. Template with a `{date}` parameter.

**Role:** GPU-accelerated data processing specialist using NVIDIA RAPIDS inside a Modal sandbox.

**Key sections:**
- **Available Skills**: cudf-analytics, cuml-machine-learning, data-visualization, gpu-document-processing. **Must** read the relevant `SKILL.md` via `read_file` before writing any code.
- **Workflow**: 7-step process — understand task → read skills → write script to `/workspace/[name].py` → execute → display charts (via `read_file` for inline display) → review output → iterate if needed → write findings to `/shared/[task_topic].txt`.
- **Code Execution Guidelines**: Always use GPU-accelerated libraries (cuDF, cuML) first. Keep stdout under 10KB. Never print entire DataFrames. Handle errors with try/except.
- **Output Format**: Structured with "Task Topic", "Summary", "Results", and "Insights".
- **Self-Improvement**: After resolving errors or discovering undocumented library behavior, immediately update the relevant `/skills/<skill-name>/SKILL.md` via `edit_file`. Specifies what to save (API anomalies, working patterns, known limitations, error fixes) and what not to save (one-off input errors, transient errors, unvalidated speculation).
