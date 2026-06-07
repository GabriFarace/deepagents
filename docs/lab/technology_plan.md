# Technology Plan — a2ui-lab

Companion to `gen_ui_system_design.md`. This document is the **stack inventory**:
which libraries, protocols, and principles we'll use, and **why**.

Two services share the same baseline stack — only the third column changes by
component (mock backend vs. agent service vs. shared).

---

## 1. Baseline language stack (both services)

| Concern | Choice | Why |
|---|---|---|
| Language / runtime | Python 3.12+ | LangChain/LangGraph ecosystem is Python-native; FastAPI async. |
| Package manager | **uv** | Fast resolver, lockfiles, project workflow. Standard going forward. |
| Web framework | **FastAPI** | First-class async, Pydantic-driven schemas, OpenAPI for free, SSE-friendly. |
| Validation / models | **Pydantic v2** | De-facto standard; LangGraph's structured outputs depend on it. |
| ORM | **SQLAlchemy 2.x async** via **SQLModel** | SQLModel = SQLAlchemy + Pydantic in one class; clean separation between table-binding models and API schemas. |
| DB driver | **asyncpg** | Async Postgres driver; the right pairing for SQLAlchemy async. |
| Database | **PostgreSQL 16** | Required for `pgvector`, `jsonb`, robust transactional semantics. One DB per service (or one DB with two schemas — tradeoff is operational, not architectural). |
| Migrations | **None for the lab.** `setup_db.py` recreates the schema + seeds. | The user specified this. Add Alembic later if needed. |
| HTTP client | **httpx** (`AsyncClient`) | Standard async HTTP client. |
| Concurrency | `asyncio` everywhere. Windows: `WindowsSelectorEventLoopPolicy`. | Already in user's `setup_db.py` template. |
| Linting / formatting | **ruff** (lint + format) | Single tool, fast, sensible defaults. |
| Tests | **pytest**, `pytest-asyncio`, `pytest-postgresql` or testcontainers, `respx` (HTTP mocking for the agent's tools) | Standard test stack. |
| Logging | **structlog** with JSON output; `trace_id` in context | Trace-aware logs are essential for an agent system. |
| Env / config | **pydantic-settings** | Type-checked env loading; one `Settings` class per service. |

---

## 2. Agent service stack

| Concern | Choice | Why |
|---|---|---|
| Graph runtime | **langgraph** (latest) | The decided runtime. Provides nodes/edges, subgraphs, streaming, checkpointers, interrupts, `Command`. |
| LLM framework | **langchain-core** + provider packages | `init_chat_model("anthropic:claude-...")` for provider-agnostic code. Default **Anthropic** Claude. |
| LLM provider (default) | **langchain-anthropic** | Provider-agnostic via env (`LLM_PROVIDER=anthropic|openai|...`). |
| Structured outputs | LangChain's `.with_structured_output(...)` against Pydantic models | Used heavily by router (`IntentClassification`), planner, analytics (`AnalysisRequest`). |
| Checkpointer | **langgraph-checkpoint-postgres** (`AsyncPostgresSaver`) | Persists thread state; enables `interrupt()` / `Command(resume=...)` mechanics. |
| Vector store | **pgvector** via `pgvector-python` / `langchain-postgres` | Same Postgres → fewer moving parts. |
| Embeddings | Provider-agnostic via `langchain_<provider>` packages. Default: Anthropic if/when supported, else OpenAI `text-embedding-3-small` | Provider toggle via env. |
| AG-UI integration | **`ag-ui-langgraph`** if it cleanly maps our graph; otherwise hand-rolled stream adapter. | Decision per discussion: try the off-the-shelf bridge first. |
| AG-UI / A2UI / event / viewmodel Pydantic models | **Agent-internal**, in `agent_service/.../protocols/` | Not shared — only the agent produces/consumes them (see design §2.1). |
| Cross-service contracts | Shared **`contracts/`** package (capabilities, routes, semantic model) | Backend serves these, agent consumes them; one definition prevents drift (design §2.1). |
| Tool definitions | **Registry-generated `StructuredTool`s** over one generic HTTP executor | Typed tools built at startup from `/capabilities`; add endpoint + registry entry → no agent code change. See design §5.5. |
| Orchestration | **Deterministic router + `planner_subgraph`** for compound; supervisor-orchestrator deferred | Predictable/testable for v1; safe to promote later because security gates live inside workers. See design §5.4. |
| JSON-schema validation (composer outputs) | **jsonschema** | Validate emitted A2UI surfaces against `component_catalog.json` at the validator node. |
| CLI consumer | Plain Python + `httpx-sse` | Tiny script that POSTs an event and prints SSE events. |

---

## 3. Mock backend stack

| Concern | Choice | Why |
|---|---|---|
| Web framework | FastAPI | Same as agent service. |
| State machine | Plain Python (`enum.Enum` + a `TRANSITIONS` dict) | Not worth a library for the lab; in-code is auditable. |
| Analytics compiler | SQLAlchemy Core (`select`, `func`, parameters) compiled from the semantic DSL. No string concatenation. | Eliminates the entire SQLi class of bugs. |
| Docs RAG ingestion | LangChain text splitters + pgvector | Reused at `setup_db.py` time to ingest `docs_corpus/*.md`. |
| Idempotency | `command_executions` table keyed by `idempotency_key` | Replays prior result on duplicate execute calls. |

---

## 4. Shared package — `contracts/` (and what is NOT shared)

A shared package is justified **only** for types that both services exchange over
HTTP. That is the contract surface the **backend serves and the agent consumes**:
the capability registry, the route registry, and the analytics semantic model.
Sharing those Pydantic definitions guarantees the backend's serializer and the
agent's validator cannot silently drift. Everything else is agent-internal. See
design doc §2.1 for the full rationale.

**Shared — `contracts/` package** (installable, path-dependency of both services
via the `uv` workspace; no PyPI publish):

| Module | Contents | Served by |
|---|---|---|
| `contracts.capabilities` | `Capability`, `CapabilityType`, `InputSpec`, `SideEffect` — mirrors `/capabilities` payloads. | mock backend |
| `contracts.routes` | `RouteEntry` (path, label, description, requiredPermissions, params) — mirrors `/routes`. | mock backend |
| `contracts.semantic_model` | `SemanticModel`, `Dataset`, `Dimension`, `Measure` — mirrors `/semantic-model`. | mock backend |

**Agent-internal — `agent_service/src/agent_service/protocols/`** (NOT shared; the
mock backend never imports these):

| Module | Contents |
|---|---|
| `agui` | `AGUIEvent`, `RunStarted`, `RunFinished`, `RunInterrupted`, `TextMessageContent`, `ToolCallStart/End`, `StateDelta`, `CustomEvent`, … |
| `a2ui` | `A2UISurface`, `Component`, `Action`, plus catalog-aware variants per component type. |
| `events` | `ClientEvent`, `EventType`, `AppContext` — the inbound event envelope. |
| `viewmodels` | All `ViewModel` types the composer dispatches on (e.g., `OperationConfirmation`, `RagAnswer`, `ChartIntent`, …). |

---

## 5. Protocols (transport / contracts)

| Protocol | Where used | Notes |
|---|---|---|
| **HTTP/REST** | Client ↔ agent (POST events + SSE stream), agent ↔ mock backend (tools) | Stay vanilla REST. JSON request bodies, JSON SSE event payloads. |
| **Server-Sent Events (SSE)** | Agent → client streaming of AG-UI events | FastAPI `StreamingResponse(media_type="text/event-stream")`. One-way is sufficient. |
| **AG-UI** | The event vocabulary on the SSE stream | Pydantic models in `agent_service/.../protocols/agui.py`. Subset implemented in v1: `RUN_STARTED`, `RUN_FINISHED`, `RUN_INTERRUPTED`, `TEXT_MESSAGE_*`, `TOOL_CALL_START/END`, `STATE_DELTA`, `CUSTOM`. |
| **A2UI** | Component-tree payloads inside `CUSTOM` AG-UI events and inside `interrupt()` payloads | JSON validated against `component_catalog.json`. |
| **MCP** | Not in v1. Design tool definitions so they could be exposed via MCP later. | Out of scope. |
| **OpenAPI** | Both services publish their OpenAPI specs at `/openapi.json`. The capability registry cross-references endpoint paths. | Useful for debugging; the agent does NOT consume OpenAPI at runtime. |

---

## 6. Database concerns

| Concern | Choice |
|---|---|
| One DB or two | Two logical databases (`mock_db`, `agent_db`) running on the same Postgres instance. Cross-DB access prohibited — agent talks to mock backend only via HTTP. |
| Postgres version | 16+ |
| Extensions | `pgvector` (mock backend only, for the docs corpus). LangGraph checkpointer creates its own tables in `agent_db`. |
| Schema management | None. `setup_db.py` drops & recreates `public`, then `SQLModel.metadata.create_all`, then seeds. Same script in both services. |
| Seeds | Python module per service. Cases generated programmatically (~500), users/roles/permissions literal, docs chunked + embedded at setup time. |
| Connection pooling | SQLAlchemy default async pool (size 5–10). |

---

## 7. Security / authorization

| Concern | Lab choice | Production target |
|---|---|---|
| Client → agent auth | `X-User-Id` header trusted in lab mode | OAuth2/OIDC with token exchange |
| Agent → mock auth | Forwarded `X-User-Id` + a service token (`X-Service-Token`) | mTLS or on-behalf-of token exchange |
| Read scoping | Permission service applies `visible_case_filter` in route handler | Postgres RLS for defense-in-depth |
| Write enforcement | Re-check inside the execute transaction, after row lock (TOCTOU-safe) | Same, plus signed audit trail |
| Secrets | `.env` via pydantic-settings | Vault / cloud secrets manager |
| LLM prompt safety | Retrieved content is data; tool descriptions versioned; agent never sees raw tokens/credentials | Same + red-team eval suite |

---

## 8. Observability

| Concern | Choice |
|---|---|
| Structured logs | `structlog` JSON to stdout. Each service has a `bound_logger` with `trace_id`, `ai_session_id`, `thread_id`, `run_id`, `user_id`. |
| Tracing | Lightweight: `X-Trace-Id` propagation + structlog context. LangSmith optional (off by default; enabled via `LANGSMITH_API_KEY`). |
| Audit | `ai_audit_events` table in `agent_db`. One row per node fire / tool call / interrupt / capability execution. |
| Metrics | Not in v1. |

---

## 9. Streaming details

- The agent's `/ai/sessions/{ai_session_id}/events` endpoint:
  - Method: POST.
  - Request body: `ClientEvent` (see protocols).
  - Response: `text/event-stream`. Each line: `event: <name>\ndata: <json>\n\n`.
  - Closes when the graph reaches `RUN_FINISHED` or `RUN_INTERRUPTED`.
- `LangGraph.astream(..., stream_mode=["updates", "messages", "custom"], version="v2")` produces the chunks the adapter maps.
- Heartbeats: every 15s the adapter emits a `:ping` SSE comment to keep proxies/connections alive while the LLM is thinking.

---

## 10. Design principles (the rules we never break)

These are the principles that came out of both brainstorm sessions and are
documented here as the non-negotiable spine:

1. **The agent is never a security boundary.** Authority — RBAC, business rules,
   state-machine legality — lives in the mock backend. The agent operates *on
   behalf of* the user. A hallucinated request fails at the boundary.
2. **State first, UI second.** Every turn: compute state → build ViewModel →
   compose UI. Never let the LLM invent the data.
3. **Generative UI is not generative authority.** The agent emits structure from
   a *trusted catalog*, never executable code, never new endpoints.
4. **Pre-flight is advisory; the endpoint is authoritative.** Permission checks
   happen at execution time, transactionally, after locking the row. The
   pre-flight is for UX, not security.
5. **Use deterministic gates for security/correctness; LLM tools for judgment.**
   Authorization, confirmation, identity resolution are fixed graph edges. "Which
   chart to render" is an LLM-driven choice — and gets validated against the
   catalog before emission.
6. **Catalogs are the bridge.** Capability, A2UI components, routes, semantic
   model. These four registries are the deterministic contracts between
   agent reasoning and the rest of the system.
7. **Start-vs-resume is decided by thread state, not event type.** Chat and UI
   actions flow through the same endpoint. The `ai_session` row + LangGraph
   checkpointer together decide.
8. **All writes are idempotent and confirmable.** Idempotency keys + HITL
   `interrupt()` for state-changing operations.
9. **One enforcement point per concern.** RBAC in one place (mock backend
   permission service), composer validation in one place (catalog validator),
   audit in one place (`ai_audit_events`).
10. **No dynamic SQL from the LLM.** Analytics goes through a semantic DSL
    compiled by the backend. No exceptions in v1.
11. **Retrieved content is data, not instructions.** Docs, case notes, user
    comments — all treated as untrusted content. Tool descriptions are the only
    trusted instructions.
12. **Identity propagates end-to-end.** Every HTTP call from agent → backend
    forwards `X-User-Id`. The LLM never sees raw IDs/tokens.
13. **Audit everything load-bearing.** Every node fire, tool call, interrupt,
    capability execution → one audit row. Cheap insurance.
14. **Small catalog v1; expand under demand.** Don't preemptively design 50
    components. Start with the dozen that demonstrate the architecture.

---

## 11. What is *intentionally* excluded

- MCP server (design tools as if they will become MCP someday).
- LangGraph managed deployment.
- Postgres RLS (mock enforces in Python; design doc names RLS as the production
  path).
- OPA / external policy engines.
- Tier-C constrained text-to-SQL.
- Real frontend / Angular A2UI renderer (testing is pytest + a CLI consumer).
- Multi-tenant scoping.
- Rate limiting, mTLS, secrets manager, production observability stack.
- Alembic migrations.

---

## 12. Versions (initial pins, subject to update)

These are starting versions; lock exact ones at `uv lock` time.

```
python                          >= 3.12
fastapi                         ~= 0.115
uvicorn[standard]               ~= 0.30
pydantic                        ~= 2.8
pydantic-settings               ~= 2.5
sqlmodel                        ~= 0.0.22
sqlalchemy                      ~= 2.0
asyncpg                         ~= 0.29
pgvector                        ~= 0.3
httpx                           ~= 0.27
structlog                       ~= 24.4
ruff                            ~= 0.6
pytest                          ~= 8.3
pytest-asyncio                  ~= 0.24
respx                           ~= 0.21

langchain                       latest
langchain-core                  latest
langchain-anthropic             latest
langchain-openai                latest          # optional, for provider swap
langgraph                       latest
langgraph-checkpoint-postgres   latest
langchain-postgres              latest          # pgvector wrappers
jsonschema                      ~= 4.23

# attempted; fall back to hand-rolled adapter if friction
ag-ui-langgraph                 latest
```
