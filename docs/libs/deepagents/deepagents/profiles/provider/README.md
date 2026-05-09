# `libs/deepagents/deepagents/profiles/provider/`

> Provider profiles customize chat-model construction before the agent harness is assembled.

## Position in the system

`deepagents._models.resolve_model()` uses provider profiles when it turns a
string model spec into a LangChain chat model. A provider profile can add static
`init_chat_model` kwargs, dynamic kwargs from environment state, and pre-init
checks such as dependency version enforcement.

## Module map

| Module | Doc | What to read it for |
|---|---|---|
| `__init__.py` | [`__init__.md`](./__init__.md) | Provider-profile re-exports. |
| `provider_profiles.py` | [`provider_profiles.md`](./provider_profiles.md) | Registry, merge rules, lookup, and application. |
| `_openai.py` | [`_openai.md`](./_openai.md) | Built-in OpenAI Responses API default. |
| `_openrouter.py` | [`_openrouter.md`](./_openrouter.md) | OpenRouter version check and attribution defaults. |

## Gotchas

Provider profiles do not alter system prompts, tools, middleware, or subagents.
Those belong to harness profiles.

