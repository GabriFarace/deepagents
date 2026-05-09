# `examples/content-builder-agent/`

> A local content-writing agent configured mostly through files on disk.

## Purpose

This example shows how to make an agent feel like a small product workspace
rather than a single Python script. Brand voice lives in `AGENTS.md`, reusable
workflows live under `skills/`, and delegated research behavior lives in
`subagents.yaml`.

## Entry files

`content_writer.py` is the runnable CLI. It defines the custom tools, loads the
YAML subagent specs, creates the agent, invokes it, and renders progress with
Rich.

`AGENTS.md` supplies persistent content and style instructions through memory.
`skills/blog-post/SKILL.md` and `skills/social-media/SKILL.md` teach concrete
writing workflows. `subagents.yaml` externalizes the researcher subagent spec
that `content_writer.py` converts into the SDK's expected Python dictionaries.

## Tools, subagents, and backends

The script defines `web_search` for the researcher subagent and two Gemini image
tools: `generate_cover` and `generate_social_image`. The main agent receives the
image tools directly, while the `researcher` subagent receives `web_search`.

The agent uses `FilesystemBackend(root_dir=EXAMPLE_DIR)`, so generated drafts,
research notes, and images are written into the example directory. It also passes
`memory=["./AGENTS.md"]` and `skills=["./skills/"]`, letting the built-in memory
and skills middleware load instructions from disk.

## Concept demonstrated

This is the clearest example of file-native agent configuration. Deep Agents
loads memory and skills directly, while the example adds a small
`load_subagents()` helper to get the same ergonomic feel for subagents. It is a
useful pattern when non-engineers should be able to edit voice, workflow, and
delegation behavior without changing most of the Python code.
