# `tests/chat_model.py`

## High-Level Purpose

A reusable fake LangChain chat model for use in unit and integration tests throughout the `deepagents_acp` test suite. Allows tests to deterministically control what the LLM returns and how it streams content, without making real API calls.

## Classes

### `GenericFakeChatModel`

**Purpose:** A controllable fake implementation of `BaseChatModel` that replays pre-configured messages, supports streaming with configurable delimiters, and handles tool calls.

**Inherits from:** `langchain_core.language_models.chat_models.BaseChatModel`

#### Attributes

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `messages` | `Iterator[AIMessage \| str]` | required | Iterator of messages to return. Use `iter([...])` to pass a list. |
| `stream_delimiter` | `str \| None` | `None` | Controls streaming chunk behavior. `None` = single chunk; a string or regex pattern = split content on that delimiter (delimiters are preserved as separate chunks) |

#### Methods

##### `_generate(messages, stop, run_manager, **kwargs) -> ChatResult`
Advances the iterator by one message and returns it as a `ChatResult`. Converts plain strings to `AIMessage` objects.

##### `_stream(messages, stop, run_manager, **kwargs) -> Iterator[ChatGenerationChunk]`
Calls `_generate` internally and then breaks the result into streaming chunks:
- If `stream_delimiter` is `None`: emits the full content as a single `ChatGenerationChunk`.
- If `stream_delimiter` is set: uses `re.split()` to break content into pieces, yielding one chunk per piece (empty strings filtered out).
- Tool calls are attached only to the last content chunk.
- If there is no content but there are tool calls, emits a single chunk with the tool calls.
- Handles `additional_kwargs` (e.g., `function_call`) by streaming each key/value as separate chunks, breaking `function_call` strings on commas.

##### `_llm_type -> str`
Returns `"generic-fake-chat-model"`.

##### `bind_tools(tools, *, tool_choice, **kwargs) -> Runnable`
No-op override that returns `self`, allowing tests to use the model with agents that call `bind_tools`.

#### Usage Examples

```python
# Return "Hello!" as a single non-streaming chunk
model = GenericFakeChatModel(messages=iter([AIMessage(content="Hello!")]))

# Stream "Hello world" split on whitespace
model = GenericFakeChatModel(
    messages=iter([AIMessage(content="Hello world")]),
    stream_delimiter=r"(\s)"
)
# Yields chunks: "Hello", " ", "world"

# Return an AI message with a tool call
model = GenericFakeChatModel(
    messages=iter([AIMessage(
        content="",
        tool_calls=[{"name": "my_tool", "args": {}, "id": "1", "type": "tool_call"}]
    )]),
    stream_delimiter=None,
)
```

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| `BaseChatModel` | `langchain_core.language_models.chat_models` | Base class for chat models |
| `AIMessage`, `AIMessageChunk`, `BaseMessage` | `langchain_core.messages` | Message types |
| `ChatGeneration`, `ChatGenerationChunk`, `ChatResult` | `langchain_core.outputs` | Output types |
| `re` | stdlib | Regex-based content splitting |
