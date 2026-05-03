# `libs/cli/deepagents_cli/server_manager.py`

## High-Level Purpose

`server_manager.py` orchestrates the LangGraph server startup sequence. It is the bridge between the TUI (which needs a running agent) and the server subprocess (which needs a workspace and configuration). It scaffolds a temporary workspace, writes all required config files, starts the `langgraph dev` subprocess, waits for it to become ready, and returns a `RemoteAgent` client to the TUI.

---

## Key Function

### `start_server_and_get_agent(server_kwargs) → tuple[RemoteAgent, str]`

The single public function of this module. Called from `CLIApp._start_server()` during TUI mount. Returns `(remote_agent, server_url)`.

**Steps:**

1. **Workspace scaffold** — creates a temporary directory (`/tmp/deepagents-server-{hash}/`) containing:
   - `server_graph.py` — copied from the CLI package's own `server_graph.py`
   - `checkpointer.py` — generated from a template; reads the SQLite DB path from `DEEPAGENTS_CLI_DB_PATH` env var
   - `pyproject.toml` — generated with `deepagents-cli` as the only dependency, pointing to the installed CLI package
   - `langgraph.json` — LangGraph server config pointing to `server_graph:make_graph`

2. **Environment preparation** — sets `DEEPAGENTS_CLI_SERVER_*` environment variables that the server subprocess will read to configure itself:
   - `DEEPAGENTS_CLI_SERVER_MODEL` — model spec
   - `DEEPAGENTS_CLI_SERVER_MCP_CONFIG` — path to MCP config file
   - `DEEPAGENTS_CLI_SERVER_AGENT` — selected agent name
   - `DEEPAGENTS_CLI_SERVER_AUTO_APPROVE` — HITL mode
   - `DEEPAGENTS_CLI_DB_PATH` — SQLite checkpoint database path

3. **Subprocess start** — calls `ServerProcess.start()` which runs:
   ```
   uv run langgraph dev --host 127.0.0.1 --port <ephemeral_port>
   ```
   inside the scaffolded workspace directory.

4. **Readiness poll** — polls the server's `/ok` health endpoint every 200ms until it responds 200 (or times out after ~30 seconds).

5. **Client creation** — creates a `RemoteAgent` pointing to `http://127.0.0.1:<port>` and returns it.

---

## Workspace Structure

```
/tmp/deepagents-server-{hash}/
├── server_graph.py       ← copied from CLI package (make_graph entry point)
├── checkpointer.py       ← generated; wires up SQLite checkpointer
├── pyproject.toml        ← generated; [project] name + deepagents-cli dep
└── langgraph.json        ← generated; {"graphs": {"agent": "server_graph:make_graph"}}
```

The workspace is reused across sessions if the config hasn't changed. On first run (or when config changes), it is recreated from scratch.

---

## Architecture Notes

**Why a temporary workspace?** `langgraph dev` requires a Python package with a `langgraph.json` pointing to a graph factory. Rather than requiring the user to create this manually, the CLI generates it on the fly. The workspace is stable across invocations (same hash = same dir), so `uv` doesn't reinstall dependencies on every launch.

**Port selection:** An ephemeral port is chosen by binding to port 0 and reading the assigned port back. This avoids conflicts with other running services.

**Database path:** The SQLite checkpoint database is stored at `~/.deepagents/.state/threads.db` by default. The path is passed to the server via environment variable, so the TUI and server agree on the same file.

---

## See Also

- [server.md](server.md) — `ServerProcess` (the subprocess manager)
- [server_graph.md](server_graph.md) — `make_graph()` inside the server
- [remote_client.md](remote_client.md) — the `RemoteAgent` client returned here
- [app.md](app.md) — `CLIApp._start_server()` that calls this function
