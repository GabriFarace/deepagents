# `deepagents/middleware/subagents.py`

## High-Level Purpose

`SubAgentMiddleware` provides the `task` tool that allows the main agent to delegate complex, multi-step sub-tasks to ephemeral sub-agents. Sub-agents have isolated context windows, execute independently, and return a single final message to the main agent.

This module defines two TypedDicts for specifying subagents declaratively (`SubAgent`) or with a pre-compiled runnable (`CompiledSubAgent`), and the `SubAgentMiddleware` class that builds and invokes them.

## TypedDicts

### `SubAgent`

A declarative specification for a synchronous sub-agent.

| Field | Required | Type | Description |
|---|---|---|---|
| `name` | Yes | `str` | Unique identifier; used by the main agent in `task` tool calls |
| `description` | Yes | `str` | What this subagent does; main agent uses this to decide when to delegate |
| `system_prompt` | Yes | `str` | Instructions for the sub-agent |
| `tools` | No | `Sequence[...]` | Tools for this agent; inherits main agent's tools if omitted |
| `model` | No | `str \| BaseChatModel` | Override the main agent's model |
| `middleware` | No | `list[AgentMiddleware]` | Additional middleware |
| `interrupt_on` | No | `dict[str, bool \| InterruptOnConfig]` | HITL tool interrupts |
| `skills` | No | `list[str]` | Skill source paths for `SkillsMiddleware` |

When using `create_deep_agent`, `SubAgent` entries automatically receive the default middleware stack prepended (TodoList, Filesystem, Summarization, PatchToolCalls, optional Skills, AnthropicPromptCaching).

### `CompiledSubAgent`

A pre-compiled agent spec. Use when you have a custom LangGraph graph or a `create_agent()` runnable.

| Field | Required | Type | Description |
|---|---|---|---|
| `name` | Yes | `str` | Unique identifier |
| `description` | Yes | `str` | What this subagent does |
| `runnable` | Yes | `Runnable` | The compiled agent graph |

The runnable's state schema must include a `messages` key. The final message is extracted and returned as a `ToolMessage` to the parent agent.

## Constants

### `DEFAULT_SUBAGENT_PROMPT`
Default system prompt addition for sub-agents: `"In order to complete the objective that the user asks of you, you have access to a number of standard tools."`

### `GENERAL_PURPOSE_SUBAGENT` (imported from this module in `graph.py`)
The spec for the auto-added general-purpose subagent. Has `name="general-purpose"`.

### `TASK_TOOL_DESCRIPTION`
Large agent-facing description of the `task` tool. Includes:
- Available agent types with descriptions
- 7 usage rules (parallel launch, context isolation, detailed task descriptions)
- Multiple worked examples with commentary explaining when/why to use subagents

### `_EXCLUDED_STATE_KEYS`
`{"messages", "todos", "structured_response", "skills_metadata", "memory_contents"}` — State keys filtered out when passing state to sub-agents and when returning updates. Prevents parent state from leaking to child agents and avoids conflicts with non-reduceable keys.

### `_subagent_tracing_context() -> Generator`

A context manager that tags subagent runs with `ls_agent_type="subagent"` in the LangSmith tracing metadata. This mirrors LangChain's `ls_agent_type="root"` tagging behavior on the main agent, enabling trace display features in LangSmith to distinguish root from sub-agent runs. All other current tracing-context fields (parent, client, tags, etc.) are forwarded unchanged so the enclosing context is not clobbered. Both sync (`task`) and async (`atask`) invocations are wrapped with this context manager.

## Class: `SubAgentMiddleware(AgentMiddleware)`

Wraps the LLM call to provide the `task` tool. Does not inject system prompt content; the task tool description conveys the available subagents.

### Constructor

```python
SubAgentMiddleware(
    backend: BackendProtocol | BackendFactory,
    subagents: list[SubAgent | CompiledSubAgent],
)
```

### Tool: `task`

The main tool exposed by this middleware. Takes:
- `description` — Detailed task description for the sub-agent.
- `subagent_type` — Name of the sub-agent to use.

**Execution flow:**
1. Looks up the subagent spec by `subagent_type`.
2. For `CompiledSubAgent`: invokes `spec["runnable"]` directly.
3. For `SubAgent`: calls `create_agent(model, system_prompt, tools, middleware)` to build the agent, then invokes it.
4. Passes shared state (excluding `_EXCLUDED_STATE_KEYS`) to the sub-agent so it shares the parent's filesystem and todos.
5. Extracts the final message from the sub-agent's `messages` list.
6. Returns a `Command(update={"messages": [ToolMessage(result)]})` to merge the sub-agent's state updates (files, todos, etc.) back into the parent agent's state.

Both sync (`task`) and async (`atask`) versions are provided.

## Dependencies

- `langchain.agents.create_agent`
- `langchain.agents.middleware.HumanInTheLoopMiddleware`, `InterruptOnConfig`
- `langchain_core.messages`, `langchain_core.tools`
- `langgraph.types.Command`
- `langsmith.run_helpers.get_tracing_context`, `tracing_context` — for `ls_agent_type` tagging
- `deepagents.backends.protocol` — `BackendFactory`, `BackendProtocol`
- `deepagents.middleware._utils.append_to_system_message`
