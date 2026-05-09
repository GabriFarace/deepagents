# Book Diary — `deepagents` Documentation Project

> **Purpose.** This file is a persistent progress log for the multi-session effort
> to rewrite `docs/` from scratch. It records what has been done, what is in flight,
> what is queued, and any decisions made along the way. When a session ends due to
> token exhaustion, the next session **MUST** begin by reading this file and
> resuming from the "Next up" section.

---

## Project goals (confirmed with user 2026-05-09)

1. **Self-contained, comprehensive documentation** of the `deepagents` codebase, with
   an emphasis on **how the orchestration works**, the **deep-agent flow**, the
   **system prompts**, and the **CLI** (which the user is most interested in —
   they want to understand how agent CLIs like Claude Code, Codex, and deepagents-cli
   are built).
2. **Block-by-block code explanations.** Each source file documented function-by-function
   and class-by-class, with the logic of each block explained in 1–3 paragraphs.
   This is **deeper than the previous docs** (which were API-surface only).
3. **Self-contained prerequisite chapters** for LangGraph and LangChain, sourced from
   the official documentation, covering exactly the concepts deepagents uses so the
   reader does not have to leave the docs.
4. **Generic CLI architecture chapter** that contextualises deepagents-cli's design
   among the broader pattern of agent CLIs (Claude Code, Codex, etc.).
5. **CI/CD, infrastructure, and release tooling are out of scope for now.**
   They will be addressed in a later pass.
6. **Cost discipline.** The user has asked us to reserve **Opus 4.7** for the
   most important work (orchestration core, system prompts, CLI core, the
   roadmap, prerequisites) and use **cheaper models (Sonnet, Haiku) via subagents**
   for leaf documentation (peripheral packages, examples, simple helpers).

---

## Source-tree map (reconnaissance, 2026-05-09)

### Core SDK — `libs/deepagents/` (~25 source files)

```
libs/deepagents/deepagents/
├── __init__.py                     ← public re-exports
├── _version.py
├── _models.py                      ← model resolution (string → ChatModel)
├── _tools.py                       ← built-in tool helpers
├── _excluded_middleware.py
├── graph.py                        ← create_deep_agent() — THE entry point
├── _api/
│   ├── __init__.py
│   └── deprecation.py
├── backends/                       ← BackendProtocol + 8 implementations
│   ├── protocol.py                 ← the contract
│   ├── state.py                    ← StateBackend (default)
│   ├── filesystem.py               ← FilesystemBackend
│   ├── local_shell.py              ← LocalShellBackend
│   ├── sandbox.py                  ← BaseSandbox
│   ├── store.py                    ← StoreBackend (LangGraph BaseStore)
│   ├── composite.py                ← CompositeBackend (path-prefix routing)
│   ├── langsmith.py                ← LangSmith integration
│   └── utils.py
├── middleware/                     ← AgentMiddleware subclasses
│   ├── filesystem.py               ← all file/shell tools
│   ├── subagents.py                ← `task` delegation tool
│   ├── async_subagents.py          ← remote background tasks
│   ├── memory.py                   ← AGENTS.md loading
│   ├── skills.py                   ← skill catalog
│   ├── summarization.py            ← context compaction
│   ├── patch_tool_calls.py         ← dangling tool call fixes
│   ├── permissions.py              ← HITL + permission gates
│   ├── _tool_exclusion.py
│   └── _utils.py
└── profiles/                       ← per-model harness/provider profiles
    ├── _builtin_profiles.py
    ├── _keys.py
    ├── harness/                    ← claude opus/sonnet/haiku, openai codex
    └── provider/                   ← openai, openrouter
```

> **Note.** There is no `state.py` or `types.py` at the package root in this version.
> State types live inside `graph.py` and middleware files.

### CLI — `libs/cli/` (~95 source files, heaviest focus)

```
libs/cli/deepagents_cli/
├── main.py                         ← CLI entry point
├── __main__.py
├── app.py                          ← Textual TUI app
├── agent.py                        ← agent factory used by the embedded server
├── server.py                       ← langgraph dev subprocess controller
├── server_manager.py               ← lifecycle of the spawned LG server
├── server_graph.py                 ← graph definition the spawned server runs
├── remote_client.py                ← HTTP/SSE client to the LG server
├── input.py                        ← input handling
├── ui.py / output.py               ← rendering helpers
├── command_registry.py             ← slash commands
├── sessions.py                     ← session persistence
├── hooks.py                        ← user-configurable hooks
├── subagents.py                    ← subagent orchestration glue
├── config.py / model_config.py / configurable_model.py
├── tools.py / tool_display.py / file_ops.py
├── mcp_tools.py / mcp_auth.py / mcp_commands.py / mcp_trust.py
├── mcp_providers/                  ← github, slack registries
├── skills/                         ← skill load/invoke/commands
├── deploy/                         ← deepagents deploy bundling
├── integrations/                   ← sandbox factory/provider
├── built_in_skills/                ← shipped skills (skill-creator)
├── widgets/                        ← Textual widgets (28 files)
│   ├── messages.py / message_store.py
│   ├── chat_input.py / approval.py / ask_user.py
│   ├── notification_*.py / model_selector.py / theme_selector.py
│   ├── thread_selector.py / agent_selector.py / autocomplete.py
│   ├── tool_widgets.py / tool_renderers.py / diff.py / loading.py
│   └── ... (welcome, history, status, mcp_viewer, auth, launch_init,
│            update_available, _links)
├── editor.py / clipboard.py / formatting.py
├── theme.py / unicode_security.py / terminal_capabilities.py / iterm_cursor_guide.py
├── notifications.py / event_bus.py / token_state.py / _session_stats.py
├── update_check.py / extras_info.py / state_migration.py
├── auth_store.py / project_utils.py / local_context.py
├── ask_user.py / _ask_user_types.py
├── offload.py / non_interactive.py / textual_adapter.py / _textual_patches.py
├── _cli_context.py / _server_config.py / _git.py / _debug.py
├── _constants.py / _env_vars.py / _testing_models.py / _version.py
├── media_utils.py / onboarding.py
└── scripts/check_imports.py
```

### Adapters and peripheral packages

- `libs/acp/` (6 files) — ACP server adapter (`server.py`, `utils.py`, examples).
- `libs/evals/` (15 files) — `deepagents_evals/` (`cli.py`, `radar.py`,
  `trial_summary.py`) and `deepagents_harbor/` (`backend.py`,
  `deepagents_wrapper.py`, `failure.py`, `langsmith*.py`, `metadata.py`, `stats.py`)
  plus scripts.
- `libs/partners/` (12 files):
  - `daytona/langchain_daytona/sandbox.py`
  - `modal/langchain_modal/sandbox.py`
  - `quickjs/langchain_quickjs/` (sandbox `_repl.py`, `_skills.py`, `_format.py`,
    `_prompt.py`, `_ptc.py`, `middleware.py`)
  - `runloop/langchain_runloop/sandbox.py`
- `libs/code/` (init+version only) — placeholder package.
- `libs/repl/langchain_repl/` (`interpreter.py`, `middleware.py`,
  `_foreign_function_docs.py`).

### Examples — `examples/` (14 agents)

`async-subagent-server`, `better-harness`, `content-builder-agent`, `deep_research`,
`deploy-coding-agent`, `deploy-content-writer`, `deploy-gtm-agent`,
`deploy-mcp-docs-agent`, `downloading_agents`, `nvidia_deep_agent`,
`ralph_mode`, `repl_swarm`, `rlm_agent`, `text-to-sql-agent`.

---

## Roadmap stages and ownership

**Opus 4.7 (this assistant directly):**
- Stage 0 — Reconnaissance (DONE)
- Stage 1 — Scaffolding (`book_diary.md`, `instructions.md`, `CLAUDE.md`,
  `docs/` directory tree)
- Stage 2 — Prerequisites chapters (`docs/prerequisites/langchain.md`,
  `docs/prerequisites/langgraph.md`)
- Stage 3 — `docs/ROADMAP.md` and `docs/README.md`
- Stage 4 — Core SDK orchestration: `graph.py`, `_models.py`, `backends/protocol.py`,
  `middleware/__init__.py`, `middleware/subagents.py`, `middleware/filesystem.py`,
  system prompts.
- Stage 5 — CLI core: `main.py`, `app.py`, `server.py`, `server_manager.py`,
  `server_graph.py`, `remote_client.py`, `command_registry.py`, `sessions.py`,
  `hooks.py`, `mcp_tools.py`, key widgets (`approval.py`, `messages.py`,
  `chat_input.py`).
- Stage 6 — Generic CLI architecture chapter
  (`docs/libs/cli/cli_architecture.md`).

**Sonnet 4.6 (via Agent subagents):**
- Stage 7 — Remaining SDK middleware (`memory.py`, `skills.py`, `summarization.py`,
  `permissions.py`, `patch_tool_calls.py`, `async_subagents.py`).
- Stage 8 — Backends other than `protocol.py` (state, filesystem, local_shell,
  sandbox, store, composite, langsmith).
- Stage 9 — Profiles (`profiles/harness/`, `profiles/provider/`).
- Stage 10 — Remaining CLI files (helpers, configs, widgets, deploy/, skills/,
  mcp_providers/, integrations/).
- Stage 11 — `libs/acp/`, `libs/evals/`, `libs/partners/`, `libs/repl/`.

**Haiku 4.5 (via Agent subagents):**
- Stage 12 — `examples/` agents (one short doc each).
- Stage 13 — Trivial files (`__init__.py` re-exports, `_version.py`,
  `py.typed.py`).

---

## Decisions log

- **2026-05-09** — Confirmed block-level (function/class) granularity, not line-by-line.
- **2026-05-09** — Confirmed clean rebuild of `docs/`.
- **2026-05-09** — Confirmed self-contained LangChain/LangGraph primers via WebFetch.
- **2026-05-09** — Confirmed generic CLI architecture chapter that contextualises
  deepagents-cli among other agent CLIs (no scraping of closed-source repos).
- **2026-05-09** — CI/CD, infrastructure, release tooling explicitly excluded
  from the current pass.

---

## Per-stage status

| Stage | Description | Status | Owner |
|---|---|---|---|
| 0 | Reconnaissance | DONE | Opus |
| 1a | `book_diary.md` | DONE | Opus |
| 1b | Rewrite `instructions.md` | DONE | Opus |
| 1c | Update root `CLAUDE.md` | DONE | Opus |
| 1d | Build `docs/` directory scaffold | DONE | Opus |
| 2a | `docs/prerequisites/langchain.md` | DONE | Opus |
| 2b | `docs/prerequisites/langgraph.md` | DONE | Opus |
| 3a | `docs/ROADMAP.md` | DONE | Opus |
| 3b | `docs/README.md` | DONE | Opus |
| 4 | SDK orchestration core | DONE | Opus |
| 5 | CLI core | DONE | Opus |
| 6 | Generic CLI architecture chapter | DONE | Opus |
| 7 | Remaining SDK middleware | DONE | Sonnet |
| 8 | Remaining backends | DONE | Sonnet |
| 9 | Profiles | DONE | Sonnet |
| 10 | Remaining CLI files | DONE | Sonnet |
| 11 | acp/evals/partners/repl | DONE | Sonnet |
| 12 | Examples | DONE | Haiku |
| 13 | Trivial files | DONE | Haiku |

---

## Next up (read this when resuming)

1. Review the generated documentation for depth/voice consistency, especially
   the very large CLI pages where some pages are intentionally more synthetic
   than line-adjacent.
2. Optionally run a Markdown link checker if one is added to the project.
3. Future pass: CI/CD, release tooling, `.github/workflows/`, and other
   infrastructure remain out of scope and still undocumented.

---

## Session log

### Session 1 — 2026-05-09 (Opus)

- Confirmed scope and approach with user (4 questions, all "Recommended" picks).
- Reset `docs/` (deleted partial leftover tree).
- Mapped full source tree (~140 source files in `libs/`, 14 example agents).
- Created this `book_diary.md`.
- *(in flight)* `instructions.md`, `CLAUDE.md` update, scaffold, prereqs,
  `ROADMAP.md`, `README.md`.

### Session 2 — 2026-05-09 (Codex, resumed after Claude token exhaustion)

- Read `AGENTS.md`, `instructions.md`, `book_diary.md`, and every existing file
  under `docs/`.
- Found the diary lagging the filesystem: `instructions.md`, `CLAUDE.md`,
  `docs/README.md`, `docs/ROADMAP.md`, and prerequisite chapters already
  existed; updated the stage table and "Next up" section accordingly.
- Began stage 4 directly (high-leverage core SDK work, no cheaper subagent
  delegation yet because these files are in the direct/Opus tier).
- Added:
  - `docs/libs/README.md`
  - `docs/libs/deepagents/README.md`
  - `docs/libs/deepagents/deepagents/README.md`
  - `docs/libs/deepagents/deepagents/graph.md`
  - `docs/libs/deepagents/deepagents/_models.md`
  - `docs/libs/deepagents/deepagents/_tools.md`
  - `docs/libs/deepagents/deepagents/backends/README.md`
  - `docs/libs/deepagents/deepagents/backends/protocol.md`
  - `docs/libs/deepagents/deepagents/middleware/README.md`
  - `docs/libs/deepagents/deepagents/middleware/subagents.md`
  - `docs/libs/deepagents/deepagents/middleware/filesystem.md`
- `graph.md` quotes the full `BASE_AGENT_PROMPT` and documents the current
  model/profile/subagent/middleware assembly flow, including async subagents,
  permissions, harness exclusions, and prompt caching.
- `protocol.md` documents all public result dataclasses, `BackendProtocol`,
  `SandboxBackendProtocol`, legacy compatibility bridges, and the timeout
  introspection helper.
- `subagents.md` documents the synchronous `task` tool, subagent specs,
  state isolation, structured-response serialization, and quotes the full task
  system prompt and tool description.
- `filesystem.md` documents filesystem permissions, backend-backed tools,
  multimodal reads, execute support gating, human/tool-result eviction, and
  quotes the filesystem/execution prompts plus all built-in tool descriptions.

What's left: continue stage 4 with `middleware/__init__.py` if needed for
exports, then finish any remaining direct-tier core docs before delegating
lower-tier leaf files.

### Session 3 — 2026-05-09 (Codex, completion pass)

- Treated the original Claude prompt as active scope and finished the full
  rewrite target.
- Used workers for lower-tier slices:
  - Remaining SDK backends.
  - Remaining SDK middleware.
  - Profiles, `_api`, and root helper files.
  - Example agents.
  - Peripheral CLI files/widgets/deploy/skills/MCP providers/integrations.
  - Non-SDK packages: `libs/acp`, `libs/evals`, `libs/partners`,
    `libs/repl`, and `libs/code`.
- Main thread handled/filled high-leverage CLI documentation:
  - `docs/libs/cli/README.md`
  - `docs/libs/cli/cli_architecture.md`
  - Core CLI docs for `main.py`, `app.py`, `server.py`,
    `server_manager.py`, `server_graph.py`, `remote_client.py`, `agent.py`,
    `command_registry.py`, `config.py`, `model_config.py`,
    `configurable_model.py`, `hooks.py`, `sessions.py`, `mcp_tools.py`,
    `non_interactive.py`, and key widgets.
- Added/confirmed docs across:
  - `docs/libs/deepagents/`
  - `docs/libs/cli/`
  - `docs/libs/acp/`
  - `docs/libs/evals/`
  - `docs/libs/partners/`
  - `docs/libs/repl/`
  - `docs/libs/code/`
  - `docs/examples/`
- Verification performed:
  - Every directory under `docs/` has a `README.md`.
  - Markdown fence balance check passed for all `.md` files.
  - Scoped source/doc coverage check passed: every in-scope package Python
    file under `libs/` has a corresponding docs page, excluding tests,
    virtualenv/vendor material, and explicitly peripheral CLI example/check
    import scripts.
  - Total docs/prereq/example files at completion check: 269.
- No tests were run because this was a documentation-only pass.

Known caveats:
- Some large CLI files have high-level block-group documentation rather than
  every private helper expanded to the same depth as the SDK core. The most
  important orchestration paths are covered, but a future polishing pass could
  deepen the biggest UI/config pages further.
- Infrastructure/CI/release docs remain out of scope by user request.
