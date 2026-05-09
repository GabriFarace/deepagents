# `libs/cli/deepagents_cli/_testing_models.py`

> Internal chat models used by local integration tests.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from langchain_core.language_models.fake_chat_models import GenericFakeChatModel`

- `from langchain_core.messages import AIMessage, BaseMessage`

- `from langchain_core.outputs import ChatGeneration, ChatResult`


## Functions and classes

### `DeterministicIntegrationChatModel`

Deterministic chat model for CLI integration tests.

Additional notes from the source docstring:

```text
This subclasses LangChain's `GenericFakeChatModel` so the implementation
stays aligned with the core fake-chat-model test surface, while overriding
generation to remain prompt-driven and restart-safe for real CLI server
integration tests.

Why the existing `langchain_core` fakes cannot be reused here:

1. Every core fake (`GenericFakeChatModel`, `FakeListChatModel`,
    `FakeMessagesListChatModel`) pops from an iterator or cycles an index —
    the actual prompt is ignored. CLI integration tests start and stop the
    server process, which resets in-memory state. An iterator-based model
    either raises `StopIteration` or replays from the beginning after a
    restart, producing wrong or missing responses. This model derives output
    solely from the prompt text, so identical input always produces
    identical output regardless of process lifecycle.

2. The agent runtime calls `model.bind_tools(schemas)` during
    initialization. None of the core fakes implement `bind_tools`, so they
    raise `AttributeError` in any agent-loop context. This model provides a
    no-op passthrough.

3. The CLI server reads `model.profile` for capability negotiation (e.g.
    `tool_calling`, `max_input_tokens`). Core fakes have no such attribute,
    causing `AttributeError` or silent misconfiguration at runtime.

Additionally, the compact middleware issues summarization prompts mid-
conversation. A list-based model cannot distinguish these from normal user
turns without pre-knowledge of exact call ordering, whereas this model
detects summary requests by inspecting the prompt content.
```

Methods worth reading inside this class:

- `bind_tools(self, tools: Sequence[dict[str, Any] | type | Callable | BaseTool], *, tool_choice: str | None=None, **kwargs: Any)`: Return self so the agent can bind tool schemas during tests.

- `_generate(self, messages: list[BaseMessage], stop: list[str] | None=None, run_manager: CallbackManagerForLLMRun | None=None, **kwargs: Any)`: Produce a deterministic reply derived from the prompt text.

- `_llm_type(self)`: Return the LangChain model type identifier.

- `_stringify_message(message: BaseMessage)`: Flatten message content into plain text for deterministic responses.

- `_looks_like_summary_request(prompt: str)`: Detect the middleware's summary-generation prompt.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
