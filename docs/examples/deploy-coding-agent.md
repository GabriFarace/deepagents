# `examples/deploy-coding-agent/`

> A deployable coding assistant configured for `deepagents deploy`.

## Purpose

This example shows the deployment shape for an autonomous coding agent. The
agent plans, edits, tests, reviews, and delivers changes inside a LangSmith
sandbox rather than the user's local machine.

## Entry files

There is no custom Python builder. `deepagents.toml` declares the deployment
configuration, including model and sandbox settings. `deepagents.assistant-scope.toml`
is a variant for assistant-scoped configuration. `AGENTS.md` contains the main
coding workflow and behavior rules.

`mcp.json` configures MCP servers available to the deployment. The `skills/`
directory adds focused behaviors: planning, coding preferences, and code review
with a lint helper.

## Tools, subagents, and backends

The notable backend is the LangSmith coding sandbox declared in the deploy
config. That gives the deployed agent shell and filesystem access in an isolated
environment.

The example is skill-heavy and subagent-light: its specialization comes from
`AGENTS.md`, `skills/code-review/`, `skills/coding-prefs/`, and
`skills/planning/`, plus whatever tools the deployment runtime exposes from the
sandbox and MCP configuration.

## Concept demonstrated

This is the minimal "coding agent as a deployed service" pattern. Instead of
writing Python construction code, the example relies on the deploy runtime to
discover configuration files, load skills, provision the sandbox, and expose the
agent through LangSmith/LangGraph APIs.
