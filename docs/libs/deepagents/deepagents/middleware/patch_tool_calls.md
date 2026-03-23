# `deepagents/middleware/patch_tool_calls.py`

## High-Level Purpose

`PatchToolCallsMiddleware` resolves a class of errors where an `AIMessage` contains tool calls that have no corresponding `ToolMessage` response in the conversation history (dangling tool calls). This situation can occur when a new user message arrives before a tool call completes, causing an incomplete history that most LLMs reject.

The middleware runs before each agent step and injects synthetic `ToolMessage` responses for any unmatched tool calls.

## Class: `PatchToolCallsMiddleware(AgentMiddleware)`

### State Schema
Uses the base `AgentState` (no additional state fields).

### Methods

#### `before_agent(state, runtime) -> dict | None`

Scans the message history for dangling tool calls.

**Logic:**
1. Iterates through `state["messages"]`.
2. For each `AIMessage` with `tool_calls`:
   - For each `tool_call` in the message:
     - Searches forward in the message list for a `ToolMessage` with a matching `tool_call_id`.
     - If no match is found, creates a synthetic `ToolMessage` with content: `"Tool call {tool_call['name']} with id {tool_call['id']} was cancelled - another message came in before it could be completed."`
3. Returns `{"messages": Overwrite(patched_messages)}` if any patches were made, or `None` otherwise.

The `Overwrite` wrapper tells LangGraph to replace the `messages` list entirely rather than appending.

## Dependencies

- `langchain.agents.middleware.AgentMiddleware`, `AgentState`
- `langchain_core.messages.AIMessage`, `ToolMessage`
- `langgraph.runtime.Runtime`
- `langgraph.types.Overwrite`

## Example Scenario

```
Turn 1: User asks question
Turn 2: AIMessage [tool_call: read_file(id=abc)]
Turn 3: User sends new message (interrupting before tool result arrived)
```

Without patching: The LLM sees an AIMessage with a tool_call followed by a HumanMessage — an invalid conversation structure.

With patching: A synthetic ToolMessage is inserted between turns 2 and 3 explaining the tool call was cancelled, resulting in a valid conversation structure.
