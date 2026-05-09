# `libs/deepagents/deepagents/middleware/_utils.py`

> Shared middleware helper module. Today it contains the system-message append
> utility used by prompt-injecting middleware.

## Position in the system

`MemoryMiddleware`, `SkillsMiddleware`, `SubAgentMiddleware`,
`AsyncSubAgentMiddleware`, `SummarizationToolMiddleware`, and filesystem
request hooks all need to append text to the request's system message without
discarding existing content blocks. This helper centralizes that operation.

## Imports and module-level state

The module imports `ContentBlock` and `SystemMessage` from LangChain Core. It
has no constants or import-time side effects.

## Functions and classes

### `append_to_system_message(system_message, text)`

Returns a new `SystemMessage` whose content blocks contain the existing system
message blocks followed by a new text block. If there was already system
content, the appended text is prefixed with two newlines so independent
middleware prompt sections remain visually separated.

The helper does not mutate the original `SystemMessage`; callers pass the
returned message into `request.override(system_message=...)`. If
`system_message` is `None`, it creates a one-block system message containing
only `text`.

## Gotchas

The helper appends a dictionary content block, not a plain string. Middleware
that later inspects provider-specific block fields should preserve that shape.
