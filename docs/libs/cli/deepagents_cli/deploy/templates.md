# `libs/cli/deepagents_cli/deploy/templates.py`

> String templates for generated deployment artifacts.

## Position in the system

This file belongs to the `deepagents deploy` path. The deploy package reads a project layout, validates `deepagents.toml`, bundles prompts, memories, skills, MCP config, optional frontend assets, and emits files that `langgraph deploy` can run.

## Functions and classes

This module has no public functions or classes. It exists for package discovery, typing markers, constants, side-effect imports, or re-export behavior described above.

## System prompts, tool descriptions, and templates

### `MCP_TOOLS_TEMPLATE`

```text
async def _load_mcp_tools():
    """Load MCP tools from bundled config (http/sse only)."""
    import json
    from pathlib import Path

    mcp_path = Path(__file__).parent / "_mcp.json"
    if not mcp_path.exists():
        return []

    try:
        raw = json.loads(mcp_path.read_text())
    except Exception as exc:  # noqa: BLE001
        logger.warning("Failed to parse _mcp.json: %s", exc)
        return []

    servers = raw.get("mcpServers", {})
    connections = {}
    for name, cfg in servers.items():
        transport = cfg.get("type", cfg.get("transport", "stdio"))
        if transport in ("http", "sse"):
            conn = {"transport": transport, "url": cfg["url"]}
            if "headers" in cfg:
                conn["headers"] = cfg["headers"]
            connections[name] = conn

    if not connections:
        return []

    try:
        from langchain_mcp_adapters.client import MultiServerMCPClient

        client = MultiServerMCPClient(connections)
        return await client.get_tools()
    except Exception as exc:  # noqa: BLE001
        logger.warning(
            "Failed to load MCP tools from %d server(s): %s",
            len(connections),
            exc,
        )
        return []
```

### `SYNC_SUBAGENTS_TEMPLATE`

```text
from deepagents.middleware.subagents import SubAgent


async def _build_sync_subagents(seed, store, assistant_id):
    """Build SubAgent dicts from seed data and seed their memories/skills."""
    subagents_data = seed.get("subagents", {})
    if not subagents_data:
        return []

    subagents = []
    for name, data in subagents_data.items():
        sa: SubAgent = {
            "name": data["config"]["name"],
            "description": data["config"]["description"],
            "system_prompt": data["memories"]["/AGENTS.md"],
        }
        if data["config"].get("model"):
            sa["model"] = data["config"]["model"]

        # Seed subagent memories and skills into store under subagent namespace.
        sa_ns = (assistant_id, "subagents", name)
        if store is not None:
            for path, content in data.get("memories", {}).items():
                if await store.aget(sa_ns, path) is None:
                    await store.aput(
                        sa_ns,
                        path,
                        {"content": content, "encoding": "utf-8"},
                    )
            for path, content in data.get("skills", {}).items():
                if await store.aget(sa_ns, path) is None:
                    await store.aput(
                        sa_ns,
                        path,
                        {"content": content, "encoding": "utf-8"},
                    )

        sa_prefix = f"/memories/subagents/{name}/"
        if data.get("skills"):
            sa["skills"] = [f"{sa_prefix}skills/"]

        if data.get("mcp"):
            sa["tools"] = await _load_subagent_mcp_tools(data["mcp"])

        # Restrict filesystem access to the subagent's own namespace.
        # Allow comes first (first-match wins); the deny rule blocks
        # everything else under /memories/ — parent AGENTS.md, skills, etc.
        sa["permissions"] = [
            FilesystemPermission(
                operations=["read", "write"],
                paths=[f"{sa_prefix}**"],
                mode="allow",
            ),
            FilesystemPermission(
                operations=["read", "write"],
                paths=["/memories/**"],
                mode="deny",
            ),
        ]

        subagents.append(sa)
    return subagents


async def _load_subagent_mcp_tools(mcp_config):
    """Load MCP tools for a subagent from its mcp config."""
    servers = mcp_config.get("mcpServers", {})
    connections = {}
    for sname, cfg in servers.items():
        transport = cfg.get("type", cfg.get("transport", "stdio"))
        if transport in ("http", "sse"):
            conn = {"transport": transport, "url": cfg["url"]}
            if "headers" in cfg:
                conn["headers"] = cfg["headers"]
            connections[sname] = conn

    if not connections:
        return []

    try:
        from langchain_mcp_adapters.client import MultiServerMCPClient

        client = MultiServerMCPClient(connections)
        return await client.get_tools()
    except Exception as exc:  # noqa: BLE001
        logger.warning("Failed to load subagent MCP tools: %s", exc)
        return []
```

### `DEPLOY_GRAPH_TEMPLATE`

```text
"""Auto-generated deepagents deploy entry point.

Created by `deepagents deploy`. Do not edit manually — changes will be
overwritten on the next deploy.
"""

import json
import logging
import os
from pathlib import Path
from typing import TYPE_CHECKING

from deepagents import create_deep_agent
from deepagents.backends.composite import CompositeBackend
from deepagents.backends.protocol import SandboxBackendProtocol
from deepagents.backends.store import StoreBackend
from deepagents.middleware.permissions import FilesystemPermission
from langchain.agents.middleware.types import (
    AgentMiddleware,
    AgentState,
    ModelRequest,
    ModelResponse,
    PrivateStateAttr,
)
from langchain_core.runnables import RunnableConfig
from langgraph.prebuilt import ToolRuntime

if TYPE_CHECKING:
    from langgraph.runtime import Runtime
    from langgraph_sdk.runtime import ServerRuntime

logger = logging.getLogger(__name__)

SANDBOX_SNAPSHOT = {sandbox_snapshot!r}
SANDBOX_IMAGE = {sandbox_image!r}

# Mount points inside the composite backend.
# Everything lives under /memories/ — longest-prefix-first routing
# ensures /memories/user/ and /memories/skills/ match before /memories/.
MEMORIES_PREFIX = "/memories/"
SKILLS_PREFIX = "/memories/skills/"
USER_PREFIX = "/memories/user/"

HAS_USER_MEMORIES = {has_user_memories!r}

# `/memories/` backing store. "store" routes through the LangGraph runtime
# store (in-memory for `langgraph dev`, Postgres on the platform). "hub"
# persists into a LangSmith Hub agent repo via ContextHubBackend — a single
# hub repo for the agent, plus a per-user repo when user memories are on.
MEMORIES_BACKEND = {memories_backend!r}
MEMORIES_HUB_IDENTIFIER = {memories_hub_identifier!r}

# What to seed into the store on first run.
SEED_PATH = Path(__file__).parent / "_seed.json"


class SandboxSyncMiddleware(AgentMiddleware):
    """Sync skill files from the store into the sandbox filesystem.

    Downloads all files under the configured skill sources from the composite
    backend (which routes /skills/ to the store) and uploads them directly
    into the sandbox so scripts can be executed.
    """

    def __init__(self, *, backend, sources):
        self._backend = backend
        self._sources = sources
        self._synced_keys: set = set()

    def _get_backend(self, state, runtime, config):
        if callable(self._backend):
            tool_runtime = ToolRuntime(
                state=state,
                context=runtime.context,
                stream_writer=runtime.stream_writer,
                store=runtime.store,
                config=config,
                tool_call_id=None,
            )
            return self._backend(tool_runtime)
        return self._backend

    async def _collect_files(self, backend, path):
        """Recursively list all files under *path* via ls (not glob)."""
        result = await backend.als(path)
        files = []
        for entry in result.entries or []:
            if entry.get("is_dir"):
                files.extend(await self._collect_files(backend, entry["path"]))
            else:
                files.append(entry["path"])
        return files

    async def abefore_agent(self, state, runtime, config):
        backend = self._get_backend(state, runtime, config)
        if not isinstance(backend, CompositeBackend):
            return None
        sandbox = backend.default
        if not isinstance(sandbox, SandboxBackendProtocol):
            return None

        # Only sync once per sandbox instance
        cache_key = id(sandbox)
        if cache_key in self._synced_keys:
            return None
        self._synced_keys.add(cache_key)

        files_to_upload = []
        for source in self._sources:
            paths = await self._collect_files(backend, source)
            if not paths:
                continue
            responses = await backend.adownload_files(paths)
            for resp in responses:
                if resp.content is not None:
                    files_to_upload.append((resp.path, resp.content))

        if files_to_upload:
            results = await sandbox.aupload_files(files_to_upload)
            uploaded = sum(1 for r in results if r.error is None)
            logger.info(
                "Synced %d/%d skill files into sandbox",
                uploaded,
                len(files_to_upload),
            )

        return None

    def wrap_model_call(self, request, handler):
        return handler(request)

    async def awrap_model_call(self, request, handler):
        return await handler(request)


_SEED_CACHE: dict | None = None


def _load_seed() -> dict:
    """Load and cache the bundled seed payload."""
    global _SEED_CACHE
    if _SEED_CACHE is not None:
        return _SEED_CACHE
    if not SEED_PATH.exists():
        _SEED_CACHE = {{"memories": {{}}, "skills": {{}}, "user_memories": {{}}}}
        return _SEED_CACHE
    try:
        _SEED_CACHE = json.loads(SEED_PATH.read_text(encoding="utf-8"))
    except Exception as exc:  # noqa: BLE001
        logger.warning("Failed to parse _seed.json: %s", exc)
        _SEED_CACHE = {{"memories": {{}}, "skills": {{}}, "user_memories": {{}}}}
    return _SEED_CACHE


# Per-(process, assistant_id) gate.
_SEEDED_ASSISTANTS: set[str] = set()

# Per-(process, assistant_id) gate for hub seeding.
_SEEDED_HUB_ASSISTANTS: set[str] = set()

# Per-(process, assistant_id, user_id) gate for user memories.
_SEEDED_USERS: set[tuple[str, str]] = set()

# Per-(process, assistant_id, user_id) gate for per-user hub seeding.
_SEEDED_HUB_USERS: set[tuple[str, str]] = set()


_USER_ID_SAFE_CHARS = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789_-"


def _sanitize_user_id(user_id: str) -> str:
    """Translate a user identity into a hub-repo-name-safe slug.

    Replaces any non `[A-Za-z0-9_-]` character with `-` and truncates to 40
    chars so callers with long identities (emails, provider-prefixed IDs,
    long UUIDs) still get a workable repo name.
    """
    slug = "".join(c if c in _USER_ID_SAFE_CHARS else "-" for c in user_id)
    return slug[:40]


async def _seed_store_if_needed(store, assistant_id: str) -> None:
    """Seed memories + skills under `assistant_id` once per process."""
    if assistant_id in _SEEDED_ASSISTANTS:
        return
    _SEEDED_ASSISTANTS.add(assistant_id)

    seed = _load_seed()

    memories_ns = (assistant_id,)
    for path, content in seed.get("memories", {{}}).items():
        if await store.aget(memories_ns, path) is None:
            await store.aput(
                memories_ns,
                path,
                {{"content": content, "encoding": "utf-8"}},
            )

    skills_ns = (assistant_id,)
    for path, content in seed.get("skills", {{}}).items():
        if await store.aget(skills_ns, path) is None:
            await store.aput(
                skills_ns,
                path,
                {{"content": content, "encoding": "utf-8"}},
            )


async def _seed_user_memories_if_needed(
    store, assistant_id: str, user_id: str,
) -> None:
    """Seed user memory templates once per (assistant_id, user_id).

    Only writes entries that do not yet exist in the store, so
    user-modified memories are never overwritten.
    """
    key = (assistant_id, user_id)
    if key in _SEEDED_USERS:
        return
    _SEEDED_USERS.add(key)

    seed = _load_seed()
    user_memories = seed.get("user_memories", {{}})
    if not user_memories:
        return

    user_ns = (assistant_id, user_id)
    for path, content in user_memories.items():
        if await store.aget(user_ns, path) is None:
            await store.aput(
                user_ns,
                path,
                {{"content": content, "encoding": "utf-8"}},
            )
    logger.info(
        "Seeded %d user memory template(s) for user %s",
        len(user_memories),
        user_id,
    )


def _hub_route_or_none(backend, prefix: str):
    """Return the ContextHubBackend behind ``prefix`` on the composite, or None."""
    routes = getattr(backend, "routes", None)
    if routes is None:
        return None
    return routes.get(prefix)


def _log_seed_errors(responses, scope: str) -> None:
    """Surface upload errors at warn level; the deploy continues either way."""
    failures = [r for r in responses if r.error is not None]
    if failures:
        logger.warning(
            "Hub seed had %d failed file(s) in %s: %s",
            len(failures),
            scope,
            [(r.path, r.error) for r in failures],
        )


async def _seed_hub_if_needed(backend, assistant_id: str) -> None:
    """Seed the agent hub repo on first deploy as a single multi-file commit.

    Skips entirely once the repo has any prior commits, so user edits/deletes
    in the LangSmith UI are never silently undone on redeploy or restart.
    """
    if assistant_id in _SEEDED_HUB_ASSISTANTS:
        return
    _SEEDED_HUB_ASSISTANTS.add(assistant_id)

    hub = _hub_route_or_none(backend, MEMORIES_PREFIX)
    if hub is not None and hub.has_prior_commits():
        return

    seed = _load_seed()
    batch: list[tuple[str, bytes]] = []
    for path, content in seed.get("memories", {{}}).items():
        full_path = f"{{MEMORIES_PREFIX}}{{path.lstrip('/')}}"
        batch.append((full_path, content.encode("utf-8")))
    for path, content in seed.get("skills", {{}}).items():
        full_path = f"{{SKILLS_PREFIX}}{{path.lstrip('/')}}"
        batch.append((full_path, content.encode("utf-8")))
    for sa_name, sa_data in seed.get("subagents", {{}}).items():
        sa_prefix = f"{{MEMORIES_PREFIX}}subagents/{{sa_name}}/"
        for path, content in sa_data.get("memories", {{}}).items():
            full_path = f"{{sa_prefix}}{{path.lstrip('/')}}"
            batch.append((full_path, content.encode("utf-8")))
        for path, content in sa_data.get("skills", {{}}).items():
            full_path = f"{{sa_prefix}}skills/{{path.lstrip('/')}}"
            batch.append((full_path, content.encode("utf-8")))

    if not batch:
        return
    responses = await backend.aupload_files(batch)
    _log_seed_errors(responses, scope=f"agent={{assistant_id}}")


async def _seed_user_hub_if_needed(
    backend, assistant_id: str, user_id: str,
) -> None:
    """Seed user memory templates into the per-user hub repo on first use.

    Same first-deploy gate as :func:`_seed_hub_if_needed`: once the user's
    repo has any commits, subsequent invocations skip seeding so the user's
    own changes (including deletes) survive across runs.
    """
    key = (assistant_id, user_id)
    if key in _SEEDED_HUB_USERS:
        return
    _SEEDED_HUB_USERS.add(key)

    seed = _load_seed()
    user_memories = seed.get("user_memories", {{}})
    if not user_memories:
        return

    user_hub = _hub_route_or_none(backend, USER_PREFIX)
    if user_hub is not None and user_hub.has_prior_commits():
        return

    batch = [
        (f"{{USER_PREFIX}}{{path.lstrip('/')}}", content.encode("utf-8"))
        for path, content in user_memories.items()
    ]
    responses = await backend.aupload_files(batch)
    _log_seed_errors(responses, scope=f"user={{user_id}}")
    logger.info(
        "Seeded %d user memory template(s) into hub for user %s",
        len(user_memories),
        user_id,
    )


{sandbox_block}

{mcp_tools_block}

{sync_subagents_block}


def _make_namespace_factory(assistant_id: str, *extra: str):
    """Return a namespace factory closed over an assistant id + extra."""
    ns = (assistant_id, *extra)
    def _factory(ctx):  # noqa: ARG001
        return ns
    return _factory


def _make_user_namespace_factory(assistant_id: str):
    """Return a namespace factory that includes the user_id.

    Uses `rt.server_info.user.identity` from custom auth.  The platform
    always injects user_id from auth, so no configurable fallback is needed.
    """
    def _factory(rt):
        user = getattr(rt.server_info, "user", None) if rt.server_info else None
        identity = getattr(user, "identity", None) if user else None
        if not identity:
            raise ValueError(
                "user_id is required when user memories are enabled. "
                "Set it via custom auth (runtime.user.identity)."
            )
        return (assistant_id, str(identity))
    return _factory


SANDBOX_SCOPE = {sandbox_scope!r}


def _build_backend_factory(assistant_id: str, user_id: str | None = None):
    """Return a backend factory that builds the composite per invocation."""
    def _factory(ctx):  # noqa: ARG001
        from langgraph.config import get_config

        if SANDBOX_SCOPE == "assistant":
            cache_key = f"assistant:{{assistant_id}}"
        else:
            thread_id = get_config().get("configurable", {{}}).get("thread_id", "local")
            cache_key = f"thread:{{thread_id}}"
        sandbox_backend = _get_or_create_sandbox(cache_key)

        if MEMORIES_BACKEND == "hub":
            # Vendored alongside the generated graph by the bundler.
            from _context_hub import ContextHubBackend

            routes = {{
                MEMORIES_PREFIX: ContextHubBackend(identifier=MEMORIES_HUB_IDENTIFIER),
            }}
            if HAS_USER_MEMORIES and user_id:
                user_hub_identifier = (
                    f"{{MEMORIES_HUB_IDENTIFIER}}-user-{{_sanitize_user_id(user_id)}}"
                )
                routes[USER_PREFIX] = ContextHubBackend(identifier=user_hub_identifier)
        else:
            routes = {{
                MEMORIES_PREFIX: StoreBackend(
                    namespace=_make_namespace_factory(assistant_id),
                ),
                SKILLS_PREFIX: StoreBackend(
                    namespace=_make_namespace_factory(assistant_id),
                ),
            }}
            if HAS_USER_MEMORIES:
                routes[USER_PREFIX] = StoreBackend(
                    namespace=_make_user_namespace_factory(assistant_id),
                )

            # Subagent store routes only apply in store mode. In hub mode,
            # subagent content lives under /memories/subagents/... in the
            # agent's hub repo and is reached via the single /memories/ mount.
            seed = _load_seed()
            for sa_name in seed.get("subagents", {{}}):
                sa_prefix = f"{{MEMORIES_PREFIX}}subagents/{{sa_name}}/"
                routes[sa_prefix] = StoreBackend(
                    namespace=_make_namespace_factory(
                        assistant_id, "subagents", sa_name
                    ),
                )

        return CompositeBackend(
            default=sandbox_backend,
            routes=routes,
        )
    return _factory


async def make_graph(config: RunnableConfig, runtime: "ServerRuntime"):
    """Async graph factory.

    Accepts the invocation's `RunnableConfig` for `assistant_id` and
    the `ServerRuntime` for `store` and `user.identity`.  Seeds
    memories + skills once per (process, assistant_id), and user memories
    once per (process, assistant_id, user_id).  Gracefully skips user
    memory features when no user_id is available.
    """
    configurable = (config or {{}}).get("configurable", {{}}) or {{}}
    assistant_id = str(configurable.get("assistant_id") or {default_assistant_id!r})

    store = getattr(runtime, "store", None)
    user_id = None
    if HAS_USER_MEMORIES:
        user = getattr(runtime, "user", None)
        identity = getattr(user, "identity", None) if user else None
        user_id = str(identity) if identity else None
    if HAS_USER_MEMORIES and not user_id:
        logger.warning(
            "User memories are enabled but no user_id found "
            "(runtime.user.identity is empty). User memory features "
            "will be skipped for this invocation."
        )

    tools: list = []
    {mcp_tools_load_call}

    seed = _load_seed()
    all_subagents: list = []
    {sync_subagents_load_call}

    backend_factory = _build_backend_factory(assistant_id, user_id)

    if MEMORIES_BACKEND == "hub":
        # Seed via the composite so writes land in the hub repo(s).
        _seed_backend = backend_factory(None)
        await _seed_hub_if_needed(_seed_backend, assistant_id)
        if HAS_USER_MEMORIES and user_id:
            await _seed_user_hub_if_needed(_seed_backend, assistant_id, user_id)
    elif store is not None:
        await _seed_store_if_needed(store, assistant_id)
        if HAS_USER_MEMORIES and user_id:
            await _seed_user_memories_if_needed(store, assistant_id, user_id)

    # Preload AGENTS.md + user memory into the agent's context.
    memory_sources = [f"{{MEMORIES_PREFIX}}AGENTS.md"]
    if HAS_USER_MEMORIES and user_id:
        memory_sources.append(f"{{USER_PREFIX}}AGENTS.md")

    # When agent_writable=False (default), all agent memory is read-only;
    # only user memories are writable. Allow rule comes first
    # (first-match-wins), then deny everything else under /memories/.
    # When agent_writable=True, no permissions are needed (everything writable).
    AGENT_WRITABLE = {agent_writable!r}
    if AGENT_WRITABLE:
        permissions = []
    else:
        permissions = [
            FilesystemPermission(
                operations=["write"],
                paths=[f"{{USER_PREFIX}}**"],
                mode="allow",
            ),
            FilesystemPermission(
                operations=["write"],
                paths=[f"{{MEMORIES_PREFIX}}**"],
                mode="deny",
            ),
        ]

    return create_deep_agent(
        model={model!r},
        memory=memory_sources,
        skills=[SKILLS_PREFIX],
        tools=tools,
        subagents=all_subagents or None,
        backend=backend_factory,
        permissions=permissions,
        middleware=[
            SandboxSyncMiddleware(backend=backend_factory, sources=[SKILLS_PREFIX]),
        ],
    )


graph = make_graph
```

### `PYPROJECT_TEMPLATE`

```text
[project]
name = {agent_name!r}
version = "0.1.0"
requires-python = ">=3.12"
dependencies = [
    "deepagents==0.5.3",
{extra_deps}]

[tool.setuptools]
py-modules = []
```

### `APP_PY_TEMPLATE`

```text
"""Starlette app mounting the bundled chat UI on /app.

Generated by `deepagent deploy`. LangGraph Platform reads the `http.app`
key in `langgraph.json` and attaches this app alongside the graph.

Uses Starlette directly (not FastAPI) because Starlette is already a
transitive dep of langgraph-cli / langgraph-api in both the dev runtime
and the deployed runtime, whereas FastAPI would require an explicit
install step that `langgraph dev` does not perform.
"""

from __future__ import annotations

from pathlib import Path

from starlette.applications import Starlette
from starlette.responses import JSONResponse, RedirectResponse
from starlette.routing import Mount, Route
from starlette.staticfiles import StaticFiles

_FRONTEND_DIR = Path(__file__).parent / "frontend_dist"


async def healthz(_request):
    return JSONResponse({"ok": True})


async def app_root_redirect(_request):
    # Starlette's Mount at "/app" matches "/app/*" — a bare "/app" 404s
    # otherwise. Redirect so users typing the clean URL land correctly.
    return RedirectResponse(url="/app/", status_code=308)


app = Starlette(
    routes=[
        Route("/healthz", healthz),
        Route("/app", app_root_redirect),
        Mount(
            "/app",
            app=StaticFiles(directory=str(_FRONTEND_DIR), html=True),
            name="frontend",
        ),
    ],
)
```

## Gotchas

Deploy helpers usually write files, read environment variables, or shell out through command helpers. Keep validation errors explicit because deployment failures otherwise surface late inside LangGraph Cloud tooling.
