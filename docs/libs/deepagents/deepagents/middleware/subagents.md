# `libs/deepagents/deepagents/middleware/subagents.py`

> Synchronous sub-agent middleware. It adds the `task` tool, turns sub-agent
> specs into runnable graphs, isolates each invocation's state, and returns one
> `ToolMessage` back to the parent agent.

## Position in the system

`create_deep_agent()` prepares declarative and compiled sub-agent specs, then
installs `SubAgentMiddleware` only when there is at least one synchronous
subagent. Async/background subagents are handled by a different module.

```
main model
  │ emits task(description, subagent_type)
  ▼
SubAgentMiddleware task tool
  ├─ builds child state from parent runtime.state
  ├─ invokes selected child Runnable
  └─ returns Command(update={"messages": [ToolMessage(...)]})
```

The parent sees only the returned tool message and any allowed state updates.
The child agent's intermediate model/tool transcript stays isolated.

## Imports and module-level state

The module imports LangChain's `create_agent()` for declarative subagents,
`StructuredTool` for the task tool, and `Command` so a tool call can update
LangGraph state directly. `dataclasses` and `json` are used to serialize
structured subagent outputs.

Module-level constants define the default subagent prompt, state keys that
must not cross the parent/child boundary, the task tool schema, the default
task tool description, the system prompt addendum, and the built-in
`general-purpose` subagent spec.

## Functions and classes

### `SubAgent`

`SubAgent` is the declarative spec for a child agent. `name`, `description`,
and `system_prompt` are required; `tools`, `model`, `middleware`,
`interrupt_on`, `skills`, `permissions`, and `response_format` are optional at
the type level.

When users pass `SubAgent` specs through `create_deep_agent()`, `graph.py`
fills in missing defaults before this middleware sees them: model resolution,
inherited tools, filesystem/summarization middleware, profile middleware, and
inherited or overridden permissions. When constructing `SubAgentMiddleware`
directly, callers must provide a fully specified `model` and `tools`.

### `CompiledSubAgent`

`CompiledSubAgent` wraps an existing `Runnable` or compiled graph. It requires
`name`, `description`, and `runnable`. The runnable state schema must include
`messages`, because `_build_task_tool()` extracts either
`structured_response` or the final message to produce the parent-facing
`ToolMessage`.

Compiled subagents do not receive automatic middleware or inherited
`interrupt_on` inside this file. They are assumed to be fully configured before
registration.

### `_EXCLUDED_STATE_KEYS`

This set defines state channels that are stripped before parent state is passed
to a subagent and stripped again before child state updates are returned to the
parent. `messages` is handled specially, `todos` and `structured_response` do
not have obvious parent reducers, and skills/memory private state must not leak
from parent to child.

This is the isolation boundary that makes `task` useful as a context silo. A
subagent starts with a single `HumanMessage` containing the task description,
not the whole parent conversation.

### `TaskToolSchema`

Pydantic schema for the `task` tool. It requires a detailed `description` and
a `subagent_type`. The schema text tells the model to include enough context
for autonomous execution and to choose one of the available agent types listed
in the tool description.

### `_SubagentSpec`

Internal normalized representation used after declarative and compiled specs
have been converted into runnable objects. It stores only `name`,
`description`, and `runnable`, which is exactly what the task tool needs.

### `_build_task_tool(subagents, task_description=None)`

`_build_task_tool()` constructs the `StructuredTool` named `task`. It first
indexes subagent runnables by name and renders the available-agent list into
the default or custom tool description. A custom description can include an
`{available_agents}` placeholder; otherwise it is used as-is.

The nested `_validate_and_prepare_state()` function copies parent runtime
state except excluded keys, then replaces `messages` with one `HumanMessage`
containing the task description. The sync and async `task` implementations
validate that `subagent_type` exists and that LangChain supplied a
`tool_call_id`, invoke the selected runnable with `ls_agent_type="subagent"`
in config, and hand the result to `_return_command_with_state_update()`.

`_return_command_with_state_update()` requires a `messages` key in the child
result. It returns a LangGraph `Command` that updates parent state with any
allowed child state channels plus a single parent `ToolMessage`. If the child
produced `structured_response`, that object is JSON-serialized and used as the
tool content; otherwise the final child message text is used with trailing
whitespace stripped to avoid provider API errors.

### `SubAgentMiddleware`

`SubAgentMiddleware` is an `AgentMiddleware` whose `tools` list contains only
the generated `task` tool. Its constructor rejects an empty subagent list,
stores backend/specs, normalizes specs with `_get_subagents()`, and appends an
available-agent list to the task system prompt.

`_get_subagents()` has two paths. For compiled specs, it calls
`with_config()` to attach `lc_agent_name` metadata and `run_name` without
mutating the original runnable. For declarative specs, it validates that
`model` and `tools` are present, resolves the model, copies supplied
middleware, appends `HumanInTheLoopMiddleware` when `interrupt_on` exists, and
calls LangChain `create_agent()` with the subagent prompt, tools, middleware,
name, and optional response format.

`wrap_model_call()` and `awrap_model_call()` append the middleware's system
prompt to the request's system message before delegating to the next handler.
They do not alter the model response; their job is tool registration plus
instruction injection.

## System prompts and tool descriptions

`TASK_TOOL_DESCRIPTION`:

```text
Launch an ephemeral subagent to handle complex, multi-step independent tasks with isolated context windows.

Available agent types and the tools they have access to:
{available_agents}

When using the Task tool, you must specify a subagent_type parameter to select which agent type to use.

## Usage notes:
1. Launch multiple agents concurrently whenever possible, to maximize performance; to do that, use a single message with multiple tool uses
2. When the agent is done, it will return a single message back to you. The result returned by the agent is not visible to the user. To show the user the result, you should send a text message back to the user with a concise summary of the result.
3. Each agent invocation is stateless. You will not be able to send additional messages to the agent, nor will the agent be able to communicate with you outside of its final report. Therefore, your prompt should contain a highly detailed task description for the agent to perform autonomously and you should specify exactly what information the agent should return back to you in its final and only message to you.
4. The agent's outputs should generally be trusted
5. Clearly tell the agent whether you expect it to create content, perform analysis, or just do research (search, file reads, web fetches, etc.), since it is not aware of the user's intent
6. If the agent description mentions that it should be used proactively, then you should try your best to use it without the user having to ask for it first. Use your judgement.
7. When only the general-purpose agent is provided, you should use it for all tasks. It is great for isolating context and token usage, and completing specific, complex tasks, as it has all the same capabilities as the main agent.

### Example usage of the general-purpose agent:

<example_agent_descriptions>
"general-purpose": use this agent for general purpose tasks, it has access to all tools as the main agent.
</example_agent_descriptions>

<example>
User: "I want to conduct research on the accomplishments of Lebron James, Michael Jordan, and Kobe Bryant, and then compare them."
Assistant: *Uses the task tool in parallel to conduct isolated research on each of the three players*
Assistant: *Synthesizes the results of the three isolated research tasks and responds to the User*
<commentary>
Research is a complex, multi-step task in it of itself.
The research of each individual player is not dependent on the research of the other players.
The assistant uses the task tool to break down the complex objective into three isolated tasks.
Each research task only needs to worry about context and tokens about one player, then returns synthesized information about each player as the Tool Result.
This means each research task can dive deep and spend tokens and context deeply researching each player, but the final result is synthesized information, and saves us tokens in the long run when comparing the players to each other.
</commentary>
</example>

<example>
User: "Analyze a single large code repository for security vulnerabilities and generate a report."
Assistant: *Launches a single `task` subagent for the repository analysis*
Assistant: *Receives report and integrates results into final summary*
<commentary>
Subagent is used to isolate a large, context-heavy task, even though there is only one. This prevents the main thread from being overloaded with details.
If the user then asks followup questions, we have a concise report to reference instead of the entire history of analysis and tool calls, which is good and saves us time and money.
</commentary>
</example>

<example>
User: "Schedule two meetings for me and prepare agendas for each."
Assistant: *Calls the task tool in parallel to launch two `task` subagents (one per meeting) to prepare agendas*
Assistant: *Returns final schedules and agendas*
<commentary>
Tasks are simple individually, but subagents help silo agenda preparation.
Each subagent only needs to worry about the agenda for one meeting.
</commentary>
</example>

<example>
User: "I want to order a pizza from Dominos, order a burger from McDonald's, and order a salad from Subway."
Assistant: *Calls tools directly in parallel to order a pizza from Dominos, a burger from McDonald's, and a salad from Subway*
<commentary>
The assistant did not use the task tool because the objective is super simple and clear and only requires a few trivial tool calls.
It is better to just complete the task directly and NOT use the `task` tool.
</commentary>
</example>

### Example usage with custom agents:

<example_agent_descriptions>
"content-reviewer": use this agent after you are done creating significant content or documents
"greeting-responder": use this agent when to respond to user greetings with a friendly joke
"research-analyst": use this agent to conduct thorough research on complex topics
</example_agent_descriptions>

<example>
user: "Please write a function that checks if a number is prime"
assistant: Sure let me write a function that checks if a number is prime
assistant: First let me use the Write tool to write a function that checks if a number is prime
assistant: I'm going to use the Write tool to write the following code:
<code>
function isPrime(n) {{
  if (n <= 1) return false
  for (let i = 2; i * i <= n; i++) {{
    if (n % i === 0) return false
  }}
  return true
}}
</code>
<commentary>
Since significant content was created and the task was completed, now use the content-reviewer agent to review the work
</commentary>
assistant: Now let me use the content-reviewer agent to review the code
assistant: Uses the Task tool to launch with the content-reviewer agent
</example>

<example>
user: "Can you help me research the environmental impact of different renewable energy sources and create a comprehensive report?"
<commentary>
This is a complex research task that would benefit from using the research-analyst agent to conduct thorough analysis
</commentary>
assistant: I'll help you research the environmental impact of renewable energy sources. Let me use the research-analyst agent to conduct comprehensive research on this topic.
assistant: Uses the Task tool to launch with the research-analyst agent, providing detailed instructions about what research to conduct and what format the report should take
</example>

<example>
user: "Hello"
<commentary>
Since the user is greeting, use the greeting-responder agent to respond with a friendly joke
</commentary>
assistant: "I'm going to use the Task tool to launch with the greeting-responder agent"
</example>
```

`TASK_SYSTEM_PROMPT`:

```text
## `task` (subagent spawner)

You have access to a `task` tool to launch short-lived subagents that handle isolated tasks. These agents are ephemeral — they live only for the duration of the task and return a single result.

When to use the task tool:
- When a task is complex and multi-step, and can be fully delegated in isolation
- When a task is independent of other tasks and can run in parallel
- When a task requires focused reasoning or heavy token/context usage that would bloat the orchestrator thread
- When sandboxing improves reliability (e.g. code execution, structured searches, data formatting)
- When you only care about the output of the subagent, and not the intermediate steps (ex. performing a lot of research and then returned a synthesized report, performing a series of computations or lookups to achieve a concise, relevant answer.)

Subagent lifecycle:
1. **Spawn** → Provide clear role, instructions, and expected output
2. **Run** → The subagent completes the task autonomously
3. **Return** → The subagent provides a single structured result
4. **Reconcile** → Incorporate or synthesize the result into the main thread

When NOT to use the task tool:
- If you need to see the intermediate reasoning or steps after the subagent has completed (the task tool hides them)
- If the task is trivial (a few tool calls or simple lookup)
- If delegating does not reduce token usage, complexity, or context switching
- If splitting would add latency without benefit

## Important Task Tool Usage Notes to Remember
- Whenever possible, parallelize the work that you do. This is true for both tool_calls, and for tasks. Whenever you have independent steps to complete - make tool_calls, or kick off tasks (subagents) in parallel to accomplish them faster. This saves time for the user, which is incredibly important.
- Remember to use the `task` tool to silo independent tasks within a multi-part objective.
- You should use the `task` tool whenever you have a complex task that will take multiple steps, and is independent from other tasks that the agent needs to complete. These agents are highly competent and efficient.
```

## Flow walk-through

1. `graph.py` gives `SubAgentMiddleware` processed specs that already include
   model, tools, and middleware.
2. `subagents.py:532` rejects empty lists.
3. `subagents.py:537` normalizes specs into `_SubagentSpec` objects.
4. `subagents.py:539` builds the `task` tool.
5. `subagents.py:542` appends available agent names/descriptions to the
   system prompt.
6. At runtime, `subagents.py:610` appends that prompt to the model request.
7. When the model calls `task`, `subagents.py:433` validates the requested
   `subagent_type`.
8. `subagents.py:424` copies allowed parent state and installs a one-message
   child conversation.
9. `subagents.py:443` or `subagents.py:460` invokes the child runnable.
10. `subagents.py:413` returns a `Command` that inserts one `ToolMessage` into
    parent state.

## Gotchas

Subagent output is not shown to the user directly. The parent model sees the
tool result and must synthesize or relay it in a normal assistant message.

`SubAgentMiddleware` has a `backend` constructor argument for compatibility and
for caller symmetry, but this file does not use it directly. Backends are
installed on child agents through their middleware stacks.
