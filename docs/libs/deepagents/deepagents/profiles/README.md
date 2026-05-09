# `libs/deepagents/deepagents/profiles/`

> Beta profile system for model-construction defaults and runtime harness behavior.

## Position in the system

Profiles sit between user-facing model specs and the final agent graph:

```text
create_deep_agent()
  -> resolve_model()              -> ProviderProfile
  -> middleware/prompt assembly   -> HarnessProfile
```

Provider profiles tune how `init_chat_model` constructs a chat model. Harness
profiles tune how `create_deep_agent` runs that model: prompt suffixes, tool
description overrides, tool exclusion, middleware exclusion, extra middleware,
and the default general-purpose subagent.

Built-ins are loaded lazily by `_builtin_profiles.py` the first time either
registry is accessed. This keeps import cheap and gives third-party plugins a
single controlled bootstrap point.

## Module map

| Module | Doc | What to read it for |
|---|---|---|
| `__init__.py` | [`__init__.md`](./__init__.md) | Public beta re-exports. |
| `_builtin_profiles.py` | [`_builtin_profiles.md`](./_builtin_profiles.md) | Lazy built-in and entry-point bootstrap. |
| `_keys.py` | [`_keys.md`](./_keys.md) | Shared validation for profile registry keys. |
| `provider/` | [`provider/`](./provider/README.md) | Model-construction profiles. |
| `harness/` | [`harness/`](./harness/README.md) | Runtime behavior profiles and built-in prompt suffixes. |

## Gotchas

The profile APIs are explicitly beta. They are public enough to export from
`deepagents`, but the docs should preserve the distinction between provider
profiles, which affect construction of a chat model, and harness profiles,
which affect the agent stack after the model exists.

