# `libs/cli/deepagents_cli/_ask_user_types.py`

> Lightweight types for the ask-user interrupt protocol.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Functions and classes

### `Choice`

A single choice option for a multiple choice question.

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `Question`

A question to ask the user.

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `AskUserRequest`

Request payload sent via interrupt when asking the user questions.

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `AskUserAnswered`

Widget result when the user submits answers.

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `AskUserCancelled`

Widget result when the user cancels the prompt.

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
