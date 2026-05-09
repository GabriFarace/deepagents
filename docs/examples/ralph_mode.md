# `examples/ralph_mode/`

> An autonomous looping wrapper around the Deep Agents CLI.

## Purpose

Ralph mode repeats a task across fresh agent invocations. Each iteration starts
with new model context, but the filesystem and git working tree carry progress
forward.

## Entry files

`ralph_mode.py` is the whole implementation. Its `ralph()` coroutine builds an
iteration prompt and calls `deepagents_cli.non_interactive.run_non_interactive`.
`main()` exposes CLI flags for iteration count, work directory, model, sandbox,
streaming, and model parameters.

`ralph_mode_diagram.png` illustrates the loop, and `README.md` explains the
origin of the pattern.

## Tools, subagents, and backends

This example does not construct SDK tools or subagents. It delegates tool
registration, model resolution, checkpointing, streaming, approvals, and sandbox
selection to the CLI non-interactive runner.

The backend is determined by CLI flags. Local mode uses the current working
directory; sandbox mode can route code operations into providers such as Modal
or other CLI-supported sandboxes.

## Concept demonstrated

Ralph mode is an outer control loop. It shows that persistence does not always
need to be conversation memory: a clean context plus a durable filesystem can be
enough for incremental autonomous work.
