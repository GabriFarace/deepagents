# `evals/deepagents_harbor/langsmith_environment.py`

> Harbor environment backed by LangSmith sandboxes.

## Position in the system

This module belongs to the Harbor integration layer. It adapts Harbor environments and LangSmith metadata into interfaces that the deepagents SDK and eval tooling can consume.

## Imports and module-level state

This file imports `__future__, asyncio, re, shlex, pathlib, typing, dockerfile_parse, harbor.environments.base, harbor.models.trial.paths, harbor.utils.logger, langsmith.sandbox, deepagents_harbor.langsmith`.
Module constants worth noticing: `_DEFAULT_EXEC_TIMEOUT_SEC`, `_BYTES_PER_MB`, `_MAX_NAME_LEN`.

## Functions and classes

### `LangSmithEnvironment`

Harbor environment backed by LangSmith sandboxes. Uses `--environment-import-path` because harbor's `EnvironmentType` enum does not include `langsmith` yet. Example: harbor run --environment-import-path \ deepagents_harbor.langsmith_environment:LangSmithEnvironment ... The environment reads the task's Dockerfile to extract the base image, ensures a LangSmith snapshot exists for that image (building it on first use), and boots a sandbox from it with the task's resource config applied at `create_sandbox` time. Snapshots are keyed purely by image and are **shared across trials**. They are intentionally never deleted on `stop()` — rebuilding the same image for every trial would be wasteful, and the LangSmith workspace is the canonical place to prune them manually. Per-trial vCPU / memory / filesystem sizing lives on `create_sandbox`, not on the snapshot, so sharing is safe. This class inherits from `BaseEnvironment` and is the main object for this part of the module.

#### `LangSmithEnvironment.__init__(self, environment_dir: Path, environment_name: str, session_id: str, trial_paths: TrialPaths, task_env_config: EnvironmentConfig, **kwargs: Any)`

Initialize a LangSmith harbor environment. Args: environment_dir: Path to the task's environment directory. environment_name: Logical name for this environment. session_id: Unique trial session identifier. trial_paths: Local paths for trial artifacts. task_env_config: Resource and network configuration. **kwargs: Forwarded to `BaseEnvironment` (e.g. `logger`, `override_cpus`, `override_memory_mb`). Key arguments are `environment_dir`, `environment_name`, `session_id`, `trial_paths`, `task_env_config`. It mutates `self._session_id`, `self._sandbox`, `self._client`, `self._snapshot_name`, `self._default_cwd`. Internally it delegates to `super`. It runs synchronously in the caller and returns directly.

#### `LangSmithEnvironment.type()`

Not applicable — this environment is loaded via import path. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `NotImplementedError`. It runs synchronously in the caller and returns directly.

#### `LangSmithEnvironment.is_mounted(self)`

Whether the environment mounts host logging directories. It does not keep durable module state; effects come from returned values or delegated calls. It runs synchronously in the caller and returns directly.

#### `LangSmithEnvironment.supports_gpus(self)`

Whether LangSmith sandboxes support GPU allocation. It does not keep durable module state; effects come from returned values or delegated calls. It runs synchronously in the caller and returns directly.

#### `LangSmithEnvironment.can_disable_internet(self)`

Whether LangSmith sandboxes support network isolation. It does not keep durable module state; effects come from returned values or delegated calls. It runs synchronously in the caller and returns directly.

#### `LangSmithEnvironment.start(self, force_build: bool)`

Provision a LangSmith sandbox from the task's Dockerfile image. Ensures a shared snapshot exists for the resolved image, then boots a sandbox from it with per-trial resource limits applied at `create_sandbox` time. Args: force_build: Accepted for interface compatibility but unused. Snapshots are shared across trials; the first trial to touch a given image builds it, every subsequent trial reuses it. Key arguments are `force_build`. It mutates `self._client`, `self._snapshot_name`, `self._sandbox`, `self._default_cwd`. Internally it delegates to `resolve_langsmith_api_key`, `AsyncSandboxClient`, `info`, `ValueError`, `create_sandbox`, `run`. This is asynchronous and awaits I/O or framework operations before returning.

#### `LangSmithEnvironment.stop(self, delete: bool)`

Tear down the LangSmith sandbox. The backing snapshot is **never** deleted here: snapshots are shared across trials and are only cleaned up manually in the LangSmith workspace. Args: delete: If True, delete the sandbox before closing the client. Key arguments are `delete`. It mutates `self._sandbox`, `self._client`, `self._snapshot_name`, `self._default_cwd`. Internally it delegates to `info`, `aclose`, `delete_sandbox`, `warning`. This is asynchronous and awaits I/O or framework operations before returning.

#### `LangSmithEnvironment.exec(self, command: str, cwd: str | None=None, env: dict[str, str] | None=None, timeout_sec: int | None=None)`

Execute a command inside the LangSmith sandbox. When `cwd` is not provided, defaults to the container's `WORKDIR` (resolved at `start()`) rather than LangSmith's dataplane default of `/`. This preserves the semantics that terminal-bench verifier scripts assume when they probe `$PWD`. Args: command: Shell command string to execute. cwd: Working directory for command execution. Overrides the detected default when provided. env: Environment variables to set. timeout_sec: Timeout in seconds. Returns: Execution result containing stdout, stderr, and return code. Key arguments are `command`, `cwd`, `env`, `timeout_sec`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `ExecResult`, `run`. This is asynchronous and awaits I/O or framework operations before returning.

#### `LangSmithEnvironment.upload_file(self, source_path: Path | str, target_path: str)`

Upload a local file to the sandbox. Args: source_path: Local file path. target_path: Destination path inside the sandbox. Key arguments are `source_path`, `target_path`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `str`, `to_thread`, `write`, `Path`, `run`, `quote`. This is asynchronous and awaits I/O or framework operations before returning.

#### `LangSmithEnvironment.upload_dir(self, source_dir: Path | str, target_dir: str)`

Upload a local directory to the sandbox recursively. Args: source_dir: Local directory path. target_dir: Destination directory inside the sandbox. Key arguments are `source_dir`, `target_dir`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `Path`, `to_thread`, `relative_to`, `str`, `upload_file`. This is asynchronous and awaits I/O or framework operations before returning.

#### `LangSmithEnvironment.download_file(self, source_path: str, target_path: Path | str)`

Download a file from the sandbox to the local machine. Args: source_path: File path inside the sandbox. target_path: Local destination path. Key arguments are `source_path`, `target_path`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `Path`, `mkdir`, `read`, `to_thread`. This is asynchronous and awaits I/O or framework operations before returning.

#### `LangSmithEnvironment.download_dir(self, source_dir: str, target_dir: Path | str)`

Download a directory from the sandbox to the local machine. Args: source_dir: Directory path inside the sandbox. target_dir: Local destination directory. Key arguments are `source_dir`, `target_dir`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `Path`, `to_thread`, `exec`, `warning`, `info`, `relative_to`. This is asynchronous and awaits I/O or framework operations before returning.

## Gotchas

Several blocks are async and assume they run inside the owning framework event loop. Calling them from sync code needs the package CLI or an explicit async runner.
