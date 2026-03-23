# `integrations/sandbox_provider.py`

## High-Level Purpose

This module defines the abstract `SandboxProvider` interface and error types for sandbox backend operations. It provides the contract that all sandbox provider implementations must fulfill (get/create, delete), along with async wrappers that run synchronous operations in a thread pool.

## Classes

### `SandboxError`

**Inherits from:** `Exception`

Base error for sandbox provider operations.

**Properties:**
- `original_exc: BaseException | None` — Returns the original exception that caused this error (`self.__cause__`), if any.

### `SandboxNotFoundError`

**Inherits from:** `SandboxError`

Raised when the requested sandbox cannot be found (e.g., when trying to reuse a `--sandbox-id` that no longer exists).

### `SandboxProvider`

**Inherits from:** `abc.ABC`

Abstract interface for creating and deleting sandbox backends.

**Abstract Methods:**

#### `get_or_create(*, sandbox_id: str | None = None, **kwargs) -> SandboxBackendProtocol`

Get an existing sandbox by ID, or create a new one if `sandbox_id` is `None` or not found.

**Raises:** `SandboxNotFoundError` if `sandbox_id` is provided but doesn't exist.

#### `delete(*, sandbox_id: str, **kwargs) -> None`

Delete a sandbox by its ID.

**Async Wrappers (concrete methods):**

#### `aget_or_create(*, sandbox_id: str | None = None, **kwargs) -> Awaitable[SandboxBackendProtocol]`

Async wrapper around `get_or_create` using `asyncio.to_thread`.

#### `adelete(*, sandbox_id: str, **kwargs) -> Awaitable[None]`

Async wrapper around `delete` using `asyncio.to_thread`.

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `asyncio` | stdlib | `to_thread` for async wrappers |
| `abc.ABC`, `abstractmethod` | stdlib | Abstract base class |
| `SandboxBackendProtocol` | `deepagents.backends.protocol` | Backend protocol type hint |
