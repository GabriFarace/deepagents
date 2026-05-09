# `libs/cli/deepagents_cli/sessions.py`

> Session/thread helpers around the SQLite LangGraph checkpoint database.

## Position in the system

The CLI uses LangGraph thread ids as sessions. This module lists, formats,
deletes, resumes, and summarizes those threads.

## Functions and classes

### Database helpers

`_patch_aiosqlite()`, `_connect()`, `get_db_path()`, and `get_checkpointer()`
connect the CLI to its SQLite checkpoint store.

### Formatting and data shapes

`ThreadInfo`, `_CheckpointSummary`, `format_timestamp()`,
`format_relative_timestamp()`, and `format_path()` support session UI display.

### Listing, caching, and checkpoint parsing

`list_threads()`, populate/prewarm/cache helpers, `_summarize_checkpoint()`,
`_checkpoint_messages()`, and initial-prompt extraction helpers turn raw
checkpoint blobs into useful list rows.

### Thread operations

`generate_thread_id()`, `get_most_recent()`, `get_thread_agent()`,
`thread_exists()`, `find_similar_threads()`, `delete_thread()`, and command
wrappers back resume and session-management UX.

## Gotchas

There is no separate chat database; session history is checkpoint state.
