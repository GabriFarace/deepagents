# `examples/deploy-gtm-agent/`

> A deployed go-to-market strategist that combines skills, MCP, and subagents.

## Purpose

This example builds a GTM planning agent. Given a product or feature, the
supervisor coordinates market research, positioning, channel strategy, and
supporting content work.

## Entry files

`deepagents.toml` configures the deployed supervisor. `AGENTS.md` defines the
strategy workflow. `mcp.json` adds external tool access. The top-level
`skills/competitor-analysis/` skill gives the supervisor a repeatable competitor
research workflow.

The nested `subagents/market-researcher/` directory is its own deploy-style
agent package, with `AGENTS.md`, `deepagents.toml`, and an `analyze-market`
skill.

## Tools, subagents, and backends

The main subagent is `market-researcher`, discovered from the `subagents/`
directory at deploy time. It writes a detailed market report to memory and
returns structured extracts for the supervisor to use.

The example also describes an async content-writer pattern in the README, while
the checked-in tree centers on the sync market researcher plus top-level skills
and MCP configuration.

## Concept demonstrated

This example shows deployment-time agent composition. The supervisor and its
subagent are represented as folders with their own config and instructions, so
the deploy runtime can assemble a multi-agent workflow from filesystem layout
rather than a single Python factory.
