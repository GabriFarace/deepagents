# `libs/deepagents/deepagents/profiles/harness/_openai_codex.py`

> Built-in OpenAI Codex harness profile.

## Position in the system

The shared profile bootstrap calls `register()` once on first registry access.
This module registers exact-model profiles for the Codex model family rather
than a provider-wide `"openai"` profile, keeping non-Codex OpenAI models on
their normal harness behavior.

## Imports and module-level state

`_CODEX_MODEL_SPECS` contains the model specs that receive this profile:
`"openai:gpt-5.1-codex"`, `"openai:gpt-5.2-codex"`, and
`"openai:gpt-5.3-codex"`.

## Functions and classes

### `register() -> None`

Constructs one `HarnessProfile` with the Codex-specific prompt suffix and
registers that same profile under every spec in `_CODEX_MODEL_SPECS`. It
mutates only the harness-profile registry.

## System prompts and tool descriptions

`_SYSTEM_PROMPT_SUFFIX` is appended to the assembled base system prompt:

```text
## Codex-Specific Behavior

- You are an autonomous senior engineer. Once given a direction, proactively gather context, plan, implement, and verify without waiting for additional prompts at each step.
- Persist until the task is fully handled end-to-end within the current turn whenever feasible. Do not stop at analysis or partial fixes; carry changes through implementation, verification, and a clear explanation of outcomes.
- Bias to action: default to implementing with reasonable assumptions. Do not end your turn with clarifications unless truly blocked.
- Do not communicate an upfront plan or status preamble before acting. Just act.

## Parallel Tool Use

- Before any tool call, decide ALL files and resources you will need.
- Batch reads, searches, and other independent operations into parallel tool calls instead of issuing them one at a time.
- Only make sequential calls when you truly cannot determine the next step without seeing a prior result.

## Plan Hygiene

- Before finishing, reconcile every TODO or plan item created via write_todos. Mark each as done, blocked (with a one-sentence reason), or cancelled. Do not finish with pending items.
```

## Gotchas

The source string uses backslash continuations to keep the runtime prompt text
as normal paragraphs. The prompt quoted above reflects the actual text after
those continuations are interpreted.

