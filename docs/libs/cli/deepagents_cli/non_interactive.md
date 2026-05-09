# `libs/cli/deepagents_cli/non_interactive.py`

> Console streaming runner for `deepagents -p`.

## Position in the system

Uses the same server-backed `RemoteAgent` as the TUI but renders to stdout and
handles HITL policy without widgets.

## Functions and classes

### State and console helpers

`HITLIterationLimitError`, `_write_text()`, `_write_newline()`,
`_ConsoleSpinner`, `StreamState`, and `ThreadUrlLookupState` track output and
run state.

### Stream processors

`_process_interrupts()`, `_process_ai_message()`, `_process_message_chunk()`,
and `_process_stream_chunk()` translate stream events into console output.

### HITL helpers

`_make_hitl_decision()`, `_collect_action_request_warnings()`, and
`_process_hitl_interrupts()` apply non-interactive approval/allow-list policy.

### Run loop

`_stream_agent()`, `_run_agent_loop()`, `_build_non_interactive_header()`,
`_run_startup_command()`, and `run_non_interactive()` implement the print-mode
entry point.

## Gotchas

Because there is no approval UI, safety must be configured up front.
