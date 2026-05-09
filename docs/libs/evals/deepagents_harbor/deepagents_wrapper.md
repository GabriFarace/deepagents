# `evals/deepagents_harbor/deepagents_wrapper.py`

> Harbor benchmark adapter that runs a deepagents SDK or CLI agent inside a Harbor environment and saves ATIF trajectories.

## Position in the system

This module belongs to the Harbor integration layer. It adapts Harbor environments and LangSmith metadata into interfaces that the deepagents SDK and eval tooling can consume.

## Imports and module-level state

This file imports `__future__, importlib.metadata, json, logging, os, uuid, datetime, typing, deepagents, deepagents_cli.agent, dotenv, harbor.agents.base` and other helpers.
Module constants worth noticing: `_MAX_FILE_LISTING`, `SYSTEM_MESSAGE`.

## Functions and classes

### `DeepAgentsWrapper`

Harbor agent implementation using LangChain Deep Agents. Wraps Deep Agents to execute tasks in Harbor environments. This class inherits from `BaseAgent` and is the main object for this part of the module.

#### `DeepAgentsWrapper.__init__(self, logs_dir: Path, model_name: str, temperature: float=0.0, verbose: bool=True, use_cli_agent: bool=True, openrouter_provider: str | None=None, *args: Any, openrouter_allow_fallbacks: bool=False, **kwargs: Any)`

Initialize Deep AgentsWrapper. Args: logs_dir: Directory for storing logs. model_name: Name of the LLM model to use. temperature: Temperature setting for the model. verbose: Enable verbose output. use_cli_agent: If `True`, use `create_cli_agent` from `deepagents-cli` (default). If `False`, use `create_deep_agent` from the SDK. openrouter_provider: Pin OpenRouter routing to one or more providers. Accepts a single name (e.g. `"MiniMax"`) or a comma-separated allowlist (e.g. `"MiniMax,Fireworks"`), fed into OpenRouter's `provider.only` field. Requires an `openrouter:` model prefix. openrouter_allow_fallbacks: When `False` (default), OpenRouter will only route to a provider in `openrouter_provider` and hard-fail otherwise (strict allowlist). When `True`, the listed providers are preferred but OpenRouter may fall back to any other provider hosting the model. Has no effect on its own and requires `openrouter_provider` to be set. Raises: ValueError: If `model_name` is empty/whitespace, if `openrouter_provider` is set without an `openrouter:` prefix, if `openrouter_provider` is non-empty but contains no provider names (e.g. only commas/whitespace), or if `openrouter_allow_fallbacks` is `True` without `openrouter_provider`. Key arguments are `logs_dir`, `model_name`, `temperature`, `verbose`, `use_cli_agent`, `openrouter_provider`, `openrouter_allow_fallbacks`. It mutates `self._model_name`, `self._model`, `self._temperature`, `self._verbose`, `self._use_cli_agent`, `self._langsmith_run_id`. Internally it delegates to `init_chat_model`, `ValueError`, `strip`, `super`, `startswith`, `Client`. It runs synchronously in the caller and returns directly.

#### `DeepAgentsWrapper.name()`

Return the agent name identifier. It does not keep durable module state; effects come from returned values or delegated calls. It runs synchronously in the caller and returns directly.

#### `DeepAgentsWrapper.setup(self, environment: BaseEnvironment)`

Setup the agent with the given environment. Args: environment: Harbor environment (Docker, Modal, etc.) Key arguments are `environment`. It does not keep durable module state; effects come from returned values or delegated calls. This is asynchronous and awaits I/O or framework operations before returning.

#### `DeepAgentsWrapper.version(self)`

The version of the agent. It does not keep durable module state; effects come from returned values or delegated calls. It runs synchronously in the caller and returns directly.

#### `DeepAgentsWrapper.run(self, instruction: str, environment: BaseEnvironment, context: AgentContext)`

Execute the deep agent on the given instruction. Args: instruction: The task to complete environment: Harbor environment (Docker, Modal, etc.) context: Context to populate with metrics Key arguments are `instruction`, `environment`, `context`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `loads`, `HarborSandbox`, `update`, `get`, `read_text`, `isinstance`. This is asynchronous and awaits I/O or framework operations before returning.

## System prompts and tool descriptions

### `SYSTEM_MESSAGE`

```text
You are an autonomous agent executing tasks in a sandboxed environment. Follow these instructions carefully.

## WORKING DIRECTORY & ENVIRONMENT CONTEXT

Your current working directory is:
{current_directory}

{file_listing_header}
{file_listing}

**IMPORTANT**: This directory information is provided for your convenience at the start of the task. You should:
- Use this information to understand the initial environment state
- Avoid redundantly calling `ls` or similar commands just to list the same directory
- Only use file listing commands if you need updated information (after creating/deleting files) or need to explore subdirectories
- Work in the /app directory unless explicitly instructed otherwise
```

## Flow walk-through

1. `__init__()` validates the model configuration, optionally pins OpenRouter providers, initializes a LangChain chat model, and preloads a LangSmith instruction-to-example mapping when `LANGSMITH_EXPERIMENT` is set.
2. `run()` reads the Harbor trial config, wraps the Harbor environment in `HarborSandbox`, and tries to collect host/sandbox metadata. Metadata failures are logged but never fail the trial.
3. The wrapper then chooses between the CLI agent factory and the SDK `create_deep_agent()` path. Both paths receive the Harbor backend plus the formatted sandbox-aware `SYSTEM_MESSAGE`; the CLI path also disables HITL and CLI skills for benchmark determinism.
4. If `LANGSMITH_EXPERIMENT` is configured, the invocation runs inside a LangSmith `trace()` linked to the matching dataset example. Otherwise, the same metadata is attached directly to the LangGraph runnable config.
5. `_save_trajectory()` converts LangChain `HumanMessage`, `AIMessage`, and `ToolMessage` objects into Harbor ATIF `Step`, `ToolCall`, `Observation`, and `FinalMetrics` records, then writes `trajectory.json` in the trial logs directory.

## Gotchas

The directory listing in `SYSTEM_MESSAGE` is intentionally a snapshot from the start of the task. Agents should still inspect the filesystem after they mutate it, but the prompt discourages wasting a first tool call on an unchanged `ls`.

Several blocks are async and assume they run inside the owning framework event loop. Calling them from sync code needs the package CLI or an explicit async runner.
