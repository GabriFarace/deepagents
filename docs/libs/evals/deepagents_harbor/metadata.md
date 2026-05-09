# `evals/deepagents_harbor/metadata.py`

> Infrastructure metadata collection for eval trials.

## Position in the system

This module belongs to the Harbor integration layer. It adapts Harbor environments and LangSmith metadata into interfaces that the deepagents SDK and eval tooling can consume.

## Imports and module-level state

This file imports `__future__, logging, os, platform, dataclasses, datetime, typing`.

## Functions and classes

### `collect_host_metadata()`

Collect metadata from the orchestrator host (non-sandbox). Returns: Dictionary with host platform and Python version. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `platform`, `python_version`. It runs synchronously in the caller and returns directly.

### `collect_sandbox_metadata(backend: SandboxLike)`

Collect infrastructure metadata from inside the sandbox environment. Runs lightweight shell commands to capture CPU, memory, and OS info. Designed to be called once at the start of a trial run. Args: backend: Harbor sandbox backend to query. Returns: Populated infrastructure metadata. Key arguments are `backend`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `InfraMetadata`, `collect_host_metadata`, `isdigit`, `isoformat`, `get`, `aexecute`. This is asynchronous and awaits I/O or framework operations before returning.

### `SandboxLike`

Structural protocol for objects usable by `collect_sandbox_metadata`. Any object exposing an `environment` attribute and an async `aexecute` method satisfies this protocol — including `HarborSandbox` and test fakes. This class inherits from `Protocol` and is the main object for this part of the module.

#### `SandboxLike.aexecute(self, command: str, *, timeout: int | None=None)`

Handles the `aexecute` step for this module. Key arguments are `command`, `timeout`. It does not keep durable module state; effects come from returned values or delegated calls. This is asynchronous and awaits I/O or framework operations before returning.

### `InfraMetadata`

Infrastructure metadata captured at trial execution time. Enables post-hoc analysis of infrastructure noise by recording the execution environment details alongside eval results. This class inherits from `object` and is the main object for this part of the module.

#### `InfraMetadata.to_dict(self)`

Serialize to dictionary for JSON storage. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `asdict`. It runs synchronously in the caller and returns directly.

## Gotchas

Several blocks are async and assume they run inside the owning framework event loop. Calling them from sync code needs the package CLI or an explicit async runner.
