# Instructions: Building a `docs/` Tree, Roadmap, and Prerequisite Chapters for the `deepagents` Codebase

This document describes the **current** approach for documenting the `deepagents`
monorepo. It supersedes the prior version of `instructions.md` (which targeted
API-surface-only documentation). The current pass is deeper, CLI-focused, and
self-contained.

> **For Claude:** read `book_diary.md` first when resuming a session — it tracks
> per-stage progress so work can be resumed across context windows.

---

## Goal

Produce four artefacts:

1. **`docs/`** — a directory mirroring the source hierarchy of `libs/` (and a thin
   summary of `examples/`). Each documented source file gets a corresponding
   `.md` that explains the file **block-by-block** (every function and class
   gets a 1–3 paragraph explanation of its logic, inputs, side effects, and
   role in the larger flow).
2. **`docs/ROADMAP.md`** — a dependency-ordered learning path from prerequisites
   through the SDK core, CLI internals, and adapters. Every stage names the
   exact files to read in order.
3. **`docs/prerequisites/`** — self-contained chapters on **LangGraph** and
   **LangChain**, sourced from the official documentation (`python.langchain.com`,
   `langchain-ai.github.io/langgraph`) via `WebFetch`. The chapters cover only
   the concepts deepagents actually uses, but cover them deeply enough that the
   reader never has to leave the docs to follow the rest of the material.
4. **`docs/libs/cli/cli_architecture.md`** — a generic chapter on **how agent
   CLIs are built** (TUI loop, streaming, slash commands, MCP tools, sessions,
   HITL approvals), explaining where deepagents-cli's choices align with or
   diverge from comparable projects (Claude Code, Codex). The comparison stays
   conceptual — no scraping of closed-source repositories.

### Out of scope (for this pass)

- CI/CD workflows (`.github/workflows/`)
- Infrastructure, release tooling, action.yml
- `release-please-config.json`, `Makefile` internals beyond a sentence

These will be documented in a future pass.

---

## Granularity rules

For every source file:

- **List every public function and class** as its own block. Skip trivial
  one-line getters/setters and pure dataclass fields with no methods.
- **For each block, write 1–3 paragraphs** covering:
  - what the block computes / what it returns,
  - which arguments matter and what they constrain,
  - which state it mutates (LangGraph state, env vars, filesystem, etc.),
  - which other blocks call it and which it calls in turn,
  - any non-obvious behaviour (control flow inversions, hidden side effects,
    retry logic, dispatch tables, decorators that change the signature).
- **Do not transcribe code line-by-line** — that becomes stale and bloats tokens.
  But do walk the reader through any non-obvious *control flow* (e.g., a state
  machine, a two-pass algorithm, a generator wired to a callback).
- **Quote system prompts in full** when documenting middleware or factories that
  inject them. System prompts are part of the orchestration design and are
  invisible to a reader who only reads the code skeleton.
- **Quote tool descriptions** when documenting tool-bearing middleware.

---

## Directory structure

```
docs/
├── README.md                                ← top-level navigation index
├── ROADMAP.md                               ← 14-stage learning path
├── prerequisites/
│   ├── README.md
│   ├── langchain.md                         ← self-contained LangChain primer
│   └── langgraph.md                         ← self-contained LangGraph primer
├── libs/
│   ├── README.md                            ← package dependency graph
│   ├── deepagents/
│   │   ├── README.md
│   │   └── deepagents/
│   │       ├── README.md
│   │       ├── graph.md
│   │       ├── _models.md
│   │       ├── _tools.md
│   │       ├── backends/
│   │       │   ├── README.md
│   │       │   ├── protocol.md
│   │       │   ├── state.md
│   │       │   ├── filesystem.md
│   │       │   ├── local_shell.md
│   │       │   ├── sandbox.md
│   │       │   ├── store.md
│   │       │   ├── composite.md
│   │       │   ├── langsmith.md
│   │       │   └── utils.md
│   │       ├── middleware/
│   │       │   ├── README.md
│   │       │   ├── filesystem.md
│   │       │   ├── subagents.md
│   │       │   ├── async_subagents.md
│   │       │   ├── memory.md
│   │       │   ├── skills.md
│   │       │   ├── summarization.md
│   │       │   ├── patch_tool_calls.md
│   │       │   └── permissions.md
│   │       └── profiles/
│   │           ├── README.md
│   │           ├── harness/
│   │           └── provider/
│   ├── cli/
│   │   ├── README.md
│   │   ├── cli_architecture.md              ← generic CLI architecture chapter
│   │   └── deepagents_cli/
│   │       ├── README.md
│   │       ├── main.md
│   │       ├── app.md
│   │       ├── server.md / server_manager.md / server_graph.md
│   │       ├── remote_client.md
│   │       ├── command_registry.md
│   │       ├── sessions.md / hooks.md
│   │       ├── mcp_tools.md / mcp_auth.md / mcp_commands.md / mcp_trust.md
│   │       ├── widgets/                     ← per-widget docs
│   │       ├── deploy/
│   │       ├── skills/
│   │       ├── mcp_providers/
│   │       └── integrations/
│   ├── acp/
│   ├── evals/
│   ├── partners/
│   │   ├── daytona/
│   │   ├── modal/
│   │   ├── quickjs/
│   │   └── runloop/
│   └── repl/
└── examples/
    ├── README.md
    └── (one short doc per example agent)
```

---

## Per-file documentation template

```markdown
# `path/to/file.py`

> One-sentence summary. The role of this file in the system.

## Position in the system

How does this file fit in? Who imports it, what does it depend on, where does
it sit in the call graph (e.g., "called by `graph.py:create_deep_agent` after
backend initialisation"). Use a small ASCII diagram if helpful.

## Imports and module-level state

If the imports / module-level constants are worth flagging (private helpers,
re-exports, env-var reads at import time, frozen registries), call them out.
Otherwise skip this section.

## Functions and classes

For every function and every class:

### `Name(signature)`

What it does. What it returns. What it mutates. Who calls it. Non-obvious
behaviour. If it's a class, list its methods underneath as sub-blocks.

(Trivial helpers may be grouped: "The next four functions all just normalise
path strings; they all return `str` and ignore symlinks.")

## System prompts and tool descriptions (if any)

Quote them in full inside fenced code blocks. Annotate any placeholders.

## Flow walk-through (if the file orchestrates non-trivial control flow)

A numbered list of what happens in order, citing line numbers as
`file.py:123` so the reader can jump to source.

## Gotchas

Anything a reader is likely to misread or mis-modify.
```

---

## Roadmap structure (`docs/ROADMAP.md`)

The roadmap is a learning path, not a table of contents. Each stage:

- **Number, title, time estimate**
- **Goal sentence** ("after this stage you will understand X")
- **Prerequisites** (which earlier stages are required)
- **Files to read in order**, with one-line descriptions
- **Key insight** — the 2–3 things that should "click"

### Stage outline

1. **Orientation** — `README.md`, `AGENTS.md`, `CLAUDE.md`
2. **Prerequisites: LangChain** — `docs/prerequisites/langchain.md`
3. **Prerequisites: LangGraph** — `docs/prerequisites/langgraph.md`
4. **The deep-agent flow** — `docs/libs/deepagents/deepagents/graph.md`,
   `_models.md`, system prompt
5. **Backends** — protocol → all 8 implementations
6. **Middleware** — `subagents` → `filesystem` → `summarization` → others
7. **Profiles** — model-specific harness/provider profiles
8. **Generic CLI architecture** — `docs/libs/cli/cli_architecture.md`
9. **CLI bootstrap and server lifecycle** — `main.py`, `server.py`,
   `server_manager.py`, `server_graph.py`, `agent.py`
10. **CLI runtime: TUI, streaming, widgets** — `app.py`, `remote_client.py`,
    widgets
11. **CLI extensions** — slash commands, hooks, sessions, MCP, skills, deploy
12. **ACP adapter** — `libs/acp/`
13. **Sandboxes (partners)** — daytona / modal / quickjs / runloop
14. **Examples and evals** — examples + `libs/evals/`

---

## Prerequisites chapters

`docs/prerequisites/langchain.md` covers, at minimum:

- `BaseChatModel`, `init_chat_model`, model invocation styles
- Messages: `HumanMessage`, `AIMessage`, `ToolMessage`, `SystemMessage`
- Tool calling: `BaseTool`, the `@tool` decorator, `bind_tools`, structured tool
  responses
- Structured output (`with_structured_output`)
- Anthropic prompt caching as exposed in `langchain_anthropic`

`docs/prerequisites/langgraph.md` covers, at minimum:

- `StateGraph`, `START`, `END`, nodes and edges, conditional edges
- The reducer pattern; `MessagesState`; `add_messages`
- Compilation; `CompiledStateGraph`; `.invoke` / `.astream` / `.astream_events`
- Checkpointers and threads (persistence)
- Interrupts and resumption
- The agent loop (`create_react_agent` and the manual equivalent)
- `AgentMiddleware` (the new middleware API): hooks `before_model`,
  `after_model`, `wrap_model_call`, `wrap_tool_call`
- Sub-graphs and the `task` pattern

Both chapters are **sourced from the official docs** via WebFetch, distilled to
exactly the API surface deepagents uses. Where deepagents extends or specialises
a base concept (e.g., `AgentMiddleware`), the prereq chapter introduces the
base, and the SDK chapter documents the extension.

---

## Generic CLI architecture chapter

`docs/libs/cli/cli_architecture.md` covers:

- The **shell of an agent CLI** — entry script, config loading, session resume,
  arg parsing, event loop bootstrap.
- **Single-process vs. server-backed CLIs** — Claude Code is single-process;
  deepagents-cli spawns a separate `langgraph dev` server and talks to it
  over HTTP/SSE; this section explains why and what each choice buys you.
- **TUI architectures** — Textual (deepagents-cli), Ink (Claude Code, Codex),
  raw ANSI loops. Trade-offs in input handling, redraws, and accessibility.
- **Streaming token displays** — SSE / chunked HTTP; backpressure; partial
  rendering; cancellation.
- **Slash command registries** — how a flat registry plus auto-complete plus
  argument parsing scales to dozens of commands.
- **Tool-call rendering and approval** — pretty-printing tool inputs, diff
  rendering, HITL approval gates.
- **MCP integration** — tool discovery, auth, transports.
- **Sessions and persistence** — session files, message replay, thread IDs.
- **Hooks** — user-extensible callbacks at lifecycle points.

The chapter draws comparisons with Claude Code, Codex, and other agent CLIs
based on **public documentation only** — no proprietary code is reproduced.

---

## Model and cost strategy

This documentation is being produced by `claude-opus-4-7`, but the user has
asked us to **reserve Opus for the most important work**:

| Tier | Model | Used for |
|---|---|---|
| **Tier 1 (Opus 4.7)** | direct | `book_diary.md`, `instructions.md`, `CLAUDE.md`, `docs/ROADMAP.md`, `docs/README.md`, prerequisites chapters, generic CLI architecture chapter, deepagents `graph.py` / `_models.py` / `backends/protocol.py` / core middleware (`subagents`, `filesystem`), CLI core (`main`, `app`, `server*`, `remote_client`, `command_registry`, `sessions`, `hooks`, key widgets) |
| **Tier 2 (Sonnet 4.6)** | via `Agent` subagents | remaining SDK middleware, remaining backends, profiles, peripheral CLI files, `libs/acp/`, `libs/evals/`, `libs/partners/`, `libs/repl/` |
| **Tier 3 (Haiku 4.5)** | via `Agent` subagents | example-agent docs, trivial `__init__` re-exports, `_version.py` files |

Subagents must be given the per-file template, the path to the source file, the
path where the doc should be written, and a self-contained mini-context (one
paragraph on where the file fits in the system).

---

## Resumability

Every working session **must**:

1. Read `book_diary.md` at the start.
2. Update the per-stage status table after every meaningful step.
3. Append to the session log on session end with a "what's left" note.
4. Never delete the diary entries — they are the audit trail.

If a session ends mid-file, leave the partially written doc in place but mark
it `<!-- WIP: continue from §X -->` at the top so the next session can pick up.

---

## Quality checklist

Before considering any stage complete, verify:

- [ ] Every source file in scope has a corresponding `.md` (or is explicitly
      called out as "skipped — see parent README" for trivial cases).
- [ ] Every directory has a `README.md`.
- [ ] System prompts and tool descriptions are quoted in full.
- [ ] Block-level coverage: every public function and class has its own block.
- [ ] No forward dependencies in the roadmap.
- [ ] All internal links resolve (`./relative/path.md`).
- [ ] Prerequisites chapters are self-contained — a reader who has never
      touched LangChain or LangGraph can follow the rest of the docs.
- [ ] CLI architecture chapter cites public sources for any comparison made.
- [ ] `book_diary.md` reflects current state.

---

## Anti-patterns

| Anti-pattern | Better approach |
|---|---|
| Documenting individual lines | Document blocks (function/class), explain the logic of each block |
| Skipping system prompts | Quote them in full — they're part of the design |
| Burning Opus tokens on trivial re-exports | Delegate to Haiku via subagent |
| Letting the diary go stale | Update after every step, even small ones |
| Comparing CLIs by guessing internals | Cite public docs; admit uncertainty |
| Inline LangGraph explanations in every file doc | Centralise in the prereq chapter, link from file docs |
| Mermaid / external-tool diagrams | Plain ASCII — universally renderable |
| Asking the user to re-confirm settled scope | Read `book_diary.md`'s "Decisions log" first |
