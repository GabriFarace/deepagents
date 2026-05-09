# `examples/repl_swarm/`

> A QuickJS REPL skill that dispatches many subagent tasks with bounded concurrency.

## Purpose

This example packages swarm behavior as a skill module rather than Python agent
logic. The model imports a TypeScript helper from the REPL and uses it to call
`tools.task(...)` many times in parallel.

## Entry files

`swarm_agent.py` builds and runs the agent. The swarm implementation lives in
`skills/swarm/index.ts`, with `skills/swarm/SKILL.md` teaching the model when to
import and call it.

## Tools, subagents, and backends

The agent uses `REPLMiddleware(ptc=["task"], skills_backend=backend)` so REPL
code can call the deep-agent `task` tool. It uses a `CompositeBackend`: `/skills/`
is served from a virtual `FilesystemBackend` rooted at the checked-in skills
directory, while all other paths go to `StateBackend`.

There is no custom Python subagent spec. The example relies on the default
general-purpose subagent and uses the TypeScript `runSwarm()` helper to fan out
calls to it with bounded concurrency.

## Concept demonstrated

The lesson is that skills can ship executable code for the agent's REPL, not
just instructions. This keeps the Python driver generic while putting reusable
coordination logic in a skill package.
