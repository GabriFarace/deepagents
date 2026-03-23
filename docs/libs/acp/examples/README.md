# `libs/acp/examples/`

## What This Directory Contains

Example files demonstrating how to build a production-quality ACP-served coding agent. Includes a full agent setup with multi-mode operation, model switching, and an `AgentMiddleware` for injecting local development context.

## Files

| File | Description |
|------|-------------|
| `demo_agent.py` | Full demo coding agent: ACP server + multi-mode + model switching + local shell backend |
| `local_context.py` | `LocalContextMiddleware` — detects and injects git/project/runtime context into system prompt |
| `__init__.py` | Package marker |

## Architecture

```
demo_agent.py
    │
    ├── AgentServerACP(agent=build_agent, modes=3, models=6)
    │
    └── build_agent(context: AgentSessionContext):
            ├── model: context.model (dynamically switched)
            ├── checkpointer: MemorySaver (shared)
            ├── interrupt_on: _get_interrupt_config(context.mode)
            ├── backend: CompositeBackend
            │   ├── default: LocalShellBackend(root_dir=context.cwd)
            │   └── /memories/, /conversation_history/ → StateBackend
            └── middleware: [LocalContextMiddleware(backend)]
                            └── Runs bash detection script
                                Injects: CWD, git, project, runtimes,
                                         package managers, files, tree
```

## Key Patterns Demonstrated

- **Context-driven agent factory**: `build_agent` receives `AgentSessionContext` (cwd, mode, model) and constructs a fresh agent per session — enables per-session filesystem isolation and mode-specific interrupt behavior.
- **LocalContextMiddleware**: Runs a parallel bash detection script on first interaction and after summarization events; injects structured markdown (project type, git state, runtimes, directory tree, Makefile preview) into the system prompt.
- **CompositeBackend routing**: Default route goes to `LocalShellBackend` for real filesystem ops; special paths like `/memories/` route to an ephemeral `StateBackend`.
- **Mode-based interrupts**: Three modes controlling which tool calls require human approval, mapped cleanly via `_get_interrupt_config()`.

## Related Docs

- [`demo_agent.md`](demo_agent.md) — Full agent assembly and mode/model configuration
- [`local_context.md`](local_context.md) — Middleware, bash detection script, section functions
