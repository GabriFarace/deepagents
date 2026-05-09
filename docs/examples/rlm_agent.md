# `examples/rlm_agent/`

> A recursive Deep Agent helper that replaces `general-purpose` with compiled deeper agents.

## Purpose

This example explores recursive language-model agents. A top-level agent can
delegate to `general-purpose`, but that name points to another compiled Deep
Agent with its own REPL and its own deeper `general-purpose`, until the
configured depth bottoms out.

## Entry files

`rlm_agent.py` contains the helper and demo. `create_rlm_agent()` validates the
requested depth and subagent names, then calls the private `_build()` recursive
factory. Running the file directly starts a toy arithmetic demo.

## Tools, subagents, and backends

Every level receives the user-provided tools and a `REPLMiddleware` configured
for parallel tool calling over those tool names plus `task`. For depths above
zero, `_build()` creates a deeper graph first and registers it as a
`CompiledSubAgent` named `general-purpose`.

The helper intentionally owns the `general-purpose` subagent name. Extra
subagents are allowed, but a caller may not pass another spec with that name.
No custom backend is required by the pattern.

## Concept demonstrated

The example shows that subagents can be compiled graphs, not only prompt/tool
specifications. It also demonstrates how the QuickJS REPL and PTC can combine
with recursive delegation to fan out work at multiple depths.
