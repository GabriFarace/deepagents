# `libs/cli/deepagents_cli/widgets/ask_user.py`

> Ask user widget for interactive questions during agent execution.

## Position in the system

This widget sits on the Textual side of the CLI. `app.py` composes these screens and controls around streamed events from the LangGraph server, while the widget itself owns presentation, local interaction state, and the small event messages it emits upward.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from textual.binding import Binding, BindingType`

- `from textual.containers import Container, Vertical`

- `from textual.content import Content`

- `from textual.message import Message`

- `from textual.widgets import Markdown, Static, TextArea`

- `from deepagents_cli import theme`

- `from deepagents_cli.config import get_glyphs, is_ascii_mode`


## Functions and classes

### `AskUserTextArea`

Soft-wrapping text input for free-form ask-user questions.

Additional notes from the source docstring:

```text
Long answers wrap visually and the widget grows up to its CSS `max-height`.
Enter submits; Shift/Alt/Ctrl+Enter and Ctrl+J insert a literal newline
for users who want to author multi-paragraph answers.
```

Methods worth reading inside this class:

- `action_insert_newline(self)`: Insert a newline at the cursor.

- `_on_key(self, event: events.Key)`: This helper performs one narrow step for the surrounding module while keeping normalization, error handling, or side-effect policy in one place.

- `_find_question_widget(self)`: Walk up to find the enclosing `_QuestionWidget`, if any.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `AskUserMenu`

Interactive widget for asking the user questions.

Additional notes from the source docstring:

```text
Supports text input and multiple choice questions. Multiple choice
questions always include an "Other" option for free-form input.
```

Methods worth reading inside this class:

- `set_future(self, future: asyncio.Future[AskUserWidgetResult])`: Set the future to resolve when user answers.

- `compose(self)`: This helper performs one narrow step for the surrounding module while keeping normalization, error handling, or side-effect policy in one place.

- `on_mount(self)`: This helper performs one narrow step for the surrounding module while keeping normalization, error handling, or side-effect policy in one place.

- `focus_active(self)`: Focus the current active question's input.

- `on_ask_user_text_area_submitted(self, event: AskUserTextArea.Submitted)`: Confirm the question whose text area was submitted.

- `confirm_and_advance(self, index: int)`: Confirm the answer at `index` and advance to the next question.

- `_set_active_question(self, index: int)`: Update the visual indicator and focus for the active question.

- `_submit(self)`: This helper performs one narrow step for the surrounding module while keeping normalization, error handling, or side-effect policy in one place.

- `action_next_question(self)`: Navigate to the next question without confirming.

- `action_previous_question(self)`: Navigate to the previous question without confirming.

- `action_cancel(self)`: This helper performs one narrow step for the surrounding module while keeping normalization, error handling, or side-effect policy in one place.

- `on_blur(self, event: events.Blur)`: Prevent blur from propagating and dismissing the menu.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `_ChoiceOption`

A single selectable choice option.

Methods worth reading inside this class:

- `toggle(self)`: Toggle the selected state.

- `select(self)`: Mark this choice as selected.

- `deselect(self)`: Mark this choice as deselected.

- `_render(self)`: Build display content with cursor prefix.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `_QuestionWidget`

Widget for a single question (text or multiple choice).

Methods worth reading inside this class:

- `compose(self)`: This helper performs one narrow step for the surrounding module while keeping normalization, error handling, or side-effect policy in one place.

- `focus_input(self)`: Focus the appropriate input for this question.

- `get_answer(self)`: Return the current answer text for this question.

- `action_move_up(self)`: Move selection up in the choice list.

- `action_move_down(self)`: Move selection down in the choice list.

- `action_select_or_submit(self)`: Confirm current choice or open the Other input.

- `_find_menu(self)`: This helper performs one narrow step for the surrounding module while keeping normalization, error handling, or side-effect policy in one place.

- `_update_choice_selection(self)`: This helper performs one narrow step for the surrounding module while keeping normalization, error handling, or side-effect policy in one place.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

## Gotchas

Widget classes are coupled to Textual message names, CSS classes, and screen lifecycle hooks. Changing identifiers here can break bindings in `app.py` or stylesheet selectors even when type checks pass.
