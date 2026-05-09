# AGENTS.md — deepagents Codebase Context

This file gives Codex the context it needs to work effectively in this repository.

---

## What This Repository Is

**deepagents** is an open-source, MIT-licensed, provider-agnostic AI agent
framework built on [LangGraph](https://github.com/langchain-ai/langgraph). It is
structured as a Python monorepo managed with `uv`.

**Inspired by Codex.** It aims to be a general-purpose, batteries-included
agent harness.

**Current version:** `0.5.0a2` (alpha)

---

## Documentation effort in progress

The owner is rebuilding `docs/` from scratch with a deeper, block-level approach.
**Before doing any documentation work, you MUST read:**

1. **`book_diary.md`** — at the repo root. Persistent progress log. Tells you
   where the previous session left off, what's done, and what's next.
2. **`instructions.md`** — at the repo root. The current playbook for this
   documentation pass (block-level granularity, CLI focus, prereq chapters,
   model-cost strategy).

Do **not** consult the prior `docs/` contents — that tree was deleted and is
being rewritten under different rules.

### Documentation cost discipline

The owner has asked us to reserve **Opus 4.7** for high-leverage work and use
cheaper models for the rest. As a default policy:

| Tier | Model | Use for |
|---|---|---|
| Tier 1 — direct | `Codex-opus-4-7` | Roadmap, prerequisites chapters, generic CLI architecture chapter, deepagents core (`graph.py`, `_models.py`, `backends/protocol.py`, `middleware/subagents.py`, `middleware/filesystem.py`), CLI core (`main.py`, `app.py`, `server*`, `remote_client.py`, `command_registry.py`, `sessions.py`, `hooks.py`, key widgets), system-prompt audits, anything that touches orchestration logic. |
| Tier 2 — Agent subagent | `Codex-sonnet-4-6` | Remaining middleware, remaining backends, profiles, peripheral CLI files (helpers, configs, most widgets), `libs/acp/`, `libs/evals/`, `libs/partners/`, `libs/repl/`. |
| Tier 3 — Agent subagent | `Codex-haiku-4-5-20251001` | `examples/` agent docs, trivial `__init__.py` re-exports, `_version.py` files, anything that's effectively boilerplate. |

When delegating to a subagent, give it: the per-file template from
`instructions.md`, the path to the source file, the destination doc path, and a
one-paragraph mini-context on where the file fits.

### Out-of-scope (this pass)

CI/CD, infrastructure, `.github/workflows/`, release tooling, and `action.yml`
are explicitly out of scope. Do not produce docs for them in this pass.

---

## Repository Structure

```
deepagents/
├── libs/
│   ├── deepagents/       # Core SDK — create_deep_agent(), backends, middleware
│   ├── cli/              # deepagents-cli — Textual TUI + LangGraph server
│   ├── acp/              # ACP adapter — exposes agent as ACP server
│   ├── evals/            # Evaluation suite + Harbor benchmarks
│   ├── code/             # placeholder package
│   ├── repl/             # langchain-repl interpreter + middleware
│   └── partners/
│       ├── daytona/      # Daytona cloud sandbox
│       ├── modal/        # Modal serverless sandbox
│       ├── quickjs/      # QuickJS in-process JS sandbox
│       └── runloop/      # Runloop cloud sandbox
├── examples/             # 14 example agents (deep_research, content-builder, …)
├── docs/                 # Documentation tree (rebuilt — see book_diary.md)
├── book_diary.md         # Progress log for the doc rewrite (read first!)
├── instructions.md       # Playbook for the doc rewrite (read second!)
├── AGENTS.md             # Development conventions
└── AGENTS.md             # (this file)
```

---

## Package Dependency Graph

```
deepagents (SDK)
    ↑
    ├── deepagents-cli          (wraps SDK with Textual TUI + langgraph dev server)
    ├── deepagents-acp          (wraps SDK as ACP server)
    └── deepagents-evals        (tests SDK via CLI + Harbor)

partners/* (daytona, modal, quickjs, runloop)
    └── implement BackendProtocol from deepagents SDK
```

---

## Key Architectural Concepts (high level — implementation details live in `docs/`)

1. **`create_deep_agent()`** — single SDK entry point. Returns a LangGraph
   `CompiledStateGraph`. Default config produces a working coding assistant.
2. **`BackendProtocol`** — uniform contract for file/shell access. Multiple
   implementations (state, filesystem, local-shell, sandboxes, store,
   composite).
3. **`AgentMiddleware` stack** — hooks (`wrap_model_call`, `wrap_tool_call`,
   `before_model`, `after_model`) for tool injection, planning, summarisation,
   memory, HITL gates.
4. **CLI architecture** — the CLI **does not embed the SDK directly**; it
   spawns `langgraph dev` as a subprocess and talks to it via HTTP/SSE. Enables
   persistence, hot-reload, ACP mode, and remote subagents.
5. **ACP adapter** — bridges the ACP wire protocol to a LangGraph graph.

For specifics (exact middleware ordering, prompt content, default models,
state schema, etc.), consult `docs/` — those details change frequently and
should not be embedded here.

---

## Development Workflow

### Prerequisites

```bash
pip install uv
```

### Per-package commands

```bash
make test      # Run tests with pytest
make lint      # Run ruff linter
make format    # Run ruff formatter
make check     # lint + format check (CI mode)
```

### Running the CLI

```bash
cd libs/cli
uv run deepagents
```

### Running tests

```bash
cd libs/deepagents && uv run pytest
cd libs/cli && uv run pytest tests/unit_tests/
```

---

## Code Conventions

See [AGENTS.md](AGENTS.md) for full conventions. Key points:

- **Commit messages:** Conventional Commits (`feat:`, `fix:`, `docs:`).
- **PR titles:** Same format.
- **Linting/formatting:** `ruff`.
- **Python version:** `>=3.11` for most packages; `3.14` required for ACP.
- **Version sync:** `pyproject.toml` version must match `_version.py`.
- **No direct commits to `main`** — enforced by pre-commit hook.

---

## Key External Dependencies

| Dependency | Role |
|---|---|
| `langgraph` | Agent graph execution, state, checkpointing |
| `langchain` | LLM abstractions, tools, middleware base classes |
| `langchain_anthropic` | Anthropic/Codex model integration + prompt caching |
| `textual` | TUI framework for the CLI |
| `langgraph-sdk` | Client for communicating with LangGraph server |
| `uv` | Fast Python package manager |
| `ruff` | Linter + formatter |
| `pytest` | Test framework |
| `modal`, `daytona-sdk`, `runloop-api-client` | Cloud sandbox providers |
