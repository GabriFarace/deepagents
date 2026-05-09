# `examples/async-subagent-server/`

> A FastAPI Agent Protocol server used as a Deep Agents async subagent.

## Purpose

This example demonstrates how to run a subagent as a separate service. A
supervisor starts long-running research tasks, receives a task ID immediately,
and later checks, updates, lists, or cancels those tasks.

## Entry files

`server.py` is the FastAPI service. It implements the subset of Agent Protocol
endpoints used by the async subagent middleware: create thread, start run, poll
run, fetch thread state, cancel run, and health check.

`supervisor.py` is an interactive REPL. It creates a Deep Agent with an
`AsyncSubAgent` pointing at the server URL and prompts the model to use the
async task lifecycle tools correctly. `test_server.py` covers the service
behavior.

## Tools, subagents, and backends

The server-hosted researcher is itself a Deep Agent with an async `web_search`
tool. If `TAVILY_API_KEY` is present, the tool calls Tavily through `httpx`;
otherwise it falls back to simpler behavior.

The server persists thread and run state in an in-memory SQLite database. The
supervisor uses `MemorySaver` for local checkpointing and configures one
`AsyncSubAgent` named `researcher` with `graph_id`, `url`, and headers.

## Concept demonstrated

The important idea is protocol separation. The supervisor does not import or run
the researcher graph directly; it talks to a service that looks enough like a
LangGraph/Agent Protocol server for the async subagent middleware to manage
background tasks.
