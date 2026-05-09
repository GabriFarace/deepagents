# `libs/deepagents/deepagents/middleware/`

> Agent behavior lives here. Each module is a LangChain `AgentMiddleware`
> implementation or helper used by the default deepagents stack.

## Position in the system

`graph.py:create_deep_agent()` assembles middleware in a deliberate order:

```
TodoListMiddleware
SkillsMiddleware?              (if skills are configured)
FilesystemMiddleware
SubAgentMiddleware?            (if synchronous subagents exist)
AsyncSubAgentMiddleware?       (if remote/background subagents exist)
SummarizationMiddleware
PatchToolCallsMiddleware
user middleware
profile extra middleware
tool exclusion?
AnthropicPromptCachingMiddleware
MemoryMiddleware?              (if memory files are configured)
HumanInTheLoopMiddleware?      (if interrupt_on is configured)
```

Middleware is the main extension mechanism. The SDK rarely special-cases
behavior in the graph itself; it registers hooks and tools here.

## Files

| Source | Doc | Role |
|---|---|---|
| `__init__.py` | [`__init__.md`](./__init__.md) | Public middleware re-export surface. |
| `subagents.py` | [`subagents.md`](./subagents.md) | Synchronous `task` tool and inline sub-agent invocation. |
| `filesystem.py` | [`filesystem.md`](./filesystem.md) | File/search/write/edit/execute tools plus large-result eviction. |
| `async_subagents.py` | `async_subagents.md` | Remote/background sub-agent tools. |
| `memory.py` | `memory.md` | Loading memory files into system prompts. |
| `skills.py` | `skills.md` | Skill catalog loading and prompt/tool exposure. |
| `summarization.py` | `summarization.md` | Context compaction middleware factory. |
| `patch_tool_calls.py` | `patch_tool_calls.md` | Repair for dangling tool calls after interruption. |
| `permissions.py` | `permissions.md` | Backward-compatible `FilesystemPermission` re-export. |
| `_tool_exclusion.py` | [`_tool_exclusion.md`](./_tool_exclusion.md) | Late model-request tool filtering for profile exclusions. |
| `_utils.py` | [`_utils.md`](./_utils.md) | Shared system-message append helper. |

## Gotchas

Ordering is behavior. For example, filesystem tools must exist before
subagents inherit tool access, and HITL should be late enough to wrap the final
tool set.
