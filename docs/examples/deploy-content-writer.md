# `examples/deploy-content-writer/`

> A deployed content writer with authenticated, per-user memory.

## Purpose

This example turns the local content-writing idea into a multi-user deployment.
It writes blog posts, LinkedIn posts, and tweets while remembering each user's
preferences independently.

## Entry files

`deepagents.toml` is the deployment entrypoint. It configures the model and the
Supabase auth provider. `AGENTS.md` describes the content workflow and memory
policy. `user/AGENTS.md` provides user-scoped context loaded through the memory
system.

The `skills/blog-post/` and `skills/social-media/` directories define reusable
writing workflows. `test_user_memory.py` demonstrates the expected behavior of
authenticated memory isolation.

## Tools, subagents, and backends

The main runtime feature is deploy-managed memory under `/memories/user/`.
Authentication scopes those files to the caller's identity, so preferences and
context do not bleed across users.

The example does not define Python tools or explicit subagents. Its behavior is
driven by deploy config, memory files, and skills. Supabase supplies token
validation through the `[auth]` section in `deepagents.toml`.

## Concept demonstrated

This is the per-user memory pattern. The important lesson is that identity,
memory isolation, and persistent personalization can be configured without
custom middleware when the deployment runtime owns authentication.
