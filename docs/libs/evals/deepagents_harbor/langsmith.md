# `evals/deepagents_harbor/langsmith.py`

> LangSmith integration for Harbor: datasets, experiments, and feedback.

## Position in the system

This module belongs to the Harbor integration layer. It adapts Harbor environments and LangSmith metadata into interfaces that the deepagents SDK and eval tooling can consume.

## Imports and module-level state

This file imports `asyncio, datetime, hashlib, inspect, json, os, subprocess, sys, tempfile, urllib.parse, uuid, collections.abc` and other helpers.
Module constants worth noticing: `LANGSMITH_API_URL`, `_API_KEY_ENV_VARS`.

## Functions and classes

### `resolve_langsmith_api_key()`

Resolve the LangSmith API key from environment variables. Checks, in order: `LANGSMITH_SANDBOX_API_KEY`, `LANGSMITH_API_KEY`, `LANGCHAIN_API_KEY`. Returns a `(value, env_var_name)` tuple for the first non-empty value, or `None`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `getenv`. It runs synchronously in the caller and returns directly.

### `create_example_id_from_instruction(instruction: str, seed: int=42)`

Create a deterministic UUID from an instruction string. Normalizes the instruction by stripping whitespace and creating a SHA-256 hash, then converting to a UUID for LangSmith compatibility. Args: instruction: The task instruction string to hash. seed: Integer seed to avoid collisions with existing examples. Returns: A UUID string generated from the hash of the normalized instruction. Key arguments are `instruction`, `seed`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `strip`, `digest`, `UUID`, `str`, `to_bytes`, `encode`. It runs synchronously in the caller and returns directly.

### `create_dataset(dataset_name: str, version: str='head', overwrite: bool=False)`

Create a LangSmith dataset from Harbor tasks. Args: dataset_name: Dataset name (used for both Harbor download and LangSmith dataset). version: Harbor dataset version. overwrite: Whether to overwrite cached remote tasks. Key arguments are `dataset_name`, `version`, `overwrite`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `Client`, `Path`, `print`, `create_dataset`, `create_examples`, `mkdtemp`. It runs synchronously in the caller and returns directly.

### `ensure_dataset(dataset_name: str, version: str='head', overwrite: bool=False)`

Create the dataset if it does not already exist. Args: dataset_name: Dataset name to look up in LangSmith. version: Harbor dataset version to use when creating the dataset. overwrite: Whether to overwrite cached remote tasks when creating the dataset. Key arguments are `dataset_name`, `version`, `overwrite`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `Client`, `print`, `read_dataset`, `create_dataset`. It runs synchronously in the caller and returns directly.

### `create_experiment_async(dataset_name: str, experiment_name: str | None=None, *, model: str | None=None, metadata: dict[str, str] | None=None)`

Create a LangSmith experiment session for the given dataset. Args: dataset_name: Name of the LangSmith dataset to create experiment for. experiment_name: Optional name for the experiment (auto-generated if not provided). model: Optional model identifier (e.g. `anthropic:claude-sonnet-4-6`). Used as the suffix in auto-generated experiment names. If not provided, a random suffix will be used to avoid name collisions. metadata: Optional metadata to attach to the experiment session. Diagnostic output is printed to stderr. Returns: A `(name, url)` tuple. The *name* is the experiment session name (suitable for `LANGSMITH_EXPERIMENT`); the *url* is the comparison URL on smith.langchain.com. Raises: LookupError: If the dataset is not found. RuntimeError: If the API request fails. Key arguments are `dataset_name`, `experiment_name`, `model`, `metadata`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `ClientSession`, `print`, `strftime`, `now`, `uuid4`. This is asynchronous and awaits I/O or framework operations before returning.

### `create_experiment(dataset_name: str, experiment_name: str | None=None, *, model: str | None=None, metadata: dict[str, str] | None=None)`

Synchronous wrapper for `create_experiment_async`. Returns: The experiment name. Raises: LookupError: If the dataset is not found. RuntimeError: If the API request fails. Key arguments are `dataset_name`, `experiment_name`, `model`, `metadata`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `run`, `create_experiment_async`. It runs synchronously in the caller and returns directly.

### `add_feedback(job_folder: Path, project_name: str, dry_run: bool=False)`

Add Harbor reward feedback to LangSmith traces. Args: job_folder: Path to the Harbor job folder. project_name: LangSmith project name to search for traces. dry_run: If True, show what would be done without making changes. Key arguments are `job_folder`, `project_name`, `dry_run`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `print`, `Client`, `enumerate`, `iterdir`, `is_dir`, `len`. It runs synchronously in the caller and returns directly.

## Gotchas

Several blocks are async and assume they run inside the owning framework event loop. Calling them from sync code needs the package CLI or an explicit async runner.
