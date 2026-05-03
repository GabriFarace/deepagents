# `libs/cli/deepagents_cli/sessions.py`

## High-Level Purpose

`sessions.py` manages thread metadata for the CLI's conversation history. LangGraph handles the actual message checkpointing (via `AsyncSqliteSaver`); this module sits alongside it and stores the human-readable metadata that makes threads listable and resumable: the first prompt, the git branch, the working directory, and timestamps. The `/threads` command and `-r` flag both read from this layer.

---

## Key Classes and Functions

### `ThreadMetadata` (TypedDict)

The per-thread record stored in the metadata table.

| Field | Type | Description |
|---|---|---|
| `thread_id` | `str` | UUID hex, 32 chars |
| `agent_name` | `str \| None` | Which agent was active |
| `created_at` | `str` | ISO 8601 timestamp |
| `updated_at` | `str` | ISO 8601 timestamp |
| `git_branch` | `str \| None` | Git branch active when thread was created |
| `cwd` | `str` | Working directory at thread creation |
| `initial_prompt` | `str` | First human message (truncated to 120 chars) |

### `SessionStore`

Async manager for the thread metadata SQLite table. Stored at `~/.deepagents/.state/threads_meta.db` (separate from LangGraph's own checkpoint DB to avoid schema conflicts).

**Key methods:**

#### `async create_thread(thread_id, agent_name, initial_prompt) → ThreadMetadata`

Inserts a new thread record. Reads `git_branch` and `cwd` from the process environment at call time. Returns the created `ThreadMetadata`.

#### `async update_thread(thread_id, **kwargs) → None`

Updates `updated_at` and any other provided fields. Called after each agent response.

#### `async list_threads(agent_name=None, limit=20, sort="updated") → list[ThreadMetadata]`

Returns recent threads, optionally filtered by `agent_name`. Sort can be `"updated"` (most recently active first) or `"created"` (newest first).

#### `async get_thread(thread_id) → ThreadMetadata | None`

Returns a single thread record by ID. Returns `None` if not found.

#### `async get_most_recent(agent_name=None) → ThreadMetadata | None`

Returns the most recently updated thread, optionally filtered by agent. Used by `-r` without an explicit thread ID.

---

## Resume Flow

When the user runs `deepagents -r` (or `deepagents -r <thread_id>`):

1. `main.py::parse_args()` captures `args.resume` (may be `True` or a string ID)
2. `run_textual_cli_async()` passes `resume_thread_id` to `run_textual_app()`
3. `CLIApp._start_server()` calls `session_store.get_most_recent()` (if `resume=True`) or `session_store.get_thread(id)` (if an ID was given)
4. The resolved `thread_id` is passed to `RemoteAgent.astream()` — LangGraph picks up the checkpoint automatically

---

## Architecture Notes

**Two databases:** LangGraph's `AsyncSqliteSaver` stores the full message/state checkpoints in `threads.db`. `SessionStore` stores only human-readable metadata in `threads_meta.db`. This separation means the metadata table stays small and fast to query for listing, while the checkpoint table can be large.

**thread_id format:** Generated as `uuid.uuid4().hex` (32-char lowercase hex, no hyphens). LangGraph accepts any string as a thread_id.

**`updated_at` maintenance:** The TUI calls `session_store.update_thread()` after every successful agent response, so the "most recently active" sort order stays accurate.

---

## See Also

- [app.md](app.md) — where `create_thread()` and `update_thread()` are called
- [main.md](main.md) — `-r` flag parsing
- [command_registry.md](command_registry.md) — `/threads` slash command
- [widgets/thread_selector.md](widgets/README.md) — the `/threads` modal
