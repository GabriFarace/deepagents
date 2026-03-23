# `deepagents_harbor/deepagents_wrapper.py`

## High-Level Purpose

Wraps the Deep Agents SDK as a Harbor `BaseAgent`, allowing Harbor to use a Deep Agent to execute benchmark tasks inside Harbor sandbox environments. Handles infrastructure metadata collection, LangSmith tracing, agent invocation, and trajectory serialization in the ATIF v1.2 format.

## Module-Level Constants

| Constant | Value | Description |
|----------|-------|-------------|
| `_MAX_FILE_LISTING` | `10` | Maximum files shown in the system prompt directory context |
| `SYSTEM_MESSAGE` | (template string) | System prompt template injected into the agent, containing current directory info and file listing |

## Classes

### `DeepAgentsWrapper`

**Purpose:** Harbor agent implementation using LangChain Deep Agents. Implements `BaseAgent` from Harbor, executing tasks via either the CLI agent (`create_cli_agent`) or the SDK agent (`create_deep_agent`).

**Inherits from:** `harbor.agents.base.BaseAgent`

#### `__init__`

```python
def __init__(
    self,
    logs_dir: Path,
    model_name: str | None = None,
    temperature: float = 0.0,
    verbose: bool = True,
    use_cli_agent: bool = True,
    *args, **kwargs,
) -> None
```

**Parameters:**
- `logs_dir`: Directory for storing logs (required by `BaseAgent`).
- `model_name`: LLM model to use. If `None`, uses the SDK default model from `get_default_model()`.
- `temperature`: Model temperature.
- `verbose`: Enable verbose output.
- `use_cli_agent`: If `True` (default), uses `create_cli_agent` from `deepagents-cli`. If `False`, uses `create_deep_agent` from the SDK.

**Key Initialization Logic:**
- If `model_name` is `None`, calls `get_default_model()` and applies `temperature` if the model supports it.
- If `LANGSMITH_EXPERIMENT` env var is set, builds an `instruction → example_id` mapping by querying the LangSmith experiment's dataset. This is used later to link traced runs to dataset examples.

#### `name() -> str`
Static method. Returns `"deepagent-harbor"`.

#### `version() -> str | None`
Returns `"0.0.1"`.

#### `setup(environment: BaseEnvironment) -> None`
No-op; required by the `BaseAgent` interface.

#### `_get_formatted_system_prompt(backend: HarborSandbox) -> str`
Queries the sandbox for the current directory (`pwd`) and file listing (`als`), then formats `SYSTEM_MESSAGE` with directory context. Shows up to `_MAX_FILE_LISTING` files.

#### `run(instruction, environment, context) -> None`

**Purpose:** Main task execution entry point called by Harbor.

**Key Logic:**
1. Reads task configuration from `environment.trial_paths.config_path`.
2. Creates a `HarborSandbox` backend.
3. Collects infrastructure metadata via `collect_sandbox_metadata()`.
4. Creates the agent (CLI or SDK) with the formatted system prompt and `auto_approve=True`.
5. Builds a metadata dict including model name, SDK version, and task info.
6. If `LANGSMITH_EXPERIMENT` is set, wraps `deep_agent.ainvoke()` in a `langsmith.trace()` context to link the run to the experiment dataset.
7. Calls `_save_trajectory()` with the result.

#### `_save_trajectory(environment, instruction, result, infra_meta) -> None`

**Purpose:** Converts the agent's message history into an ATIF v1.2 `Trajectory` object and saves it to `logs_dir/trajectory.json`.

**Key Logic:**
- Iterates over `result["messages"]`, converting:
  - `AIMessage` → `Step(source="agent", ...)` with `ToolCall` objects for tool calls
  - `ToolMessage` → `ObservationResult` attached to the preceding `Step`
- Accumulates prompt/completion tokens from `AIMessage.usage_metadata`.
- Saves the trajectory as formatted JSON.

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| `create_deep_agent` | `deepagents` | SDK agent factory |
| `get_default_model` | `deepagents.graph` | Default LLM selection |
| `create_cli_agent` | `deepagents_cli.agent` | CLI agent factory (with skills/memory/shell) |
| `BaseAgent` | `harbor.agents.base` | Harbor agent interface |
| `Trajectory`, `Step`, `ToolCall`, etc. | `harbor.models.trajectories` | ATIF format |
| `langsmith.trace` | `langsmith` | LangSmith tracing context |
| `HarborSandbox` | `deepagents_harbor.backend` | Sandbox backend |
| `InfraMetadata`, `collect_sandbox_metadata` | `deepagents_harbor.metadata` | Infrastructure data collection |
