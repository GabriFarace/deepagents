# `examples/nvidia_deep_agent/`

> A multi-model Deep Agent with NVIDIA-hosted research and GPU-oriented data skills.

## Purpose

This example shows a more production-shaped agent that mixes model providers,
subagent roles, skills, and backend routing. It is inspired by NVIDIA's AIQ
Blueprint but keeps the implementation in the Deep Agents idiom.

## Entry files

`src/agent.py` creates the graph. It selects a frontier orchestrator model with
`init_chat_model`, creates a `ChatNVIDIA` Nemotron Super model for research, and
passes two subagents to `create_deep_agent`.

`src/backend.py` builds the backend used for code execution and file access.
`src/prompts.py` holds orchestrator, researcher, and data-processor prompts.
`src/tools.py` defines the Tavily search tool. `langgraph.json` exposes the
agent for LangGraph tooling.

## Tools, subagents, and backends

The top-level tools include `tavily_search`. The `researcher-agent` uses
Nemotron Super plus search tools for web research. The `data-processor-agent`
uses the frontier model and `/skills/` for analytics, machine learning,
visualization, and document processing workflows.

The backend is supplied as `create_backend`, allowing per-run context to choose
GPU or CPU sandbox behavior. The `Context` schema exposes `sandbox_type`, and
the skills directory contains GPU-oriented workflows for cuDF, cuML,
visualization, and document extraction.

## Concept demonstrated

This is the best example of provider-agnostic orchestration. The supervisor,
researcher, and processor can use different model backends, while the Deep
Agents graph presents one coherent agent with routed execution resources.
