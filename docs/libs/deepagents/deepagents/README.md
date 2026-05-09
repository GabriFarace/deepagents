# `libs/deepagents/deepagents/`

> Importable SDK package. The public surface is intentionally small, but the
> package contains the main orchestration factory and the pluggable pieces it
> assembles.

## Position in the system

Most user code imports from the package root:

```python
from deepagents import create_deep_agent
```

That symbol is re-exported from `graph.py`. The factory then pulls in model
profiles, provider profiles, backend implementations, and middleware classes to
build the final LangGraph `CompiledStateGraph`.

## Module map

| Module | Doc | What to read it for |
|---|---|---|
| `graph.py` | [`graph.md`](./graph.md) | The complete assembly flow for a deep agent. |
| `_models.py` | [`_models.md`](./_models.md) | Converting model strings into `BaseChatModel` instances and inspecting model identity. |
| `_tools.py` | [`_tools.md`](./_tools.md) | Rewriting tool descriptions from harness profiles without mutating caller-owned tools. |
| `_excluded_middleware.py` | `_excluded_middleware.md` | Profile-driven middleware exclusion validation. |
| `backends/` | [`backends/`](./backends/README.md) | Backend protocol and implementations. |
| `middleware/` | [`middleware/`](./middleware/README.md) | The bulk of agent behavior. |
| `profiles/` | [`profiles/`](./profiles/README.md) | Model/provider-specific adjustments. |

## Reading order

Read [`graph.md`](./graph.md) first. It names almost every other SDK concept
at the point where it is assembled. Then read the backend protocol before the
filesystem middleware, because the middleware's tools are thin wrappers around
backend methods.
