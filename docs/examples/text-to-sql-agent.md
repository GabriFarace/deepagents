# `examples/text-to-sql-agent/`

> A natural-language SQL agent over the Chinook SQLite database.

## Purpose

This example adapts Deep Agents to a constrained data task: answer user
questions by inspecting a database schema, planning a query, executing SQL, and
explaining the result.

## Entry files

`agent.py` is both the builder and command-line entrypoint. Its
`create_sql_deep_agent()` function connects to `chinook.db`, creates a
LangChain `SQLDatabaseToolkit`, and passes the toolkit tools to
`create_deep_agent`.

`AGENTS.md` defines the agent's identity and general behavior. The
`skills/query-writing/` and `skills/schema-exploration/` directories provide
task-specific workflows for forming safe SQL and exploring the database before
answering.

## Tools, subagents, and backends

The tool surface comes from LangChain's SQL toolkit, so the example does not
hand-write database tools. It uses `ChatAnthropic` with temperature zero both for
the toolkit setup and the main agent.

There are no custom subagents. The agent uses `FilesystemBackend(root_dir=base_dir)`
so it can persist scratch files and skill artifacts beside the example, while
the actual data access happens through the SQLite-backed SQL toolkit.

## Concept demonstrated

The example shows how Deep Agents can wrap a domain toolkit instead of custom
tools. The model gets the normal deep-agent planning and filesystem affordances,
but the database boundary remains mediated by structured SQL tools and skills.
