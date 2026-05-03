# `libs/deepagents/deepagents/graph.py`

## High-Level Purpose

`graph.py` is the public API of the `deepagents` SDK. It contains `create_deep_agent()`, the single factory function that assembles a complete LangGraph agent from its components: a model, a backend, middleware, tools, subagents, and an optional checkpointer. The function returns a `CompiledStateGraph` ready to invoke or stream.

---

## Key Function

### `create_deep_agent(**kwargs) → CompiledStateGraph`

All parameters are optional; calling `create_deep_agent()` with no arguments returns a working coding assistant using Claude Sonnet 4.6 with an in-memory backend.

**Parameters:**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `model` | `str \| BaseChatModel` | `"claude-sonnet-4-6"` | LLM to use |
| `backend` | `BackendProtocol` | `StateBackend()` | File/shell storage backend |
| `tools` | `list[BaseTool]` | `[]` | Additional tools for the agent |
| `subagents` | `list[SubAgent \| CompiledSubAgent]` | `[]` | Subagent specs for task delegation |
| `middleware` | `list[AgentMiddleware]` | `[]` | Additional middleware layers |
| `checkpointer` | `BaseCheckpointSaver \| None` | `None` | For session persistence |
| `system_prompt` | `str \| None` | `None` | Custom system prompt (replaces default) |
| `system_prompt_suffix` | `str \| None` | `None` | Appended after the base system prompt |
| `exclude_middleware` | `list[str]` | `[]` | Remove built-in middleware by class name or alias |
| `interrupt_on` | `list[str]` | `[]` | Tool names that trigger HITL approval |
| `permissions` | `list[FilesystemPermission]` | `[]` | File/path access rules |
| `memory_paths` | `list[str]` | `[]` | Paths to AGENTS.md memory files |
| `skills_paths` | `list[str \| tuple]` | `[]` | Paths to skills directories |
| `add_general_purpose_subagent` | `bool` | `True` | Auto-add a general-purpose subagent |
| `profile` | `str \| None` | `None` | Harness profile name (e.g., `"openai-responses"`) |

**Return value:** `CompiledStateGraph` — can be invoked with `graph.invoke()`, streamed with `graph.astream()`, or streamed with events via `graph.astream_events()`.

---

## Default Middleware Stack

Built in this order (bottom to top of the stack):

```
1. TodoListMiddleware          ← task planning (checklist in state)
2. SkillsMiddleware            ← skill catalog injection
3. FilesystemMiddleware        ← file + shell tools (required)
4. SubAgentMiddleware          ← "task" delegation tool (if subagents provided)
5. SummarizationMiddleware     ← context compaction
6. PatchToolCallsMiddleware    ← dangling tool call repair
7. AsyncSubAgentMiddleware     ← remote background tasks (if async subagents)
8. [user middleware here]
9. [profile extra_middleware]
10. _ToolExclusionMiddleware   ← removes excluded tools
11. AnthropicPromptCachingMiddleware ← adds cache_control marks
12. MemoryMiddleware           ← AGENTS.md injection (if memory_paths)
13. HumanInTheLoopMiddleware   ← interrupt gates (if interrupt_on)
```

The user's `middleware` parameter is inserted at position 8. This means user middleware runs after the core tool stack but before caching and HITL.

---

## System Prompt Assembly

System prompt is assembled in this order (each overwrites or appends the previous):

```
[user system_prompt]      ← if provided, replaces the base prompt
       +
[BASE_AGENT_PROMPT]       ← if user didn't provide system_prompt
       +
[system_prompt_suffix]    ← always appended (if provided)
```

`BASE_AGENT_PROMPT` (~95 lines) emphasizes: conciseness, objectivity, not preambling responses, confirming before destructive operations, and using tools rather than guessing.

---

## `BASE_AGENT_PROMPT` (constant)

The default system prompt loaded from `default_agent_prompt.md`. Key principles it establishes:
- Think step-by-step before acting
- Prefer tools over assumptions
- Report what was actually done, not what was planned
- Ask when uncertain rather than guessing
- Don't preamble or explain what you're about to do

---

## Architecture Notes

**Recursion limit:** The compiled graph uses `recursion_limit=9999` to allow long agentic loops. The `--max-turns` CLI flag and agent-level `max_turns` parameter provide softer caps.

**Subagent state exclusion:** When a subagent is invoked via the `task` tool, state keys like `todos`, `skills_metadata`, `memory_contents` are stripped before passing to the subagent. This prevents parent-level metadata from polluting subagent context.

**Profile system:** Profiles add or modify middleware for specific model families. The `"openai-responses"` profile, for example, sets OpenAI Responses API defaults. Profiles are loaded from `profiles/` and applied after user middleware.

**General-purpose subagent:** Unless `add_general_purpose_subagent=False`, a general-purpose subagent is always added. It has access to all parent tools and can handle any open-ended task. Providing a subagent named `"general-purpose"` in the `subagents` list replaces the auto-added one.

---

## Usage Example

```python
from deepagents import create_deep_agent
from deepagents.backends import FilesystemBackend
from deepagents.middleware import MemoryMiddleware

graph = create_deep_agent(
    model="anthropic:claude-sonnet-4-6",
    backend=FilesystemBackend(root_dir="/workspace"),
    memory_paths=["/workspace/AGENTS.md"],
    interrupt_on=["execute", "write_file"],
    system_prompt_suffix="\nAlways prefer Python over shell scripts.",
)

# Invoke
result = graph.invoke({
    "messages": [{"role": "user", "content": "List the Python files in src/"}]
})
print(result["messages"][-1].content)
```

---

## See Also

- [_models.md](_models.md) — model resolution
- [backends/README.md](backends/README.md) — choosing a backend
- [middleware/README.md](middleware/README.md) — middleware stack details
- [middleware/filesystem.md](middleware/filesystem.md) — file tools injected by FilesystemMiddleware
- [middleware/human_in_the_loop.md](middleware/human_in_the_loop.md) — `interrupt_on` behavior
