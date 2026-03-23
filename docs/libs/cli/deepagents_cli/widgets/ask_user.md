# `widgets/ask_user.py`

## High-Level Purpose

This module defines the `AskUserMenu` widget for interactive questions during agent execution. The agent can pause and prompt the user with text input questions or multiple-choice questions. Multiple-choice questions always include an "Other (type your answer)" option for free-form responses.

## Module-Level Constants

| Constant | Value | Description |
|---|---|---|
| `OTHER_CHOICE_LABEL` | `"Other (type your answer)"` | Label for the free-form option in multiple-choice questions |

## Classes

### `AskUserMenu`

**Inherits from:** `textual.containers.Container`

Interactive widget for asking the user questions.

**Class Variables:**
| Attribute | Value | Description |
|---|---|---|
| `can_focus` | `True` | Widget can receive keyboard focus |
| `can_focus_children` | `True` | Children (Input widgets) can also receive focus |

**Bindings:**
| Key | Action | Description |
|---|---|---|
| `escape` | `cancel` | Cancel the question |
| `tab` | `next_question` | Move to the next question |

**Inner Messages:**

- `Answered(answers: list[str])` — Sent when the user submits all answers. `answers` is a list of strings in question order.
- `Cancelled()` — Sent when the user cancels the ask_user prompt.

**Constructor:**
```python
AskUserMenu(
    questions: list[Question],
    id: str | None = None,
    **kwargs
)
```

**Parameters:**
- `questions`: List of `Question` dicts (each has `text` and optional `choices`).
- `id`: Optional widget ID.

**Methods:**

- `set_future(future: asyncio.Future) -> None` — Sets the future to resolve when the user answers. The app uses this to bridge async agent execution.
- `compose() -> ComposeResult` — Creates `_QuestionWidget` for each question.
- `action_cancel() -> None` — Posts `Cancelled` and resolves the future with a cancelled result.
- `action_next_question() -> None` — Advances focus to the next question.

**Private Classes:**

- `_QuestionWidget` — Container for a single question, renders either a text `Input` or a multiple-choice list of clickable option widgets plus an "Other" `Input`.

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `textual.containers.Container` | textual | Base container |
| `textual.widgets.Input` | textual | Text input field |
| `asyncio.Future` | stdlib | Async resolution for agent bridge |
| `AskUserWidgetResult`, `Question` | `deepagents_cli._ask_user_types` | Type definitions |
| `theme` | `deepagents_cli.theme` | Brand colors |
