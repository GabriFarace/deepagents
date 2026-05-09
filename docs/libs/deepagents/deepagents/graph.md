# `libs/deepagents/deepagents/graph.py`

> Primary SDK factory. This file turns a model, tools, middleware, subagents,
> backend, memory, permissions, and profile settings into one configured
> LangGraph `CompiledStateGraph`.

## Position in the system

`create_deep_agent()` is the SDK entry point re-exported from `deepagents`.
It wraps LangChain's `create_agent()` rather than building a graph by hand.
The work in this file is therefore assembly: resolve the model, select the
harness profile, build subagent specs, order middleware, assemble the system
prompt, and pass the result to LangChain.

```
caller
  │
  ▼
create_deep_agent(...)
  ├─ resolve model and harness profile
  ├─ choose backend
  ├─ prepare inline and async subagents
  ├─ build middleware stacks
  ├─ assemble system prompt
  └─ create_agent(...).with_config(...)
       ▼
     CompiledStateGraph
```

## Imports and module-level state

The imports reveal the assembly boundary. LangChain supplies the agent factory,
middleware base types, `TodoListMiddleware`, and `HumanInTheLoopMiddleware`.
LangGraph supplies persistence, store, cache, and compiled-graph types.
Deepagents supplies backend abstractions, custom middleware, profile lookup,
and the deprecation helpers.

Module-level state consists of a logger, the default base prompt, and three
required-middleware registries. The registries protect
`FilesystemMiddleware` and `SubAgentMiddleware` from profile-level exclusion,
because removing them would silently break core filesystem permissions and the
`task` tool.

## Functions and classes

### `BASE_AGENT_PROMPT`

`BASE_AGENT_PROMPT` is the default behavioral contract for every deep agent.
The final prompt is assembled as caller-provided `USER` instructions, then this
base prompt or a profile replacement, then any profile suffix. That ordering
puts user instructions first while still keeping model-specific tuning close to
the conversation history.

If the caller passes a `SystemMessage`, `create_deep_agent()` appends the base
prompt as a new text content block instead of flattening the message. That
preserves explicit provider metadata such as Anthropic `cache_control` markers.

### `_build_default_model()`

This private helper constructs the deprecated default model,
`ChatAnthropic(model_name="claude-sonnet-4-6")`. It exists so
`create_deep_agent(model=None)` can warn once about the parameter-level
deprecation without also triggering `get_default_model()`'s function-level
deprecation warning.

The function reads no environment variables itself, but the returned Anthropic
model needs normal provider credentials when invoked.

### `get_default_model()`

`get_default_model()` returns the same default Anthropic model but is decorated
as deprecated since `0.5.3` and scheduled for removal in `1.0.0`. It is kept
for callers who used the old public fallback helper directly.

Internally, it delegates to `_build_default_model()`. New code should construct
a model explicitly and pass it to `create_deep_agent()`.

### `_REQUIRED_MIDDLEWARE`, `_REQUIRED_MIDDLEWARE_CLASSES`, `_REQUIRED_MIDDLEWARE_NAMES`

These constants define middleware that profile exclusion is not allowed to
remove. `FilesystemMiddleware` owns the SDK's file tools and enforces
filesystem permission rules. `SubAgentMiddleware` owns the synchronous
`task` tool. If a harness profile could remove either by accident, the agent
would still compile but behave incorrectly.

The tuple form stores class objects plus any public name aliases. The frozenset
forms are derived lookup tables passed into the exclusion-validation helpers.

### `create_deep_agent(...)`

`create_deep_agent()` accepts a model spec or model object, caller tools,
optional system prompt, extra middleware, subagent specs, skill/memory paths,
filesystem permissions, a backend, human-in-the-loop config, and the standard
LangChain/LangGraph pass-throughs such as `response_format`, `context_schema`,
`checkpointer`, `store`, `debug`, `name`, and `cache`. It returns a compiled
LangGraph agent with deepagents metadata and a high recursion limit.

The first branch normalizes the model. `model=None` is still accepted but warns
and creates the deprecated Anthropic default. String models go through
`resolve_model()`, which applies provider profiles before calling LangChain's
`init_chat_model()`. The resolved model and original string spec feed harness
profile lookup, which can add prompt changes, tool description overrides, extra
middleware, excluded tools, excluded middleware, and general-purpose-subagent
settings.

The next phase prepares tools and the backend. Tool description overrides are
applied by copying supported tool objects, not by mutating caller-owned values.
If no backend is supplied, `StateBackend()` is used. This backend then flows
into the main filesystem middleware and into each generated subagent middleware
stack, so one agent invocation sees one coherent virtual filesystem.

Subagent processing has three cases. `AsyncSubAgent` specs are recognized by
`graph_id` and routed to `AsyncSubAgentMiddleware`. `CompiledSubAgent` specs
are passed through as runnable-backed inline subagents. Declarative `SubAgent`
specs are expanded: their model is resolved, their own harness profile is
selected, permissions are inherited or overridden, a base middleware stack is
built, profile exclusions are validated and applied, top-level tools are
inherited unless the subagent declares its own, and the profile-adjusted system
prompt is stored back onto the processed spec.

After caller-supplied subagents are processed, the factory may insert the
default `general-purpose` subagent. It skips this only when the active profile
disables it or the caller already supplied a synchronous subagent with the same
name. This default subagent uses the main model, backend, tools, permissions,
skills, profile middleware, tool exclusions, and prompt-caching middleware.
Profile-specific general-purpose description and prompt overrides are applied
at this point.

The main middleware stack is then assembled in a fixed order:
`TodoListMiddleware`, optional `SkillsMiddleware`, `FilesystemMiddleware`,
optional `SubAgentMiddleware`, optional `AsyncSubAgentMiddleware`,
summarization, patch-tool-call repair, user middleware, harness extra
middleware, optional tool exclusion, Anthropic prompt caching, optional memory,
and optional human-in-the-loop interrupts. Profile middleware exclusion is
applied after assembly, and coverage is verified across both the main stack and
the generated general-purpose subagent stack so stale exclusion entries fail
loudly.

Finally, the system prompt is assembled. `BASE_AGENT_PROMPT` is first modified
by the harness profile. If the caller passed no prompt, that becomes the full
system prompt. If the caller passed a string, it is prepended with a blank-line
separator. If the caller passed a `SystemMessage`, existing content blocks are
preserved and the base prompt is appended as another text block. The function
then calls `create_agent()` with the resolved model, tools, middleware, prompt,
and pass-through options, and attaches metadata:
`ls_integration="deepagents"`, the SDK version, optional agent name, and
`recursion_limit=9999`.

## System prompts and tool descriptions

`BASE_AGENT_PROMPT` is quoted in full because it is part of the default agent
behavior:

```text
You are a deep agent, an AI assistant that helps users accomplish tasks using tools. You respond with text and tool calls. The user can see your responses and tool outputs in real time.

## Core Behavior

- Be concise and direct. Don't over-explain unless asked.
- NEVER add unnecessary preamble ("Sure!", "Great question!", "I'll now...").
- Don't say "I'll now do X" — just do it.
- If the request is underspecified, ask only the minimum followup needed to take the next useful action.
- If asked how to approach something, explain first, then act.

## Professional Objectivity

- Prioritize accuracy over validating the user's beliefs
- Disagree respectfully when the user is incorrect
- Avoid unnecessary superlatives, praise, or emotional validation

## Doing Tasks

When the user asks you to do something:

1. **Understand first** — read relevant files, check existing patterns. Quick but thorough — gather enough evidence to start, then iterate.
2. **Act** — implement the solution. Work quickly but accurately.
3. **Verify** — check your work against what was asked, not against your own output. Your first attempt is rarely correct — iterate.

Keep working until the task is fully complete. Don't stop partway and explain what you would do — just do it. Only yield back to the user when the task is done or you're genuinely blocked.

**When things go wrong:**
- If something fails repeatedly, stop and analyze *why* — don't keep retrying the same approach.
- If you're blocked, tell the user what's wrong and ask for guidance.

## Clarifying Requests

- Do not ask for details the user already supplied.
- Use reasonable defaults when the request clearly implies them.
- Prioritize missing semantics like content, delivery, detail level, or alert criteria.
- Avoid opening with a long explanation of tool, scheduling, or integration limitations when a concise blocking followup question would move the task forward.
- Ask domain-defining questions before implementation questions.
- For monitoring or alerting requests, ask what signals, thresholds, or conditions should trigger an alert.

## Progress Updates

For longer tasks, provide brief progress updates at reasonable intervals — a concise sentence recapping what you've done and what's next.
```

This file does not itself define tool descriptions. It applies
harness-profile tool description overrides to caller tools and passes profile
task-description overrides into `SubAgentMiddleware`; the actual built-in tool
descriptions live in the corresponding middleware files.

## Flow walk-through

1. `graph.py:458` records the original string model spec for profile matching.
2. `graph.py:460` warns and builds the deprecated default model for
   `model=None`; otherwise `graph.py:478` resolves the caller's model.
3. `graph.py:479` selects the harness profile and `graph.py:481` validates
   profile-level middleware exclusions.
4. `graph.py:496` applies tool description overrides to top-level tools.
5. `graph.py:501` chooses `StateBackend()` when no backend is supplied.
6. `graph.py:509` separates async, compiled, and declarative subagent specs.
7. `graph.py:529` builds the declarative subagent base middleware stack.
8. `graph.py:559` applies subagent profile middleware exclusions and
   `graph.py:565` verifies they matched.
9. `graph.py:599` decides whether to add the default general-purpose subagent.
10. `graph.py:653` starts the main middleware stack and `graph.py:684` inserts
    user middleware before profile tail middleware.
11. `graph.py:706` applies main-profile middleware exclusions and
    `graph.py:716` verifies coverage across main and generated GP stacks.
12. `graph.py:724` assembles the final system prompt.
13. `graph.py:732` calls LangChain `create_agent()` and `graph.py:744`
    attaches recursion and LangSmith metadata.

## Gotchas

`permissions` are middleware-level, not backend-level. They protect built-in
filesystem tools but do not automatically constrain direct backend calls.

The default general-purpose subagent receives its own middleware stack. If a
profile excludes middleware, the exclusion can match either the main stack or
that generated subagent stack; coverage is verified only after both have been
assembled.

`AnthropicPromptCachingMiddleware` is added unconditionally with
`unsupported_model_behavior="ignore"`. Non-Anthropic models keep running; the
middleware simply has no effect.
