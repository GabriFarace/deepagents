# `sessions.py`

## High-Level Purpose

This module manages conversation thread persistence using LangGraph's SQLite checkpoint storage. It provides:

- Thread listing with filtering, sorting, and pagination
- Thread deletion
- Thread metadata extraction (initial prompt, message count, git branch, timestamps)
- A UUID7-based thread ID generator
- Timestamp formatting utilities
- An aiosqlite compatibility patch for newer langgraph-checkpoint versions

## Key Design

Threads are stored in a SQLite database (typically `~/.deepagents/checkpoints.db`) via LangGraph's `AsyncSqliteSaver`. The module reads checkpoint data directly to extract metadata such as message counts and initial prompts, using an in-process caching layer to avoid repeated database reads.

## Module-Level State

| Variable | Description |
|---|---|
| `_aiosqlite_patched` | Whether the `is_alive()` compatibility patch has been applied |
| `_jsonplus_serializer` | Cached `JsonPlusSerializer` for deserializing checkpoint data |
| `_message_count_cache` | LRU-like cache mapping `(thread_id, checkpoint_id)` to message count |
| `_initial_prompt_cache` | LRU-like cache mapping `(thread_id, checkpoint_id)` to initial prompt |
| `_recent_threads_cache` | Cache for recently listed threads |

## Classes

### `ThreadInfo`

**Type:** `TypedDict`

Thread metadata returned by `list_threads`.

| Key | Type | Description |
|---|---|---|
| `thread_id` | `str` | Unique identifier for the thread |
| `agent_name` | `str \| None` | Name of the agent that owns the thread |
| `updated_at` | `str \| None` | ISO timestamp of last update |
| `created_at` | `NotRequired[str \| None]` | ISO timestamp of thread creation |
| `git_branch` | `NotRequired[str \| None]` | Git branch active when created |
| `initial_prompt` | `NotRequired[str \| None]` | First human message in the thread |
| `message_count` | `NotRequired[int]` | Number of messages in the thread |
| `latest_checkpoint_id` | `NotRequired[str \| None]` | Most recent checkpoint ID |
| `cwd` | `NotRequired[str \| None]` | Working directory when last used |

### `_CheckpointSummary`

**Type:** `NamedTuple`

Structured data extracted from a thread's latest checkpoint.

| Field | Type | Description |
|---|---|---|
| `message_count` | `int` | Number of messages in the latest checkpoint |
| `initial_prompt` | `str \| None` | First human prompt from the checkpoint |

## Functions

### `_patch_aiosqlite() -> None`

Patches `aiosqlite.Connection` with an `is_alive()` method if it's missing, which is required by `langgraph-checkpoint>=2.1.0`.

### `_connect() -> AsyncIterator[aiosqlite.Connection]`

Async context manager that applies the compatibility patch and opens a connection to the sessions database.

### `format_timestamp(iso_timestamp: str | None) -> str`

Formats an ISO 8601 timestamp for human-readable display (e.g., `"Dec 30, 6:10pm"`).

**Returns:** Formatted timestamp string, or empty string if invalid/None.

### `generate_thread_id() -> str`

Generates a UUID7 thread ID using `uuid_utils`.

**Returns:** UUID7 string suitable as a LangGraph thread ID.

### `get_db_path() -> Path`

Returns the path to the SQLite checkpoint database.

**Returns:** Path to `~/.deepagents/checkpoints.db` (or the path configured in settings).

### `list_threads(*, agent_name=None, limit=20, sort="updated", branch=None, verbose=False, relative_time=False, output_format="text") -> list[ThreadInfo]`

Queries the checkpoint database and returns recent thread metadata.

**Parameters:**
- `agent_name`: Filter by agent name.
- `limit`: Maximum number of threads to return (default 20).
- `sort`: Sort key — `"updated"` or `"created"`.
- `branch`: Filter by git branch name.
- `verbose`: Include all columns (branch, created, prompt).
- `relative_time`: Show timestamps as relative time.
- `output_format`: `"text"` or `"json"`.

**Returns:** List of `ThreadInfo` dicts.

### `delete_thread(thread_id: str) -> bool`

Deletes all checkpoints for a thread from the database.

**Returns:** `True` if the thread existed and was deleted, `False` if not found.

### `get_or_create_checkpointer() -> AsyncIterator[AsyncSqliteSaver]`

Async context manager that yields an `AsyncSqliteSaver` connected to the sessions database.

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `aiosqlite` | `aiosqlite` | Async SQLite access |
| `langgraph.checkpoint.sqlite.aio.AsyncSqliteSaver` | `langgraph-checkpoint-sqlite` | Checkpoint persistence |
| `langgraph.checkpoint.serde.jsonplus.JsonPlusSerializer` | langgraph | Checkpoint deserialization |
| `uuid_utils` | `uuid-utils` | UUID7 generation |
