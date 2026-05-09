# `libs/deepagents/deepagents/profiles/harness/_anthropic_haiku_4_5.py`

> Built-in Claude Haiku 4.5 harness profile.

## Position in the system

The shared profile bootstrap calls `register()` once on first registry access.
This module registers an exact-model profile for
`"anthropic:claude-haiku-4-5"`.

The source docstring is an audit anchor: current Anthropic prompting guidance
does not add Haiku-4.5-specific prompt content beyond the universal Claude
snippets.

## Functions and classes

### `register() -> None`

Registers `HarnessProfile(system_prompt_suffix=_SYSTEM_PROMPT_SUFFIX)` under
the Haiku 4.5 model key via `_register_harness_profile_impl()`.

## System prompts and tool descriptions

`_SYSTEM_PROMPT_SUFFIX` is appended to the assembled base system prompt:

```text
<use_parallel_tool_calls>
If you intend to call multiple tools and there are no dependencies between the tool calls, make all of the independent tool calls in parallel. Prioritize calling tools simultaneously whenever the actions can be done in parallel rather than sequentially. For example, when reading 3 files, run 3 tool calls in parallel to read all 3 files into context at the same time. Maximize use of parallel tool calls where possible to increase speed and efficiency. However, if some tool calls depend on previous calls to inform dependent values like the parameters, do NOT call these tools in parallel and instead call them sequentially. Never use placeholders or guess missing parameters in tool calls.
</use_parallel_tool_calls>

<investigate_before_answering>
Never speculate about code you have not opened. If the user references a specific file, you MUST read the file before answering. Make sure to investigate and read relevant files BEFORE answering questions about the codebase. Never make any claims about code before investigating unless you are certain of the correct answer - give grounded and hallucination-free answers.
</investigate_before_answering>

<tool_result_reflection>
After receiving tool results, carefully reflect on their quality and determine optimal next steps before proceeding. Use your thinking to plan and iterate based on this new information, and then take the best next action.
</tool_result_reflection>
```

## Gotchas

The absence of Haiku-specific extra snippets is intentional and documented in
the source module, not an omission.

