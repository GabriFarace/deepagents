# `examples/deep_research/`

> A web research agent that delegates focused searches to a researcher subagent and synthesizes cited reports.

## Purpose

This example is the most direct SDK demonstration of a "deep research" pattern.
The top-level agent owns planning, task decomposition, citation consolidation,
and final synthesis. It delegates evidence gathering to a specialized
`research-agent` with its own prompt and tools.

## Entry files

`agent.py` is the LangGraph entrypoint used by `langgraph.json`. It builds the
prompt, selects the model, declares the research subagent, and calls
`create_deep_agent`.

`research_agent/prompts.py` contains the workflow, delegation, and researcher
instructions. `research_agent/tools.py` defines `tavily_search`, webpage content
fetching, and `think_tool`. `research_agent.ipynb` is an interactive walkthrough,
and `utils.py` contains notebook display helpers.

## Tools, subagents, and backends

The top-level agent and subagent both receive `tavily_search` and `think_tool`.
`tavily_search` uses Tavily for URL discovery, fetches page content, and returns
source material for the model to inspect. `think_tool` gives the researcher an
explicit reflection step between searches.

The `research-agent` subagent receives one topic at a time and is capped by
prompt-level guidance for search count, concurrent research units, and iteration
count. No custom backend is configured, so this example relies on the SDK's
default backend behavior rather than a host filesystem or sandbox.

## Concept demonstrated

The key idea is agent specialization without a separate service. One
`create_deep_agent` call wires a supervisor-style prompt, shared tools, and a
subagent prompt into a single compiled graph. The example also shows that
subagent count should be controlled by the task shape: simple questions use one
researcher, while explicit comparisons can fan out across several focused
research tasks.
