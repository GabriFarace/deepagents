# `libs/deepagents/deepagents/middleware/patch_tool_calls.py`

> Dangling tool-call repair middleware. It inserts synthetic `ToolMessage`
> responses for interrupted tool calls before the next agent run starts.

## Position in the system

Some providers require every `AIMessage.tool_calls[*].id` in the message
history to have a matching `ToolMessage`. If a user interrupts or sends a new
message while tool calls are pending, the next model request can otherwise
fail provider validation. This middleware repairs the history before the agent
loop resumes.

```
state["messages"]
  ├─ AIMessage(tool_calls=[...])
  ├─ maybe missing ToolMessage(...)
  └─ PatchToolCallsMiddleware.before_agent()
       └─ Overwrite([... synthetic cancellation ToolMessages ...])
```

## Imports and module-level state

The module imports `AIMessage` and `ToolMessage` from LangChain Core,
`Runtime` from LangGraph, and `Overwrite` so the whole messages list can be
replaced atomically. It defines no constants.

## Functions and classes

### `PatchToolCallsMiddleware`

An `AgentMiddleware` with a single `before_agent()` hook. It does not expose
tools or modify model requests; it only normalizes message state before the
agent starts.

#### `before_agent(state, runtime)`

Reads `state["messages"]`. If there are no messages, or if every tool call ID
already has an answered `ToolMessage`, it returns `None` and leaves state
unchanged.

When it finds unanswered tool calls, it walks the message list in order and
copies each original message. Immediately after each `AIMessage` with dangling
tool calls, it inserts one synthetic `ToolMessage` per missing ID. The content
states that the tool call was cancelled because another message arrived before
completion. The method returns `{"messages": Overwrite(patched_messages)}` so
LangGraph replaces the message channel rather than appending duplicates.

## Gotchas

This is a provider-compatibility patch, not a retry system. Synthetic tool
messages mark the old calls as cancelled; they do not execute or requeue those
tools.
