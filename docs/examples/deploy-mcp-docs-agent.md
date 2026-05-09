# `examples/deploy-mcp-docs-agent/`

> A deployed documentation researcher that searches live docs through MCP.

## Purpose

This example answers developer questions about LangChain, LangGraph, and Deep
Agents by consulting documentation tools before falling back to general model
knowledge.

## Entry files

`deepagents.toml` declares the deployed agent and model. `AGENTS.md` gives the
docs-first research policy and answer style. `mcp.json` points the deployment at
the LangChain docs MCP server.

## Tools, subagents, and backends

The tool surface is supplied by MCP, not local Python functions. The docs MCP
server at `https://docs.langchain.com/mcp` provides search and retrieval tools
that the deploy runtime discovers and exposes to the agent.

There are no custom subagents or custom backend code. The example is meant to
isolate the MCP integration path.

## Concept demonstrated

This is the simplest deployed MCP pattern: put server configuration in
`mcp.json`, tell the agent in `AGENTS.md` when and how to use the tools, and let
`deepagents deploy` wire the tool surface into the graph.
