# `libs/cli/deepagents_cli/server.py`

> Low-level `langgraph dev` process lifecycle helpers.

## Position in the system

`server_manager.py` scaffolds a temp workspace, then this module starts and
stops the LangGraph server process. It knows about ports, config files, health
checks, subprocess logs, and environment, not about Textual widgets.

## Functions and classes

### Port and URL helpers

`_port_in_use()`, `_find_free_port()`, and `get_server_url()` support local
server binding and remote-client construction.

### `generate_langgraph_json(...)`

Writes the `langgraph.json` file used by `langgraph dev`, including graph ref,
optional env file, and optional checkpointer factory path.

### `_scoped_env_overrides(overrides)`

Temporarily mutates `os.environ`, rolling back on exception. It isolates failed
startup attempts from later attempts.

### `wait_for_server_healthy(...)`

Polls `{url}/ok` until healthy, failing early if the subprocess exits and
including a tail of logs for diagnosis.

### `_build_server_cmd()` and `_build_server_env()`

Construct subprocess argv and environment for `python -m langgraph_cli dev`.

### `ServerProcess`

Owns process start/stop. It chooses host/port, starts the subprocess, waits for
health, exposes the URL, and terminates or kills the process during cleanup.

## Gotchas

Generated graph/checkpointer paths are relative to the temp workspace because
the subprocess imports them from its own cwd.
