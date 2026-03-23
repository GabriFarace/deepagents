# `deepagents/middleware/_utils.py`

## High-Level Purpose

Internal utility functions for middleware implementations. Currently contains a single helper for manipulating `SystemMessage` content blocks.

## Dependencies

- `langchain_core.messages.ContentBlock`
- `langchain_core.messages.SystemMessage`

## Functions

### `append_to_system_message(system_message, text) -> SystemMessage`

Appends a text string to a `SystemMessage`'s content blocks.

**Parameters:**
- `system_message: SystemMessage | None` — An existing system message to append to, or `None` to create a new one.
- `text: str` — Text content to add. A `"\n\n"` separator is prepended if there are already existing content blocks.

**Returns:** A new `SystemMessage` with the text appended as a `{"type": "text", "text": ...}` content block.

**Key Logic:**
1. Starts with an empty list of content blocks if `system_message` is `None`.
2. Copies existing content blocks from `system_message.content_blocks` if present.
3. If there are existing blocks, prepends `"\n\n"` to `text` to ensure visual separation.
4. Appends a new text content block.
5. Returns a new `SystemMessage(content_blocks=...)`.

**Used by:** `MemoryMiddleware`, `SkillsMiddleware`, `SummarizationMiddleware`, `AsyncSubAgentMiddleware`, `SubAgentMiddleware`.
