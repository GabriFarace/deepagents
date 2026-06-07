# Generative UI System Design — a2ui-lab

This document is the implementation blueprint for the laboratory. It is the
synthesis of the two brainstorming sessions
(`claude_brainstorming.md`, `chatgpt_brainstorming.md`), the comparison report
(`comparison_report.md`), and the design decisions agreed with the team.

Companion: `technology_plan.md` (stack + libraries + protocols + principles).

---

## 0. Glossary

| Term | Meaning |
|------|---------|
| **Mock backend** | The Python FastAPI service that stands in for the real Java/Spring + Oracle system. Owns business logic, RBAC, the case state machine. |
| **Agent service** | The Python FastAPI service that hosts the LangGraph application and talks to the Mock backend via HTTP tools. |
| **AG-UI** | The bidirectional **event** protocol between client and agent service. Carries text/tool/state/run-lifecycle events. |
| **A2UI** | The declarative **UI payload** format. JSON describing components from a *trusted catalog*. Validated by JSON Schema. |
| **Capability** | A backend operation the agent is allowed to invoke (read or write). Lives in the **Capability Registry**. |
| **Catalog** | A whitelist. We have four: capability, A2UI components, frontend routes, analytics semantic model. |
| **ViewModel** | The structured intermediate produced by domain nodes, consumed by the UI composer. Domain-neutral wrt rendering. |
| **A2UI surface** | A complete renderable component tree emitted by the composer (ID'd by `surface_id`). |
| **Thread** | A LangGraph persistence unit, identified by `thread_id`, backed by the Postgres checkpointer. One per AI session. |
| **AI session** | Application-level container around a thread. Identified by `ai_session_id`, tracked in the `ai_session` table. |
| **HITL** | Human-in-the-loop. The graph pauses at `interrupt()`, the user responds, the graph resumes via `Command(resume=…)`. |

---

## 1. Goals and non-goals

### Goals
- Demonstrate the full **chat → planning → tool → HITL → execute → render** loop using AG-UI + A2UI + LangGraph.
- Cover **all 11 flows (a–k)** end-to-end.
- Enforce the architectural principle: **the agent is never a security boundary**. The mock backend owns identity, RBAC, and business state.
- Provide a clean foundation for replacing the mock backend with the real Java/Spring/Oracle later (interface-compatible HTTP contracts).

### Non-goals
- A real frontend (Angular A2UI renderer). Testing is via pytest/curl + a tiny CLI consumer.
- Postgres RLS, OPA, MCP, LangGraph managed deployment, Tier-C text-to-SQL.
- Production hardening (rate limits, mTLS, secrets manager, …).
- Multi-tenant features.

---

## 2. Top-level architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              CLIENT (out of scope)                          │
│  cli_consumer.py prints AG-UI events; pytest httpx.AsyncClient drives runs  │
└──────────────────────────────────┬──────────────────────────────────────────┘
                                   │ HTTP POST /ai/sessions/{id}/events
                                   │ (SSE response stream)
                                   ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                            AGENT SERVICE (Python)                           │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  FastAPI gateway                                                       │  │
│  │   ├─ /ai/sessions/{id}/events  (POST, SSE response)                    │  │
│  │   ├─ /artifacts/{ref}          (GET, dataRef pattern)                  │  │
│  │   └─ /health                                                           │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  Event router                                                          │  │
│  │   - loads ai_session, checks status                                    │  │
│  │   - START vs RESUME decision                                           │  │
│  │   - client_event_id idempotency check                                  │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  LangGraph application (compiled graph)                                │  │
│  │   nodes: ingest, hydrate_context, classify_intent, route, …           │  │
│  │   subgraphs: chat_qa, docs_rag, navigation, analytics, operation,     │  │
│  │              clarification, case_summary, permission_explanation, …    │  │
│  │   composer (deterministic templates + LLM for analytics), validator   │  │
│  │   checkpointer: AsyncPostgresSaver                                     │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  Stream adapter (LangGraph chunks ↔ AG-UI events)                      │  │
│  │   uses ag-ui-langgraph if viable, else hand-rolled mapper             │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  Tool layer (registry-generated typed tools over a generic executor)   │  │
│  │   tools built at startup from /capabilities; forwards X-User-Id/Trace  │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────┬──────────────────────────────────────────┘
                                   │ HTTP (httpx.AsyncClient)
                                   │ Headers: X-User-Id, X-Trace-Id
                                   ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                          MOCK BACKEND (Python)                              │
│  FastAPI + SQLModel + SQLAlchemy async                                      │
│  ├─ /cases, /cases/{id}, /cases/{id}/transitions/{name}/{preflight,execute} │
│  ├─ /cases/{id}/activities, /cases/{id}/payments, /cases/{id}/mails …       │
│  ├─ /analytics/cases (semantic DSL endpoint)                                │
│  ├─ /capabilities                  (capability registry, read-only)         │
│  ├─ /routes                        (frontend route registry, read-only)     │
│  ├─ /docs/search                   (RAG over docs corpus, pgvector)         │
│  ├─ /users/me/permissions          (advisory summary)                       │
│  └─ /health                                                                  │
│  Permission service: users × roles × role_permissions × work_basket /        │
│                      category / product scopes                              │
└──────────────────────────────────┬──────────────────────────────────────────┘
                                   │ asyncpg
                                   ▼
                              PostgreSQL
              ├─ Mock domain schema (case, user, role, …)
              ├─ pgvector index for the docs corpus
              └─ LangGraph checkpoint schema (separate DB or same)
```

Two services, two Postgres databases (or two schemas in one Postgres if simpler).
Repo layout:

```
a2ui-lab/
├─ mock_backend/
│  ├─ pyproject.toml
│  ├─ src/mock_backend/
│  │   ├─ main.py
│  │   ├─ db_connections.py
│  │   ├─ setup_db.py
│  │   ├─ models/          # SQLModel classes
│  │   ├─ schemas/         # Pydantic API schemas (separate from ORM)
│  │   ├─ routers/         # FastAPI routers per resource
│  │   ├─ services/        # permission_service, state_machine, analytics
│  │   ├─ seeds/           # programmatic + literal seed objects
│  │   ├─ docs_corpus/     # markdown files ingested by setup_db.py
│  │   └─ catalogs/        # capability_registry.json, route_registry.json, semantic_model.yaml
│  └─ tests/
├─ agent_service/
│  ├─ pyproject.toml
│  ├─ src/agent_service/
│  │   ├─ main.py
│  │   ├─ db_connections.py
│  │   ├─ setup_db.py            # creates ai_session, ai_audit_events, client_event_dedup, artifact tables
│  │   ├─ event_router.py
│  │   ├─ stream_adapter.py
│  │   ├─ graph/
│  │   │   ├─ state.py
│  │   │   ├─ build.py           # compile_graph()
│  │   │   ├─ nodes/             # deterministic nodes
│  │   │   └─ subgraphs/         # one file per specialist
│  │   ├─ protocols/             # AGENT-INTERNAL wire models (not shared)
│  │   │   ├─ agui.py            #   AG-UI event Pydantic models (agent → client)
│  │   │   ├─ a2ui.py            #   A2UI surface + component models (agent → client)
│  │   │   ├─ events.py          #   ClientEvent / EventType / AppContext (client → agent)
│  │   │   └─ viewmodels.py      #   ViewModel types the composer dispatches on
│  │   ├─ tools/                 # generic backend executor + registry-generated typed tools
│  │   ├─ composer/
│  │   │   ├─ templates.py       # ViewModel → A2UI deterministic mappers
│  │   │   ├─ llm_composer.py    # for ChartIntent / open analytics
│  │   │   └─ validator.py       # validates against component_catalog.json
│  │   ├─ catalogs/              # component_catalog.json (A2UI) authored here; registries fetched at runtime (§4.4)
│  │   ├─ models/                # SQLModel: AiSession, AiAuditEvent, ClientEventDedup, Artifact
│  │   └─ rag/                   # pgvector store wrapper
│  ├─ scripts/
│  │   └─ cli_consumer.py
│  └─ tests/
├─ contracts/                    # SHARED package: ONLY cross-service contracts (backend serves, agent consumes)
│  ├─ pyproject.toml             # installable; path-dependency of both services
│  └─ src/contracts/
│      ├─ capabilities.py        # Capability / CapabilityType / InputSpec / SideEffect (mirrors /capabilities)
│      ├─ routes.py              # RouteEntry (mirrors /routes)
│      └─ semantic_model.py      # SemanticModel / Dataset / Dimension / Measure (mirrors /semantic-model)
├─ comparison_report.md
├─ gen_ui_system_design.md       (this file)
├─ technology_plan.md
├─ DDL.sql                       (reference — Oracle original)
├─ data.sql                      (reference — Oracle original)
└─ README.md
```

### 2.1 Why `contracts/` is small (and why most models live inside the agent)

A shared package is only justified for types that **both services touch**. The
only such surface is the **contract the backend serves and the agent consumes**:
the capability registry, the route registry, and the analytics semantic model.
Sharing *those* Pydantic definitions guarantees the backend's serializer and the
agent's validator cannot silently drift — that is the entire payoff, so
`contracts/` contains exactly those three modules and nothing else.

Everything else is **agent-internal** and therefore lives inside
`agent_service/src/agent_service/protocols/`:

| Module | Producer | Consumer | Shared? |
|---|---|---|---|
| `contracts.capabilities` | mock backend (`/capabilities`) | agent | **yes → `contracts/`** |
| `contracts.routes` | mock backend (`/routes`) | agent | **yes → `contracts/`** |
| `contracts.semantic_model` | mock backend (`/semantic-model`) | agent | **yes → `contracts/`** |
| `agui` (event models) | agent | client | no → agent-internal |
| `a2ui` (surface/components) | agent composer | client | no → agent-internal |
| `events` (ClientEvent) | client | agent | no → agent-internal |
| `viewmodels` | agent (internal) | agent (internal) | no → agent-internal |

The mock backend never imports `agui`, `a2ui`, `events`, or `viewmodels` — it has
no reason to. Both services declare `contracts` as a path dependency in their
`pyproject.toml` (`uv` workspace). This keeps the shared surface minimal and
honest: if a model is not exchanged between the two services over HTTP, it does
not belong in `contracts/`.

---

## 3. Cross-cutting concepts

### 3.1 Identity propagation
- Client sends `X-User-Id: <user_id>` on every event.
- Agent service trusts it (lab mode), reads it into the runtime context (NOT into the LLM prompt directly).
- Every outbound HTTP call from the agent to the mock backend forwards `X-User-Id` so the mock backend can enforce RBAC as if a real user were calling.
- The LLM sees only the user's display name + a short summary of their permissions (computed via `/users/me/permissions`). It never sees raw IDs/tokens.

**Why we need it.** This is the mechanical consequence of the #1 principle "the
agent is never a security boundary." The backend must enforce RBAC *as the real
user*. There are only three ways to make that happen: (a) forward the user's
identity so the backend scopes everything — what we chose; (b) give the agent
god-mode credentials and let it decide what the user sees — forbidden, that is
exactly the anti-pattern; (c) re-implement RBAC inside the agent — duplicates
authority in a place that cannot be trusted. So identity propagation is not a
"feature," it is the thing that lets every security guarantee actually hold: a
hallucinated or over-broad request simply fails at the boundary, the same way a
malicious frontend would. The secondary reason is keeping raw IDs/tokens *out of
the LLM prompt* — the model sees a name + permission summary, the HTTP layer
carries the identity.

### 3.2 Trace propagation
- Each inbound event gets a `trace_id` (UUID4) if the client didn't send one.
- Propagated agent → mock backend via `X-Trace-Id` header.
- Logged as a structlog context var in both services.
- Stored in every `ai_audit_events` row.

**Why we need it.** One user message fans out into ~10 node fires plus a handful
of HTTP calls, spread across two services and an LLM. When something goes wrong
("why did it deny that?") you need to reconstruct the *causal chain* across both
process boundaries. A shared trace id stitches the agent's structlog lines, the
backend's structlog lines, and the audit rows into one timeline. Without it,
correlating "this backend 403" to "that agent decision" is manual guesswork. It
is nearly free to add and you will use it constantly while debugging.

### 3.3 The `ai_session` table (agent service DB)

```python
class AiSession(SQLModel, table=True):
    ai_session_id: UUID = Field(primary_key=True)
    thread_id: str                                      # 1:1 with ai_session_id
    user_id: str
    status: SessionStatus                               # IDLE | RUNNING | WAITING_FOR_USER | FAILED
    active_run_id: str | None
    pending_interrupt_id: str | None
    pending_surface_id: str | None
    pending_action_ids: list[str] = Field(sa_column=Column(JSON))
    pending_resume_schema: dict | None = Field(sa_column=Column(JSON))
    last_activity_at: datetime
    created_at: datetime
```

**Rule** (start vs resume):

```python
if session.status == "WAITING_FOR_USER" and matches_pending(session, event):
    resume_payload = build_resume_payload(session, event)
    return stream(graph.astream(Command(resume=resume_payload), config={...thread_id}))
elif session.status == "WAITING_FOR_USER":
    return small_response("There is a pending confirmation. Approve, reject, or cancel it first.")
elif session.status == "RUNNING":
    return small_response("An AI operation is already running.")
else:
    session.status = "RUNNING"
    return stream(graph.astream({"incoming_event": ..., "app_context": ...}, config={...thread_id}))
```

`matches_pending` checks: `event.surface_id == session.pending_surface_id` AND `event.action_id in session.pending_action_ids` AND payload validates against `session.pending_resume_schema`.

**Why we need it.** This is the routing control-plane. The LangGraph
checkpointer holds the *graph* state, but to route an inbound event you need fast
answers to app-level questions the checkpointer does not surface cleanly: "is
this session currently parked at an interrupt, and if so, which surface/actions
is it waiting for, and what is the resume schema?" You could dig that out of
checkpoint internals, but it is awkward and version-fragile. The table is a
cheap, queryable mirror that also lets you (a) validate an incoming UI action
actually matches the pending interrupt, and (b) enforce single-active-run-per-
session. **Authority split:** the checkpointer remains the source of truth for
graph state; the `ai_session` row is a fast lookup + validation layer that must
never contradict it (it is updated in the same flow that drives the graph).

### 3.4 Idempotency — `client_event_dedup`

```python
class ClientEventDedup(SQLModel, table=True):
    client_event_id: str = Field(primary_key=True)         # supplied by client
    ai_session_id: UUID
    response_snapshot: dict = Field(sa_column=Column(JSON))  # final AG-UI events, or stream replay log
    created_at: datetime
```

On every event: check by `client_event_id`. If present, replay the snapshot. Else process and persist.

**Why we need it.** Agents and networks retry. SSE connections drop mid-stream;
a client that reconnects may resend the same event. If a resend starts a *second*
run, or worse *double-resumes* an interrupt, you get a duplicated state
transition (case approved twice, notification sent twice). Because this system
performs writes, dedup at the entry point is the cheap insurance against the
single most painful class of agent bug. Note this is the *event-level* dedup;
there is a separate `Idempotency-Key` on the backend execute endpoint (§4.3) —
belt and suspenders, because the retry can happen at either hop.

### 3.5 Audit — `ai_audit_events`

```python
class AiAuditEvent(SQLModel, table=True):
    id: UUID = Field(primary_key=True)
    ai_session_id: UUID
    thread_id: str
    run_id: str | None
    trace_id: str
    user_id: str
    kind: AuditKind                                          # NODE_FIRED | TOOL_CALLED | INTERRUPT_ISSUED | INTERRUPT_RESUMED | CAPABILITY_EXECUTED | ERROR
    name: str                                                # node name, tool name, capability name
    input_summary: dict | None = Field(sa_column=Column(JSON))
    output_summary: dict | None = Field(sa_column=Column(JSON))
    timestamp: datetime
```

Written from a graph callback + tool wrapper + composer. Volume can grow — fine for the lab.

**Why we need it.** The agent is nondeterministic. When a run does something
surprising you need a record of *what it actually did* — which nodes fired,
which tools it called with what arguments, which capability executed, what the
preflight decided. Three uses: debugging during development, the compliance
story the real enterprise system will demand, and offline evaluation of runs
later. Writing one row per load-bearing step is cheap now and impossible to
reconstruct retroactively.

### 3.6 Artifact store — `dataRef`

```python
class Artifact(SQLModel, table=True):
    ref: str = Field(primary_key=True)                       # e.g. result://run-456/table-1
    ai_session_id: UUID
    content_type: str                                        # application/json typical
    payload: dict = Field(sa_column=Column(JSON))
    created_at: datetime
```

`GET /artifacts/{ref}` returns the payload; permission check = "is the requester the owner of the session that produced it". The agent's analytics specialist stores a row when results are too large to inline (>~10 KB heuristic).

**Why we need it.** A bottleneck analysis might return thousands of rows. Three
things break if you inline that everywhere: the SSE stream bloats; the data
round-trips through the checkpointed graph state (which then reloads on every
resume — slow and expensive); and the LLM context can get polluted if the blob
passes through a node. The pattern stores the dataset once, puts a `result://…`
reference in the A2UI payload, and lets the client fetch it through a
permission-checked endpoint. Small results still inline; the ~10 KB threshold
decides. This matters more than it looks once flow (d) gets real.

### 3.7 Start-vs-resume in one sentence
**Whether a request starts a new run or resumes is decided by the `ai_session.status`, not by `eventType`.** Chat messages and UI actions both flow through the same endpoint and the same router.

**Why we need it.** This is the linchpin of the whole runtime, and it is listed
as a cross-cutting concern because *every* inbound event hits it first. The
crucial insight — start-vs-resume is decided by thread/session **state**, not by
whether the event was a chat message or a button click — is what lets chat and
UI interactions collapse into one endpoint and one mental model. It is called
out explicitly so the next implementer does not build two parallel systems for
"chat" and "UI."

---

## 4. Mock backend design

### 4.1 Domain tables (Postgres / SQLModel)

Lowercased, snake-cased adaptations of the Oracle DDL — Postgres-native types
(no `NUMBER(20,0)`, use `BigInteger`; no `VARCHAR2`, use `String`; no `CLOB`, use `Text`).

Core tables (kept conceptually 1:1 with the DDL):

- `case_record` (renamed from `EINV_CASE` to avoid SQL keyword clash with `case`). PK: `case_id`.
  - All columns documented in DDL.sql carry over.
  - **State machine**: `status` constrained to a Python `CaseStatus` enum (`OPEN, TO_ENRICH, MANUAL_REVIEW, APPROVED, REJECTED, CANCELLED, CLOSED`).
- `case_activity` — children of cases. Status enum: `TODO, OPEN, CLOSED`.
- `case_payment`.
- `case_enif_detail` (1:1 with case).
- `mail`.
- `swift_message`.
- `unifi_message`.
- `app_user` (renamed from `EINV_USER`).

Permission overlay (new for the mock — not in the original DDL):

- `role` (`role_id`, `name`, `description`)
- `user_role` (`user_id`, `role_id`) — many-to-many
- `permission` (`permission_id`, `code`) — `CASE_READ`, `CASE_WRITE`, `CASE_APPROVE`, `CASE_REJECT`, `CASE_ASSIGN`, `BASKET_ADMIN`, `ANALYTICS_READ`, …
- `role_permission` (`role_id`, `permission_id`)
- `permission_scope` (`role_id`, `scope_type`, `scope_value`) — scope_type ∈ `{WORK_BASKET, CATEGORY, PRODUCT}`. A role with `(WORK_BASKET, "ENIF_ALERT_L1")` rows can act only on cases whose `work_basket_id` is in that list. Empty scope = unrestricted.

The permission service exposes:

```python
def can_perform(user_id: str, action: PermissionCode, case: CaseRecord | None) -> Decision: ...
def visible_case_filter(user_id: str) -> SQLAlchemy filter clause: ...
def permissions_summary(user_id: str) -> dict: ...
```

Every list/get endpoint applies `visible_case_filter` as a `WHERE` clause. Every write endpoint calls `can_perform` *inside the transaction*, after locking the case row.

### 4.2 State machine

Defined in `services/state_machine.py` as a Python dict, plus a `transition` table is **not** used (in-code is enough for the lab):

```python
TRANSITIONS: dict[CaseStatus, list[Transition]] = {
    CaseStatus.OPEN: [
        Transition("enrich",  to=CaseStatus.TO_ENRICH,     required=PermissionCode.CASE_WRITE),
    ],
    CaseStatus.TO_ENRICH: [
        Transition("submit_review", to=CaseStatus.MANUAL_REVIEW, required=PermissionCode.CASE_WRITE),
    ],
    CaseStatus.MANUAL_REVIEW: [
        Transition("approve", to=CaseStatus.APPROVED, required=PermissionCode.CASE_APPROVE,
                   requires_inputs=[InputSpec("approval_reason", "text", required=True)]),
        Transition("reject",  to=CaseStatus.REJECTED, required=PermissionCode.CASE_REJECT,
                   requires_inputs=[InputSpec("rejection_reason", "text", required=True)]),
    ],
    CaseStatus.APPROVED: [
        Transition("close", to=CaseStatus.CLOSED, required=PermissionCode.CASE_WRITE),
    ],
    # CANCELLED, REJECTED, CLOSED are terminal
}
```

### 4.3 Endpoints

Resource (kept REST-flavored; agent tool layer wraps each):

| Method | Path | Purpose |
|---|---|---|
| GET | `/cases` | Filtered + paginated case list (RBAC-scoped). |
| GET | `/cases/{case_id}` | Authorized case detail (404 if not visible). |
| GET | `/cases/{case_id}/activities` | Activities for a case. |
| GET | `/cases/{case_id}/payments` | Payments for a case. |
| GET | `/cases/{case_id}/mails` | Mails. |
| GET | `/cases/{case_id}/swift_messages` | SWIFT messages. |
| GET | `/cases/{case_id}/unifi_messages` | UNIFI/ISO 20022 messages. |
| GET | `/cases/{case_id}/transitions` | Available transitions (advisory; runs preflight per transition). |
| POST | `/cases/{case_id}/transitions/{name}/preflight` | Returns `{allowed, requiresConfirmation, missingInputs, sideEffects, preview, reasonCode?}`. |
| POST | `/cases/{case_id}/transitions/{name}/execute` | Authoritative execution. Body carries `payload` + `idempotency_key`. |
| POST | `/cases/{case_id}/assign` | Assign to a user. Preflight + execute under one path with `?dry_run=true` flag (compact). |
| POST | `/cases/{case_id}/notes` | Append a note. |
| POST | `/analytics/cases` | Semantic DSL: body = `{measures, dimensions, filters, limit}`. Validates against `semantic_model.yaml`. |
| GET | `/users/me` | Echo identity + roles. |
| GET | `/users/me/permissions` | Permission summary (for the agent's runtime context, NOT prompt). |
| GET | `/capabilities` | Full capability registry. |
| GET | `/routes` | Route registry. |
| GET | `/semantic-model` | Analytics semantic model. |
| POST | `/docs/search` | RAG: `{query, top_k}` → `[{title, chunk, score, source}]`. pgvector cosine sim + BM25 (or just vector for v1). |

All write endpoints:
- Run inside a transaction.
- Re-check permission **after** locking the case row (TOCTOU-safe).
- Honor `Idempotency-Key` request header — if a row exists in `command_executions`, replay the result.

### 4.4 Capability Registry shape (consumed by agent)

```json
{
  "name": "transition_case",
  "description": "Move a case from its current state to an allowed next state.",
  "type": "write",
  "endpoint":           "POST /cases/{case_id}/transitions/{name}/execute",
  "preflight_endpoint": "POST /cases/{case_id}/transitions/{name}/preflight",
  "required_permissions": ["CASE_WRITE"],
  "requires_confirmation": true,
  "idempotent": true,
  "input_schema": {
    "type": "object",
    "properties": {
      "case_id": {"type": "integer"},
      "name":    {"type": "string"},
      "payload": {"type": "object"}
    },
    "required": ["case_id", "name"]
  },
  "side_effects": [
    "Case state changes",
    "Audit event written",
    "May notify downstream systems"
  ]
}
```

`/capabilities` returns a list of these. The agent caches them at startup, refreshes on a 5-min TTL.

### 4.5 Analytics semantic model (`semantic_model.yaml`)

```yaml
datasets:
  cases:
    description: "Cases visible to the requesting user per RBAC."
    dimensions:
      - {name: state,          column: status}
      - {name: work_basket,    column: work_basket_id}
      - {name: product,        column: product_id}
      - {name: category,       column: category_id}
      - {name: opening_channel,column: opening_channel}
      - {name: created_date,   column: creation_timestamp::date}
    measures:
      - {name: case_count,            sql: "count(*)"}
      - {name: open_count,            sql: "count(*) filter (where status not in ('APPROVED','REJECTED','CANCELLED','CLOSED'))"}
      - {name: avg_processing_hours,  sql: "avg(extract(epoch from (timestamp_of_closing - timestamp_of_opening))/3600.0)"}
    allowed_filters: [state, work_basket, product, category, opening_channel, created_date]
    allowed_visualizations: [table, bar_chart, line_chart, kpi_grid]
    max_rows: 500
```

`POST /analytics/cases` validates request → compiles to parameterized SQL → applies `visible_case_filter` → caps rows → returns `{columns, rows, metadata}`.

### 4.6 Docs corpus
`mock_backend/docs_corpus/*.md` — 5-10 short markdown files (case lifecycle, role descriptions, basket meanings, state semantics, FAQ). `setup_db.py` chunks them (~500 tokens), embeds via the configured embeddings provider, stores in a `doc_chunk` table with `pgvector` column.

---

## 5. Agent service design

### 5.1 The graph state

```python
class AgentState(TypedDict, total=False):
    # transport
    messages: Annotated[list[BaseMessage], add_messages]
    incoming_event: dict                   # CHAT_MESSAGE | UI_ACTION
    app_context: dict                      # route, selectedCaseIds, filters, locale, timezone
    user_context: dict                     # user_id, real_name, permission summary

    # planning
    intent: dict                           # {kind, confidence, requires_business_data, requires_write}
    plan: dict | None                      # for compound flows
    retrieved_docs: list[dict]
    capability_candidates: list[dict]

    # work
    business_context: dict                 # current case state, available transitions, etc.
    permission_result: dict | None         # last preflight outcome
    tool_results: list[dict]

    # rendering
    view_model: dict | None
    ui_surfaces: list[dict]                # accumulated A2UI surfaces this turn
    active_surface_id: str | None

    # HITL
    pending_interrupt: dict | None
    human_decision: dict | None

    # finalization
    final_response: dict | None
    errors: list[dict]
```

### 5.2 The top-level graph

```
START
  → ingest_event              (deterministic)
  → hydrate_context           (deterministic: loads user_context, permission summary, app_context)
  → classify_intent           (LLM, structured output ∈ IntentKind)
  → route_by_intent           (deterministic switch)
       ├── general_question         → chat_qa_subgraph
       ├── app_docs_question        → docs_rag_subgraph
       ├── navigation               → navigation_subgraph
       ├── data_visualization       → analytics_subgraph
       ├── case_summary             → case_summary_subgraph
       ├── permission_explanation   → permission_explanation_subgraph
       ├── operation                → operation_subgraph
       ├── ui_refinement            → refinement_subgraph
       ├── compound                 → planner_subgraph    (re-dispatches step by step)
       ├── clarification_needed     → clarification_subgraph
       └── infeasible               → infeasible_subgraph
  → compose_ui                (deterministic dispatch; falls back to LLM composer for ChartIntent)
  → validate_ui               (deterministic: A2UI schema-check against component catalog)
  → emit_response             (writes final messages, RUN_FINISHED)
END
```

Operation subgraph internals (the hardest path):

```
operation_subgraph:
  identify_target_case          (from app_context or tool: search_cases)
  resolve_capability            (lookup in capability registry)
  fetch_business_state          (tool: get_case + get_possible_transitions)
  preflight_command             (tool: POST /cases/{id}/transitions/{name}/preflight)
  branch_preflight_result:
    denied        → build_denied_view_model        → compose → END
    missing_input → build_input_form_view_model    → compose → interrupt
    needs_confirm → build_confirmation_view_model  → compose → interrupt
  ──── INTERRUPT ────
  validate_resume_payload       (deterministic, against pending_resume_schema)
  execute_command               (tool: POST .../execute, idempotency key = ai_session+run+capability)
  fetch_updated_state
  build_success_view_model
  compose_success_ui
  END
```

**Rule**: authorization, confirmation, and identity resolution are **fixed nodes/edges**, not LLM-callable tools. Anything load-bearing for security or correctness is graph structure; anything genuinely "use judgment" is a tool or dynamic route.

### 5.3 Subgraphs by flow

| Flow | Subgraph | Key tools | Composer template |
|---|---|---|---|
| a — General chat | `chat_qa_subgraph` | `chat_llm` (no tools) | `TextBlock` / `MarkdownBlock` |
| b — App docs | `docs_rag_subgraph` | `docs_search` | `MarkdownBlock` + sources panel |
| c — Navigation | `navigation_subgraph` | `find_route`, `get_route_permissions` | `NavigationSuggestion` |
| d — Analytics | `analytics_subgraph` | `get_semantic_model`, `run_analytics` | LLM composer (ChartIntent → BarChart/LineChart/Table/KpiGrid) |
| e — Operation | `operation_subgraph` | `get_case`, `get_possible_transitions`, `preflight_*`, `execute_*` | `ConfirmationModal` + `Form` |
| f — Clarification | `clarification_subgraph` | (LLM only) | `Form` (quick-replies as chips) |
| g — UI refinement | `refinement_subgraph` | reuses analytics tools with prior surface state | overrides previous A2UI |
| h — Compound | `planner_subgraph` | re-enters router per step | varies |
| i — Infeasible/denied | `infeasible_subgraph` | (none) | `PermissionDeniedPanel` / `TextBlock` |
| j — Case summary | `case_summary_subgraph` | `get_case`, `get_case_activities`, `docs_search` | `CaseSummaryCard` + `CaseTimeline` |
| k — "Why can't I" | `permission_explanation_subgraph` | `get_possible_transitions` (with reasons), `permissions_summary` | `PermissionDeniedPanel` |

### 5.4 Orchestration: deterministic router now, supervisor later

The top-level graph (§5.2) uses a **deterministic router**: an LLM classifies the
intent once, and a *fixed* edge dispatches to a subgraph. Compound requests go to
a `planner_subgraph` that decomposes the request and re-enters the router per
step. This section records *why* that choice was made and what the future
alternative is, so a later session can revisit it deliberately rather than by
accident.

**Two clarifications that dissolve most of the confusion.**

1. *"Deterministic router" does not mean "no LLM."* The router **is** an LLM
   classifier — the model decides the intent. What is deterministic is the
   *edge*: given a classified intent, the dispatch is fixed and unit-testable.
   We are already letting the model decide; we are only constraining *where* its
   decision can lead.
2. *The security invariants live inside the subgraphs, not in the top-level
   routing.* `operation_subgraph` always runs preflight → confirm → execute in
   that order, regardless of how it was reached. So "deterministic top-level
   routing" and "the gates always fire" are **separate** guarantees. You could
   swap the top router for an agent and still keep the gates — as long as the
   gates stay as graph structure inside the workers.

**The trade-off (router/planner vs. supervisor-orchestrator).**

| Dimension | Deterministic router + planner-for-compound (chosen) | Supervisor-orchestrator (agent loops, calls subgraphs as tools) |
|---|---|---|
| Flexibility on open-ended multi-step | Lower — must hit the `compound` branch | Higher — plans dynamically, step by step |
| Predictability / auditability | High — classify once, fixed dispatch | Lower — agent decides each hop at runtime |
| Cost (LLM turns) | Low for the common single-intent case | Higher — every turn pays orchestration overhead |
| Failure modes | Bounded; no orchestration loops | Can loop / thrash; needs step + recursion limits |
| Testability | Deterministic edges → easy assertions | Needs scenario/eval-style tests |

**Why the chosen shape for v1.** The lab's purpose is to demonstrate the
architecture *clearly and testably*. The router keeps the common case (single
intent — "what is a basket?", "approve this") on a cheap, predictable,
one-classification path, and pays orchestration cost *only* for compound
requests. Crucially, the capability you might think you lose is not lost: the
design's `compound → planner_subgraph → re-enter router per step` **is** dynamic
step-by-step orchestration — it is just fenced off to where it earns its cost. If
the main agent orchestrated *everything*, then "what is a basket?" would also pay
orchestration overhead and unpredictability for no benefit.

**The future path (safe to take later).** The top level can be promoted to a
**supervisor-orchestrator** — a LangGraph pattern where the supervisor agent's
"tools" are the subgraphs and it decides which to call next at runtime. This
promotion is **safe specifically because** the security gates live inside the
workers, not in the router: an orchestrator that calls `operation_subgraph` still
gets preflight → confirm → execute in order. Indications it is time to promote:
compound/open-ended turns dominate real usage, or the fixed `IntentKind` set
starts to feel too coarse. A defensible *middle* option is a supervisor at the
top with subgraphs-as-tools while keeping every internal gate — more flexible
than the router, still safe, but harder to test deterministically, which is why
it is **not** the v1 default.

**Decision for v1:** deterministic router + `planner_subgraph` for compound.
Revisit only on the triggers above; the worker-internal gates make the migration
low-risk.

### 5.5 Tools — generic executor exposed as registry-generated typed tools

The agent does **not** hand-write one `@tool` function per backend endpoint.
Instead there is a single generic HTTP executor, and the model is presented with
**N typed tools generated from the capability registry at startup**. This gives
us the extensibility of "add an endpoint → update the registry → the agent picks
it up with no code change" *and* the tool-calling quality of narrow, typed,
well-described tools.

**Why not a single generic `call_backend(endpoint, method, payload)` tool.** A
single untyped mega-tool was considered and rejected for three reasons:
1. **Tool-calling quality.** LLMs are post-trained to call narrow, typed,
   well-described tools; correctness on path params, enums, and nested payloads
   comes from the per-tool schema steering the model. One do-anything tool forces
   the model to reconstruct the URL/method/body from prose — accuracy drops,
   worst on the write paths where mistakes are expensive.
2. **Per-subgraph scoping.** Distinct tools let `operation_subgraph` bind only
   write tools and `analytics_subgraph` bind only analytics tools. That scoping
   is a real safety/quality lever; a single tool cannot be scoped.
3. **Security still needs the allow-list anyway.** A generic executor must
   validate the requested `(method, path)` against the registry and reject
   anything not in the AI-allowed catalog — otherwise the agent can hit *any*
   route, including ones deliberately excluded from AI mode. So you never escape
   the registry; you only move the gate to execution time.

**The synthesis (what we build).** Keep the generic executor as the single place
that talks HTTP, but expose it to the model as registry-generated typed tools:

```python
# tools/executor.py  — the ONE place that performs HTTP
async def execute_capability(cap: Capability, args: dict, ctx: RuntimeContext) -> dict:
    # 1. allow-list check: cap MUST be in the registry (it came from there)
    # 2. build request from cap.endpoint + args (path/query/body split per input_schema)
    # 3. forward X-User-Id, X-Trace-Id, and Idempotency-Key for writes
    # 4. audit emit (TOOL_CALLED / CAPABILITY_EXECUTED) + structlog
    ...

# tools/factory.py  — runs ONCE at startup, after fetching /capabilities
def build_tools(registry: list[Capability]) -> list[StructuredTool]:
    tools = []
    for cap in registry:
        ArgsModel = pydantic_model_from_json_schema(cap.input_schema)   # typed args
        tools.append(StructuredTool.from_function(
            name=cap.name,
            description=cap.description,
            args_schema=ArgsModel,
            coroutine=partial(_invoke, cap),                            # → execute_capability
        ))
    return tools
```

Consequences and rules:
- **Extensibility:** add a backend endpoint → add a registry entry → tools
  regenerate on next agent boot. **Zero agent code change.**
- **Scoping:** subgraphs bind a filtered subset (`[t for t in tools if cap.type
  in (...)]` / by tag). Read subgraphs never see write tools.
- **Allow-list by construction:** the only callable capabilities are the ones in
  the registry; the executor double-checks at call time.
- **Writes** carry `requires_confirmation` from the registry, which is what makes
  `operation_subgraph` route through the HITL interrupt (§5.9).
- **The agent never gets a "raw SQL" tool. Period.** Analytics goes through the
  `run_analytics` capability (semantic DSL), never free SQL.
- This is deliberately **MCP-shaped**: a registry of typed capabilities behind a
  uniform executor is exactly what an MCP server exposes, so graduating to MCP
  later (out of v1 scope) is a transport swap, not a redesign.

**Capabilities present in the registry for v1** (each becomes a generated tool):
- Context: `get_current_user`, `get_user_permissions`, `get_app_context`
- Cases (read): `get_case`, `search_cases`, `get_case_activities`, `get_case_payments`, `get_case_mails`
- State machine: `get_possible_transitions`, `preflight_transition`, `execute_transition`
- Other writes: `assign_case`, `add_case_note`
- Analytics: `get_semantic_model`, `run_analytics`
- Knowledge: `docs_search`, `find_route`, `get_route_permissions`

The capability registry itself is fetched via a plain client call at startup (not
a model-facing tool); `list_capabilities` is not exposed to the LLM because the
generated tools *are* the capability surface.

### 5.6 UI composer

`composer/templates.py` is a deterministic dispatch:

```python
def compose(view_model: ViewModel, prior_surface: dict | None) -> A2UISurface:
    match view_model:
        case OperationConfirmation():  return _confirmation_template(view_model)
        case CaseSummaryViewModel():   return _case_summary_template(view_model)
        case PermissionDenied():       return _denied_template(view_model)
        case NavigationSuggestion():   return _nav_template(view_model)
        case RagAnswer():              return _rag_template(view_model)
        case ChartIntent():            return _llm_chart_composer(view_model, prior_surface)
        ...
        case _:                        return _plain_text_template(view_model)
```

`_llm_chart_composer` calls the LLM with a system prompt enumerating allowed component types from the catalog, the ChartIntent payload, and any prior surface for refinement turns. Response is parsed and **always** routed through `validator.validate_against_catalog(...)`.

### 5.7 A2UI component catalog (v1)

Catalog JSON Schema lives at `agent_service/catalogs/component_catalog.json`. Components:

- Layout/text: `TextBlock`, `MarkdownBlock`, `Section`, `ActionRow`
- Cards & summary: `KpiCard`, `KpiGrid`, `CaseSummaryCard`, `CaseTimeline`, `StateTransitionSummary`
- Charts/tables: `BarChart`, `LineChart`, `Table`
- Forms: `Form`, `TextInput`, `TextArea`, `Select`, `DatePicker`, `Checkbox`
- Modals & system: `ConfirmationModal`, `PermissionDeniedPanel`, `NavigationSuggestion`, `ErrorPanel`

Every component's schema has stable `id`, optional `dataRef`, and an `event` slot for action emissions:

```json
{ "id": "approve_btn", "label": "Approve", "event": {
    "type": "RESUME_INTERRUPT",
    "surfaceId": "<set by composer>", "actionId": "approve"
}}
```

### 5.8 Stream adapter

Tries `ag-ui-langgraph` first. If it doesn't fit (missing custom events, mapping mismatches, Pydantic friction, …), use a hand-rolled adapter that translates LangGraph chunk types:

| LangGraph chunk | AG-UI event |
|---|---|
| messages (chat token) | `TEXT_MESSAGE_CONTENT` (with bookending `_START` / `_END`) |
| tool_call_start / end | `TOOL_CALL_START` / `TOOL_CALL_END` |
| updates (state mutation) | `STATE_DELTA` (only for whitelisted state keys) |
| custom (`writer({...})`) | `CUSTOM` (used for `UI_SURFACE_UPDATE` payloads) |
| interrupt | `CUSTOM { name: "UI_SURFACE_UPDATE", payload: a2ui }` + `RUN_INTERRUPTED` |
| run start/end | `RUN_STARTED` / `RUN_FINISHED` |

Only whitelisted state keys (`ui_surfaces`, `active_surface_id`, `view_model.summary` …) are emitted as `STATE_DELTA` — never raw `tool_results` blobs (those go to artifacts).

### 5.9 HITL — Pattern A

```python
def request_confirmation_node(state: AgentState) -> dict:
    a2ui = compose(state["view_model"], prior=None)
    interrupt_payload = {
        "kind": "CONFIRMATION_REQUIRED",
        "interrupt_id": f"interrupt-{uuid4()}",
        "surface_id": a2ui["surface_id"],
        "a2ui": a2ui,
        "resume_schema": {
            "type": "object",
            "properties": {
                "decision": {"enum": ["approve", "reject", "edit"]},
                "form_values": {"type": "object"}
            },
            "required": ["decision"]
        }
    }
    decision = interrupt(interrupt_payload)        # <-- pauses, surfaces payload to adapter
    return {"human_decision": decision, "active_surface_id": a2ui["surface_id"]}
```

The adapter:

1. Sees the interrupt, emits a `CUSTOM { name: "UI_SURFACE_UPDATE", payload: a2ui }` event.
2. Emits `RUN_INTERRUPTED { reason: "CONFIRMATION_REQUIRED", surface_id, interrupt_id }`.
3. Writes `ai_session.status = WAITING_FOR_USER`, populates `pending_interrupt_id`, `pending_surface_id`, `pending_action_ids = ["approve", "reject"]`, `pending_resume_schema`.
4. Closes the SSE stream.

On the user's next event, the router validates `event.surface_id == pending_surface_id` and `event.action_id in pending_action_ids`, builds the resume payload (`{decision, form_values}`), then `graph.astream(Command(resume=...), config=...)`.

---

## 6. Flow specifications

Each flow below specifies: trigger, subgraph, tools, ViewModel, composer template, HITL? Each flow is one acceptance test (or several).

### Flow a — General chat
- Trigger: intent classifier returns `general_question`.
- Subgraph: `chat_qa_subgraph`. Single LLM call, no tools.
- ViewModel: `TextResponse(content: str)`.
- Composer: `TextBlock` or `MarkdownBlock`.
- HITL: no.

### Flow b — App-docs RAG
- Trigger: `app_docs_question`.
- Subgraph: `docs_rag_subgraph` → `docs_search` tool (top_k=5) → LLM synthesis with citations.
- ViewModel: `RagAnswer(content, sources)`.
- Composer: `MarkdownBlock` + `Section "Sources"` with linked titles.
- HITL: no.

### Flow c — Navigation
- Trigger: `navigation`.
- Subgraph: `navigation_subgraph` → `find_route(query)` → `get_route_permissions(route)` → if denied, switch to flow (i).
- ViewModel: `NavigationViewModel(route, label, params)`.
- Composer: `NavigationSuggestion` component (action emits a `NAVIGATE` event the client interprets).
- HITL: no.

### Flow d — Data visualization
- Trigger: `data_visualization`.
- Subgraph: `analytics_subgraph` → `get_semantic_model` (cached) → LLM builds `AnalysisRequest` (structured output, validated) → `run_analytics` → result either inlined or stored as artifact (>10 KB threshold).
- ViewModel: `ChartIntent(title, data | dataRef, recommended_components, allowed_interactions)`.
- Composer: LLM-driven; picks `KpiGrid`/`BarChart`/`LineChart`/`Table`; validates against catalog.
- HITL: no.

### Flow e — Operation (write)
- Trigger: `operation`.
- Subgraph: `operation_subgraph` (see 5.2 internals).
- ViewModels (different paths):
  - `OperationConfirmation(case_id, current_state, next_state, missing_inputs[], side_effects[], actions[])`
  - `PermissionDenied(reason_code, message, required_permission, ...)`
  - `OperationSuccess(case_id, new_state, summary, audit_link)`
- Composer: `ConfirmationModal` + `StateTransitionSummary` + `Form` + `ActionRow`.
- HITL: **yes** (Pattern A).

### Flow f — Clarification
- Trigger: `clarification_needed` (router low confidence OR planner detected ambiguity).
- Subgraph: `clarification_subgraph`. LLM generates 2–4 quick-reply options.
- ViewModel: `ClarificationPrompt(question, options[])`.
- Composer: a small `Form` with a `Select` or quick-reply chips (modeled as `ActionRow` with buttons), each emits a `CHOICE` action.
- HITL: optionally yes (we *could* interrupt here, but simpler is to send the surface, let user reply with a chat or UI action, and start a new run).

### Flow g — UI refinement
- Trigger: incoming `UI_ACTION` on a known surface AND session is `IDLE` (i.e., the previous run finished but the surface still lives).
- Subgraph: `refinement_subgraph` reads `state.ui_surfaces[-1]` from the checkpointer and re-issues the analytics with new params. Reuses prior `ChartIntent`.
- Composer: emits an A2UI surface with the **same** `surface_id` (treated as an update, not a new surface).
- HITL: no.

### Flow h — Compound multi-step
- Trigger: `compound` (planner detects "and then" or multiple intents).
- Subgraph: `planner_subgraph` decomposes into a sequence of single-intent steps. Each step re-enters the top-level router.
- HITL: yes if any step is an operation.

### Flow i — Infeasible / denied (terminal)
- Trigger: explicit terminal from any subgraph (`infeasible: true` in the result), or denied preflight in `operation_subgraph`.
- Composer: `PermissionDeniedPanel` or `ErrorPanel` with an actionable explanation.
- HITL: no.

### Flow j — Case summary
- Trigger: `case_summary`.
- Subgraph: `case_summary_subgraph` → `get_case` + `get_case_activities` + (optional) `docs_search` for state explanations → LLM-written summary.
- ViewModel: `CaseSummaryViewModel(case, timeline, risk_flags, recommended_actions)`.
- Composer: `CaseSummaryCard` + `CaseTimeline` + (optional) `ActionRow` with permission-checked suggested actions.
- HITL: no, but suggested actions emit events that may *kick off flow e* in a subsequent turn.

### Flow k — "Why can't I do this?"
- Trigger: `permission_explanation`.
- Subgraph: `permission_explanation_subgraph` → `get_possible_transitions(verbose=true)` → `permissions_summary` → LLM phrases the explanation.
- Composer: `PermissionDeniedPanel(reasons, required_roles, missing_permissions)`.
- HITL: no.

---

## 7. End-to-end example trace (Flow e)

User says: *"Approve case 4821."*

1. Client POSTs to `/ai/sessions/{id}/events` with `eventType=CHAT_MESSAGE`, `payload.text="Approve case 4821"`, `appContext.selectedCaseIds=[4821]`. Headers: `X-User-Id`, `X-Trace-Id`, `Idempotency-Key=client_event_id`.
2. Router: dedup miss → load session → status `IDLE` → mark `RUNNING` → invoke `graph.astream(input_state, config={thread_id})`. Audit emits `NODE_FIRED: ingest_event`.
3. `hydrate_context` runs `get_current_user` + `get_user_permissions` tools → populates `user_context`. Audit.
4. `classify_intent` → `{kind: "operation", confidence: 0.94}`. Audit.
5. Router dispatches to `operation_subgraph`.
6. `identify_target_case`: case 4821 already in `appContext` → skip search.
7. `resolve_capability` → `transition_case` (variant `approve`).
8. `fetch_business_state` calls `get_case(4821)` and `get_possible_transitions(4821)`. State = `MANUAL_REVIEW`, `approve` is in the list.
9. `preflight_command` calls `POST /cases/4821/transitions/approve/preflight` → backend returns `{allowed: true, requires_confirmation: true, missing_inputs: [{name: "approval_reason", type: "textarea", required: true}], side_effects: ["..."]}`.
10. `build_confirmation_view_model` constructs `OperationConfirmation(...)`.
11. `compose` → A2UI surface `surface-confirm-approve-4821` with `ConfirmationModal` + `StateTransitionSummary` + `TextArea("approval_reason")` + `ActionRow([approve, reject])`.
12. `interrupt(payload)` — graph pauses. Adapter emits `CUSTOM(UI_SURFACE_UPDATE)` + `RUN_INTERRUPTED`. `ai_session.status=WAITING_FOR_USER`, `pending_interrupt_id`, `pending_surface_id`, `pending_action_ids=["approve","reject"]`, `pending_resume_schema={...}`. SSE closes.
13. User clicks Approve, fills reason. Client POSTs `eventType=UI_ACTION, surfaceId=surface-confirm-approve-4821, actionId=approve, payload.approval_reason="Reviewed and valid"`.
14. Router: dedup miss → load session → status `WAITING_FOR_USER` → `matches_pending` ✓ → build resume payload `{decision: "approve", form_values: {approval_reason: "..."}}` → `graph.astream(Command(resume=...), ...)`.
15. `interrupt()` returns the resume value. Node sets `human_decision`.
16. `validate_resume_payload`: schema-check passes.
17. `execute_command` calls `POST /cases/4821/transitions/approve/execute` with `Idempotency-Key`. Backend: locks case, re-checks permission, applies transition, writes audit, returns updated case.
18. `fetch_updated_state` + `build_success_view_model` → `OperationSuccess(...)`.
19. `compose_success_ui` → A2UI surface with `CaseSummaryCard` + success `TextBlock`.
20. `emit_response` → `RUN_FINISHED`. Adapter writes `ai_session.status=IDLE`. Stream closes.

---

## 8. Build order (phasing inside v1)

Even though scope is "all flows", we build in an order that de-risks the hard pieces:

1. **Plumbing**:
   - Mock backend: schema, seeds, `/cases` read endpoints, RBAC, `/users/me/permissions`, `/capabilities`.
   - Agent service: `ai_session`, idempotency, audit, structlog, `setup_db.py`.
   - Compiled empty graph + SSE endpoint streaming a single `RUN_STARTED`/`RUN_FINISHED`.
2. **Flow a + b** (chat + docs RAG):
   - Docs corpus, pgvector, `docs_search` tool.
   - Intent router (just two classes for now).
   - Composer's `MarkdownBlock` template.
3. **Flow e** (operation + HITL — the architecturally hardest path):
   - State machine, `preflight`/`execute` endpoints.
   - `operation_subgraph` with `interrupt()`.
   - Pattern-A confirmation, resume validation.
   - **This is the moment the system architecture is proven.**
4. **Flow d** (analytics, Tier A then Tier B):
   - Semantic model file + `/analytics/cases` endpoint.
   - LLM composer for charts, catalog validator.
   - Artifact store + `dataRef` for one large result demo.
5. **Flow c** (navigation), **flow j** (case summary), **flow k** (permission explanation), **flow i** (infeasible terminal). Mostly composer templates and small subgraphs.
6. **Flow f** (clarification) and **flow g** (UI refinement). Refinement requires correct surface-state persistence in checkpointed state.
7. **Flow h** (compound). Planner subgraph that re-enters the top-level router per step.
8. End-to-end tests with `pytest` + `httpx` + a `cli_consumer.py` for human inspection.

---

## 9. Open questions deferred (acknowledged, not in v1)

These are intentionally **not** addressed in this lab; the design above does not preclude any of them:

- Postgres RLS-based row-level read enforcement (production move).
- MCP server in front of the capability layer.
- LangGraph managed/standalone server deployment.
- Real Angular A2UI renderer.
- Tier-C constrained text-to-SQL.
- Proactive flows, what-if simulation, batch dry-run.
- Multi-tenant scoping.
- Concurrency control beyond single-active-run-per-session.

---

## 10. Acceptance criteria for the lab

The lab is "done" when:

- `python mock_backend/src/mock_backend/setup_db.py` and `python agent_service/src/agent_service/setup_db.py` produce clean DBs with seeds.
- `pytest` passes for both projects, including at least one e2e test per flow (a–k).
- `python agent_service/scripts/cli_consumer.py "Approve case 4821"` produces a human-readable AG-UI event log showing the full flow (e) trace from §7.
- The mock backend's `/capabilities`, `/routes`, `/semantic-model`, `/docs/search`, and case endpoints all work standalone (curl-friendly).
- The architectural principle holds end-to-end: every write path is gated by the mock backend's permission re-check, every read path is RBAC-scoped, every confirmation goes through `interrupt()`/`Command(resume=…)`.
