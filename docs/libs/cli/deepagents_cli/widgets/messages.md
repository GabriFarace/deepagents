# `libs/cli/deepagents_cli/widgets/messages.py`

> Transcript widgets for user, assistant, tool, diff, error, skill, and app
> messages.

## Functions and classes

### Formatting helpers

`_show_timestamp_toast()`, `_mode_color()`, `_strip_success_exit_line()`,
`_strip_frontmatter()`, and `FormattedOutput` prepare display text.

### Message widgets

`UserMessage`, `QueuedUserMessage`, `SkillMessage`, `AssistantMessage`,
`ToolCallMessage`, `DiffMessage`, `ErrorMessage`, `AppMessage`, and
`SummarizationMessage` render the main transcript surfaces.

### `_MutedRichMarkdown`

Helper renderer for subdued markdown content inside the transcript.

## Gotchas

Tool display is split between generic message rendering and specialized
renderers/widgets.
