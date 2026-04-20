# Instructions: Building a `docs/` Directory and Learning Roadmap for Any Codebase

This document describes a repeatable process for generating a structured documentation tree and a learning roadmap from an existing codebase. It was derived from work done on the `deepagents` monorepo but is intentionally general.

---

## Goal

Produce two things:

1. **A `docs/` directory** — one `.md` file per source file, organized in the same directory hierarchy as the source. Each file explains *what* the source file does, not just *how* (no line-by-line comments; high-level purpose, key abstractions, and API surface).
2. **A `docs/ROADMAP.md`** — a dependency-ordered learning path through the codebase, from foundational concepts to advanced integrations.

Both artifacts are intended for a developer who is new to the project and wants to understand it systematically.

---

## Phase 1 — Reconnaissance

Before writing a single doc, build a complete mental model of the codebase.

### 1.1 Understand the repository shape

- What kind of project is it? (library, monorepo, service, CLI tool, etc.)
- What are the top-level directories? Which ones contain source code vs. config vs. tests?
- What package manager and build system are used?
- What is the entry point? (e.g., `main.py`, `__init__.py`, `index.ts`, `main.go`)

### 1.2 Read the existing orientation files first

In order of priority:
1. `README.md` at the root
2. Any `CONTRIBUTING.md`, `AGENTS.md`, `CLAUDE.md`, or `DEVELOPMENT.md`
3. `pyproject.toml` / `package.json` / `go.mod` — for declared dependencies and package names
4. Any existing architecture docs or ADRs

### 1.3 Map the dependency graph

For monorepos or multi-package repos, draw (or note) the dependency graph between packages:
- Which package is the core/SDK that others depend on?
- Which packages are adapters, wrappers, or optional plugins?
- Which packages are tools (CLI, TUI) that consume the core?

This graph becomes the spine of the roadmap.

### 1.4 Identify the key architectural concepts

Read the most important source files (entry points, core abstractions, primary public API). Aim to answer:
- What is the single most important function/class a new developer should understand first?
- What are the 3–5 central abstractions? (e.g., protocols, base classes, state types)
- What are the main execution flows? (e.g., request → middleware → handler → response)

---

## Phase 2 — Build the `docs/` Directory

### 2.1 Mirror the source hierarchy

Create a `docs/` directory. For every source file you document, create a corresponding `.md` file at the same relative path, with the `.md` extension replacing the source extension.

```
src/graph.py          →  docs/src/graph.md
libs/cli/app.py       →  docs/libs/cli/app.md
cmd/server/main.go    →  docs/cmd/server/main.md
```

### 2.2 For each source file, write a doc with this structure

```markdown
# `path/to/file.ext`

## High-Level Purpose

One paragraph. What problem does this file solve? What role does it play in the system?
Do NOT describe the code line-by-line. Describe what it IS and WHY it exists.

## [Key Abstractions / Constants / Types / Functions / Classes]

For each public symbol (class, function, type, constant):
- Name and signature
- What it does (1–3 sentences)
- Parameters table if there are many
- Return value
- Any non-obvious behavior or gotchas

## [Architecture Notes] (optional)

If the file has a notable internal structure (e.g., a class hierarchy, a two-pass algorithm,
a state machine), describe it here with a small ASCII diagram if helpful.

## [Usage Example] (optional)

A short code snippet showing how to use the main export, if it's non-obvious.

## [See Also] (optional)

Links to related docs files.
```

### 2.3 Write `README.md` files for directories

For every directory that contains multiple related files, write a `docs/<dir>/README.md` that:
- States the purpose of the directory
- Lists all files with a one-line description of each
- Describes any hierarchy or relationships between files (e.g., a class hierarchy diagram)
- Provides a "choosing between X and Y" guide if the directory contains alternatives (e.g., multiple backends, multiple adapters)

### 2.4 Prioritization — what to document and what to skip

**Always document:**
- Core abstractions (protocols, base classes, primary public API)
- Entry points (factory functions, `main()`, CLI entrypoints)
- Middleware / pipeline components
- Configuration and wiring files

**Document if non-trivial:**
- Tests — only if they reveal usage patterns not obvious from source
- Examples — a short summary of what each example demonstrates
- CI/CD workflows — what each workflow does and when it triggers
- Config files (`pyproject.toml`, `Makefile`, etc.) — only if they encode important conventions

**Skip or keep minimal:**
- Auto-generated files
- Trivial `__init__.py` re-exports (one sentence is enough)
- Lock files
- Files whose purpose is fully captured by their parent directory README

### 2.5 Write `docs/README.md` (the top-level index)

This is the navigation hub. It should contain:
- A one-paragraph summary of what the project does
- A visual tree of the `docs/` directory
- A "Quick Navigation" table organized by two axes:
  - **By layer** (top to bottom, e.g., UI → protocol → core → backends)
  - **By concept** (e.g., "how do I add a new tool?", "where is auth handled?")
- Links to `ROADMAP.md` and any other entry points

---

## Phase 3 — Build `docs/ROADMAP.md`

The roadmap is not a table of contents. It is a **learning path** — ordered so that each stage builds on the previous one, with no forward dependencies.

### 3.1 Structure the roadmap by stages

Each stage has:
- A **stage number and title**
- An estimated **time to complete**
- A **goal** sentence (what understanding you'll have after this stage)
- A table of **what to read** (doc file | source file | what you'll learn)
- A **key insight** or **key concepts** section — the 2–3 things that should "click" in this stage
- A **dependencies** note — what earlier stages must be understood first

### 3.2 Ordering principles

Order stages by:
1. **Conceptual dependency** — you can't understand middleware until you understand the protocol it operates on
2. **Breadth before depth** — understand what exists before diving into any one part
3. **Core before periphery** — SDK before CLI before ACP before evals
4. **Data types before logic** — type definitions before algorithms that use them

### 3.3 Include a prerequisites stage

If the project depends on external frameworks (LangGraph, React, Kubernetes, etc.), add a **Stage 1: Prerequisites** that lists external resources the reader should consume first, and the concepts they need to have internalized before proceeding.

### 3.4 Include a "how to use this roadmap" section

At the top, explain:
- Each stage is a self-contained learning unit
- "Read" links point to docs (high-level), "Source" links point to code (deep dive)
- Time estimates are for reading docs only; add 2–3x for exploring source code

### 3.5 End with an advanced topics stage

The final stage should cover:
- CI/CD and release process
- How to contribute
- Where to find things not covered in the roadmap (e.g., test patterns, example agents)

---

## Phase 4 — Update `CLAUDE.md` (or equivalent AI context file)

If the project has a `CLAUDE.md` or similar AI assistant context file, update it to:
- Point to `docs/ROADMAP.md` as the recommended entry point
- Add a `docs/` directory description to the repository structure section
- Add a "File Navigation Tips" table that maps common tasks to specific doc/source files

---

## Quality Checklist

Before considering the docs complete, verify:

- [ ] Every significant source file has a corresponding `.md` in `docs/`
- [ ] Every `docs/<dir>/` has a `README.md`
- [ ] `docs/README.md` links to every major section
- [ ] `docs/ROADMAP.md` has no forward dependencies (each stage only requires prior stages)
- [ ] Time estimates in the roadmap are realistic (read 200 lines of source ≈ 15–30 min)
- [ ] All links in docs files are relative and resolve correctly
- [ ] The "High-Level Purpose" section of each doc is written for a newcomer, not an author
- [ ] Architecture diagrams use plain ASCII (no Mermaid or external tools required)
- [ ] No doc file contains line-by-line code commentary — that belongs in source code comments

---

## Anti-patterns to Avoid

| Anti-pattern | Better approach |
|---|---|
| Documenting every line | Document purpose and API surface; trust the source code for implementation details |
| Flat docs structure | Mirror the source hierarchy so docs are discoverable alongside the code |
| Roadmap as a table of contents | Order by learning dependency, not by directory order |
| Skipping README files for directories | Every directory is a conceptual boundary; explain it |
| Writing for the author | Write for someone who has never seen the codebase before |
| Over-documenting trivial files | One sentence for a `__init__.py` re-export is enough |
| Under-documenting core abstractions | Central protocols, base classes, and public APIs deserve full treatment |
| Using Mermaid or PlantUML | ASCII diagrams are universally renderable and version-control-friendly |

---

## Worked Example: deepagents

The `docs/` directory in this repository is the reference implementation of this process:

- **187 `.md` files** covering ~130 source files + directory READMEs
- **Mirrored hierarchy:** `libs/deepagents/deepagents/graph.py` → `docs/libs/deepagents/deepagents/graph.md`
- **14-stage roadmap** in `docs/ROADMAP.md` from orientation (15 min) to advanced CI/CD
- **Two navigation axes** in `docs/README.md`: by layer and by concept
- **Directory READMEs** for every package directory, with class hierarchy diagrams and selection guides (e.g., `docs/libs/deepagents/deepagents/backends/README.md`)
- **Individual file docs** follow the structure: High-Level Purpose → public symbols with parameter tables → architecture notes → see also

Study `docs/libs/deepagents/deepagents/graph.md` as the canonical example of a well-documented core file, and `docs/libs/deepagents/deepagents/backends/README.md` as the canonical example of a well-written directory README.
