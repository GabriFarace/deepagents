# `libs/cli/deepagents_cli/widgets/approval.py`

> Textual approval menu for LangGraph human-in-the-loop interrupts.

## Functions and classes

### `ApprovalMenu`

Renders approval choices, tracks keyboard/mouse selection, and emits the user's
decision to the app. The app turns that decision into a graph resume command.

## Gotchas

This widget does not execute tools. It only captures the decision.
