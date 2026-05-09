# `libs/cli/deepagents_cli/ask_user.py`

> Ask user middleware for interactive question-answering during agent execution.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from langchain.agents.middleware.types import AgentMiddleware, ContextT, ModelRequest, ModelResponse, ResponseT`

- `from langchain.tools import InjectedToolCallId`

- `from langchain_core.messages import AIMessage, SystemMessage, ToolMessage`

- `from langchain_core.tools import tool`

- `from langgraph.types import Command, interrupt`

- `from deepagents_cli._ask_user_types import AskUserRequest, Question`


## Functions and classes

### `_validate_questions(questions: list[Question])`

Validate ask_user question structure before interrupting.

Additional notes from the source docstring:

```text
Args:
    questions: Question definitions provided to the `ask_user` tool.

Raises:
    ValueError: If the questions list or an individual question is invalid.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_parse_answers(response: object, questions: list[Question], tool_call_id: str)`

Parse an interrupt response into a `Command` with a `ToolMessage`.

Additional notes from the source docstring:

```text
Supports explicit status signaling from the adapter:

- `answered` (default): consume provided `answers`
- `cancelled`: synthesize `(cancelled)` answers
- `error`: synthesize `(error: ...)` answers

Malformed payloads are converted into explicit error answers instead of
silently defaulting to `(no answer)`.

Args:
    response: Raw value returned by `interrupt()`.
    questions: The questions that were asked.
    tool_call_id: Originating tool call ID for the `ToolMessage`.

Returns:
    `Command` containing a formatted `ToolMessage` with Q&A pairs.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `AskUserMiddleware`

Middleware that provides an ask_user tool for interactive questioning.

Additional notes from the source docstring:

```text
This middleware adds an `ask_user` tool that allows agents to ask the user
questions during execution. Questions can be free-form text or multiple choice.
The tool uses LangGraph interrupts to pause execution and wait for user input.
```

Methods worth reading inside this class:

- `wrap_model_call(self, request: ModelRequest[ContextT], handler: Callable[[ModelRequest[ContextT]], ModelResponse[ResponseT]])`: Inject the ask_user system prompt.

- `awrap_model_call(self, request: ModelRequest[ContextT], handler: Callable[[ModelRequest[ContextT]], Awaitable[ModelResponse[ResponseT]]])`: Inject the ask_user system prompt (async).


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

## System prompts, tool descriptions, and templates

### `ASK_USER_TOOL_DESCRIPTION`

```text
Ask the user one or more questions when you need clarification or input before proceeding.

Each question can be either:
- "text": Free-form text response from the user
- "multiple_choice": User selects from predefined options (an "Other" option is always available)

For multiple choice questions, provide a list of choices. The user can pick one or type a custom answer via the "Other" option.

By default all questions are required. Set "required" to false for optional questions that the user can skip. Do not include "(required)", "(optional)", "- optional", or similar annotations in the question text — the UI renders that separately based on the "required" field.

Use this tool when:
- You need clarification on ambiguous requirements
- You want the user to choose between multiple valid approaches
- You need specific information only the user can provide
- You want to confirm a plan before executing it

Do NOT use this tool for:
- Simple yes/no confirmations (just proceed with your best judgment)
- Questions you can answer yourself from context
- Trivial decisions that don't meaningfully affect the outcome
```

### `ASK_USER_SYSTEM_PROMPT`

```text
## `ask_user`

You have access to the `ask_user` tool to ask the user questions when you need clarification or input.
Use this tool sparingly - only when you genuinely need information from the user that you cannot determine from context.

When using `ask_user`:
- Be concise and specific with your questions
- Use multiple choice when there are clear options to choose from
- Use text input when you need free-form responses
- Group related questions into a single ask_user call rather than making multiple calls
- Never ask questions you can answer yourself from the available context
```

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
