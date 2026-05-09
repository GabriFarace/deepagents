# `libs/deepagents/deepagents/profiles/harness/`

> Harness profiles customize runtime agent behavior after the chat model exists.

## Position in the system

`create_deep_agent()` resolves a harness profile for the model spec and applies
it while assembling prompts, tool descriptions, tool visibility, middleware
stacks, and the auto-added `general-purpose` subagent.

## Module map

| Module | Doc | What to read it for |
|---|---|---|
| `__init__.py` | [`__init__.md`](./__init__.md) | Harness-profile re-exports. |
| `harness_profiles.py` | [`harness_profiles.md`](./harness_profiles.md) | Runtime/config dataclasses, registry, merging, and lookup. |
| `_anthropic_opus_4_7.py` | [`_anthropic_opus_4_7.md`](./_anthropic_opus_4_7.md) | Claude Opus 4.7 prompt suffix. |
| `_anthropic_sonnet_4_6.py` | [`_anthropic_sonnet_4_6.md`](./_anthropic_sonnet_4_6.md) | Claude Sonnet 4.6 prompt suffix. |
| `_anthropic_haiku_4_5.py` | [`_anthropic_haiku_4_5.md`](./_anthropic_haiku_4_5.md) | Claude Haiku 4.5 prompt suffix. |
| `_openai_codex.py` | [`_openai_codex.md`](./_openai_codex.md) | Codex prompt suffix and per-model registrations. |

## Gotchas

Harness profiles can hide tools or middleware, but required scaffolding such as
filesystem and subagent middleware is protected. Removing the `task` tool is
done by disabling the default general-purpose subagent and providing no
synchronous subagents, not by excluding `SubAgentMiddleware`.

