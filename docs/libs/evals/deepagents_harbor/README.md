# `libs/evals/deepagents_harbor/`

## What This Directory Contains

Harbor integration layer for running Deep Agents evaluations in sandboxed environments and collecting results in LangSmith. Implements the `SandboxBackendProtocol` for Harbor, wraps Deep Agents in Harbor's `BaseAgent` interface, classifies failures, collects infrastructure metadata, and manages LangSmith datasets and experiments.

## Files

| File | Description |
|------|-------------|
| `backend.py` | `HarborSandbox` — implements `SandboxBackendProtocol` using shell commands to interact with Harbor sandbox containers |
| `deepagents_wrapper.py` | `DeepAgentsWrapper(BaseAgent)` — runs a Deep Agent inside Harbor, streaming tool calls and collecting ATIF trajectories |
| `failure.py` | `FailureCategory` enum, `classify_failure()`, and `extract_exit_codes()` for categorizing agent run outcomes |
| `langsmith.py` | Dataset creation, experiment creation, and feedback upload functions for LangSmith observability |
| `metadata.py` | `InfraMetadata` dataclass and `collect_sandbox_metadata()` for recording sandbox environment details |
| `stats.py` | `wilson_ci()`, `format_ci()`, `min_detectable_effect()` for statistical reporting of eval scores |
| `__init__.py` | Re-exports all public symbols for clean `from deepagents_harbor import ...` usage |

## Public API (from `__init__.py`)

```python
from deepagents_harbor import (
    HarborSandbox,
    DeepAgentsWrapper,
    FailureCategory,
    InfraMetadata,
    add_feedback,
    create_dataset,
    create_example_id_from_instruction,
    create_experiment,
    ensure_dataset,
)
```

## Architecture

```
Harbor CLI
    │
    └── DeepAgentsWrapper.run(task) → ATIF trajectory
            ├── HarborSandbox (backend)
            │   └── shell commands to sandbox container
            ├── create_deep_agent (agent)
            └── LangSmith tracing → experiment + feedback
```

## Related Docs

- [`backend.md`](backend.md) — `HarborSandbox` file/shell operations
- [`deepagents_wrapper.md`](deepagents_wrapper.md) — `DeepAgentsWrapper` execution and trajectory collection
- [`failure.md`](failure.md) — `FailureCategory` enum and failure classification
- [`langsmith.md`](langsmith.md) — Dataset and experiment management
- [`metadata.md`](metadata.md) — `InfraMetadata` sandbox environment collection
- [`stats.md`](stats.md) — Wilson confidence intervals and statistical reporting
