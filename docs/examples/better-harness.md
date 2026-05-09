# `examples/better-harness/`

> A research artifact for using one Deep Agent to improve another agent harness through evals.

## Purpose

`better-harness` implements an outer-loop optimization workflow. The user
defines editable harness surfaces and train/holdout eval cases; an outer Deep
Agent proposes changes; the runner keeps only candidates that improve eval
performance.

## Entry files

`better_harness/core.py` contains the experiment model, config loading,
evaluation loop, reporting, trace collection, and CLI. `better_harness/agent.py`
builds the proposer workspace and invokes the outer Deep Agent. `patching.py`
and `runners.py` support applying surface changes and executing eval commands.

`better_harness_plugin.py` exposes the package as a plugin-style entrypoint.
`examples/deepagents_example.toml` is the worked configuration to start from,
and `tests/test_better_harness.py` captures the expected loop behavior.

## Tools, subagents, and backends

The outer agent edits a materialized `/current` workspace containing only the
allowed harness surfaces. It also sees visible train failures, copied case
files, history, and task instructions. The target agent is the "inner" agent
under evaluation.

The example can point the outer agent at a local Deep Agents checkout through
configuration. Its backend and tool surface are those of the Deep Agent invoked
by the proposer, while eval execution is handled by configured shell commands.

## Concept demonstrated

The example shows Deep Agents as a meta-agent for harness engineering. It is not
just solving user tasks; it is editing prompts, tools, skills, middleware, and
wiring surfaces, then letting evals decide whether to keep the change.
