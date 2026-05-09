# `libs/cli/deepagents_cli/app.py`

> Textual application runtime for the interactive CLI.

## Position in the system

`main.py` starts a server session and constructs the app with a `RemoteAgent`.
The app renders the conversation and resumes graph interrupts; the agent graph
runs in the LangGraph server subprocess.

## Functions and classes

This large file defines the Textual `App` subclass, app-level messages/events,
layout composition, key bindings, stream tasks, command dispatch, approval
handling, selector flows, and shutdown behavior.

### Lifecycle and layout

The app composes message transcript, chat input, status, approval surfaces,
selectors, and notification widgets. Startup loads thread state and initial
context; shutdown cancels stream tasks and returns an `AppResult`.

### Stream handling

The app consumes `RemoteAgent.astream()` tuples. Message chunks update
assistant widgets; update chunks create/finish tool widgets or surface
interrupts; errors become error messages. Subgraph namespaces identify
subagent streams.

### Input and commands

Text from `ChatInput` is either local slash-command control traffic or a human
message submitted to the graph. Commands can switch model/agent/thread, open
views, compact context, manage MCP, or exit.

### Approvals

LangGraph interrupts become approval widgets. The user's decision is sent back
as a resume command so the paused graph can continue.

## Gotchas

The app coordinates UI state; it is not the source of truth for conversation
state. Checkpoints in the server are authoritative.
