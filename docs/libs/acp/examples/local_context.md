# `libs/acp/examples/local_context.py`

## High-Level Purpose

Provides `LocalContextMiddleware`, an `AgentMiddleware` that detects the local development environment (CWD, git state, project language, package managers, runtimes, test command, directory tree, Makefile) by running a bash script via the backend, and injects the results into the system prompt on every model call. Refreshes automatically after LangGraph summarization events.

## Classes

### `_ExecutableBackend` (Protocol)

**Purpose:** Structural protocol typing any backend with a synchronous `execute(command) -> ExecuteResponse` method. Used for type-safe backend injection.

---

### `LocalContextState` (TypedDict, extends AgentState)

**Purpose:** Agent state schema for the middleware.

**Fields:**
- `local_context: NotRequired[str]` — Formatted markdown context: CWD, project info, package managers, runtimes, git status, test command, file listing, tree, Makefile preview.
- `_local_context_refreshed_at_cutoff: NotRequired[Annotated[int, PrivateStateAttr]]` — Tracks which summarization event the context was last refreshed for (checkpointed, private — not exposed to subagents).

---

### `LocalContextMiddleware` (extends AgentMiddleware)

**Purpose:** Inject local environment context into the system prompt, running once on first interaction and refreshing after each summarization event.

**Attribute:**
- `state_schema = LocalContextState`
- `backend: _ExecutableBackend` — Backend used to execute the detection script.

**Constructor:**
```python
def __init__(self, backend: _ExecutableBackend) -> None
```

**Methods:**

#### `_run_detect_script() -> str | None`
Executes `DETECT_CONTEXT_SCRIPT` via `self.backend.execute()`. Returns stripped output on success, or `None` with a warning log on error or empty output.

#### `before_agent(state, runtime) -> dict | None`
Called before each agent turn. Two modes:
1. **Post-summarization refresh**: If `_summarization_event` in state has a new `cutoff_index`, re-runs the detection script and returns updated `local_context` + `_local_context_refreshed_at_cutoff`. If script fails, records cutoff without updating context (prevents retry loops).
2. **Initial detection**: If `local_context` not yet set, runs the script and returns `{"local_context": output}`.

Returns `None` if context already set and no refresh needed.

#### `wrap_model_call(request, handler) -> ModelResponse`
Appends `local_context` from state to the end of the system prompt (separated by two newlines). Returns the unmodified handler response if no context is available.

#### `awrap_model_call(request, handler) -> ModelResponse` (async)
Same as `wrap_model_call` but for async invocation paths.

## Module-Level Functions

### `_section_header() -> str`
Returns bash snippet printing `## Local Context`, `**Current Directory**: <CWD>`, and setting `CWD` and `IN_GIT` variables.

### `_section_project() -> str`
Returns bash snippet detecting: primary language (Python/JS+TS/Rust/Go/Java via lock files), monorepo detection (`lerna.json`, `pnpm-workspace.yaml`, `packages/`, `libs/+apps/`, `workspaces/`), git project root, and virtual env detection (`.venv`, `node_modules`).

### `_section_package_managers() -> str`
Detects Python package manager (uv/poetry/pipenv/pip by lock file and pyproject.toml markers) and Node package manager (bun/pnpm/yarn/npm by lock file).

### `_section_runtimes() -> str`
Detects Python 3 and Node version strings via `python3 --version` and `node --version`.

### `_section_git() -> str`
Detects current branch, available `main`/`master` branches, and uncommitted change count via `git rev-parse` and `git status --porcelain`.

### `_section_test_command() -> str`
Detects preferred test command: `make test` (if `Makefile` has a `test:` target), `pytest` (if `pyproject.toml` has pytest config or a `tests/` dir), or `npm test` (if `package.json` has a `"test"` script).

### `_section_files() -> str`
Lists files in CWD (excluding `node_modules`, `__pycache__`, `.pytest_cache`, build/cache dirs), showing at most 20 entries with a count of additional files.

### `_section_tree() -> str`
Runs `tree -L 3 --noreport --dirsfirst` (if `tree` is available), excluding common cache/build directories, capped at 22 lines.

### `_section_makefile() -> str`
Shows the first 20 lines of the nearest `Makefile` (CWD or git root for monorepos).

### `build_detect_script() -> str`

**Purpose:** Assembles all sections into a complete bash heredoc ready for `backend.execute()`.

**Key Logic:**
- Header and project sections run synchronously (they set `CWD`, `IN_GIT`, `ROOT` that other sections depend on via bash subshell fork).
- The 7 independent sections (package managers, runtimes, git, test command, files, tree, Makefile) run as parallel background subshells, each writing to a temp file. Results are concatenated in display order after `wait`.
- Uses `mktemp -d` with `trap 'rm -rf ...' EXIT` for temp dir cleanup.
- Returns a `bash <<'__DETECT_CONTEXT_EOF__' ... __DETECT_CONTEXT_EOF__` heredoc string.

## Module-Level Objects

- `DETECT_CONTEXT_SCRIPT = build_detect_script()` — Eagerly computed at module import time.

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| `AgentMiddleware`, `AgentState`, `ModelRequest`, `ModelResponse`, `PrivateStateAttr` | `langchain.agents.middleware.types` | Middleware base types |
| `ExecuteResponse` | `deepagents.backends.protocol` | Backend execute return type |
| `SummarizationEvent` | `deepagents.middleware.summarization` | Summarization event type |
| `Runtime` | `langgraph.runtime` | LangGraph runtime context |
