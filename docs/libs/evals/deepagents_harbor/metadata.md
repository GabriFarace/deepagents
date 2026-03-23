# `deepagents_harbor/metadata.py`

## High-Level Purpose

Captures infrastructure metadata (CPU, memory, OS, Python version) from both the orchestrator host and inside the Harbor sandbox at the start of a trial. This data is attached to trajectories to support post-hoc noise analysis of eval results.

## Classes

### `SandboxLike`

**Purpose:** Structural `Protocol` that any object satisfying the metadata collection interface must implement.

**Definition:** `@runtime_checkable`

**Required attributes/methods:**
- `environment: Any` — The Harbor environment instance.
- `async aexecute(command: str, *, timeout: int | None = None) -> Any` — Execute a shell command.

Both `HarborSandbox` and test fakes satisfy this protocol.

---

### `InfraMetadata`

**Purpose:** Dataclass holding infrastructure metadata captured at trial execution time.

**Definition:** `@dataclass`

**Attributes:**

| Attribute | Type | Description |
|-----------|------|-------------|
| `host_platform` | `str` | Orchestrator host OS and architecture string |
| `host_python_version` | `str` | Python version on the orchestrator |
| `sandbox_type` | `str` | Class name of the Harbor environment type |
| `sandbox_cpu_count` | `int \| None` | Number of CPUs in the sandbox |
| `sandbox_memory_total_mb` | `int \| None` | Total sandbox memory in MB |
| `sandbox_memory_available_mb` | `int \| None` | Available sandbox memory in MB |
| `sandbox_os` | `str` | Sandbox OS (`uname -s -r`) |
| `timestamp_utc` | `str` | ISO timestamp at collection time |
| `concurrency_env` | `str` | Value of `HARBOR_CONCURRENCY` env var |
| `resource_config` | `dict` | Additional resource config (extensible) |

**Method:**

##### `to_dict() -> dict[str, Any]`
Serializes to a plain dict using `dataclasses.asdict()`.

## Functions

### `collect_host_metadata() -> dict[str, str]`

**Purpose:** Collect non-sandbox metadata from the orchestrator host.

**Return Value:** Dict with `"host_platform"` (from `platform.platform()`) and `"host_python_version"`.

---

### `collect_sandbox_metadata(backend: SandboxLike) -> InfraMetadata`

**Purpose:** Collect infrastructure metadata by running lightweight shell commands inside the sandbox.

**Parameters:**
- `backend`: Any object satisfying `SandboxLike` (typically `HarborSandbox`).

**Return Value:** Populated `InfraMetadata` instance.

**Key Logic:** Each data point is collected independently with a 10-second timeout and wrapped in a broad `except Exception` so that metadata collection never aborts a trial. Uses:
- `nproc` (Linux) or `sysctl -n hw.ncpu` (macOS) for CPU count
- `/proc/meminfo` (Linux) or `sysctl -n hw.memsize` (macOS) for total memory
- `/proc/meminfo MemAvailable` for available memory (Linux only)
- `uname -s -r` for OS info

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| `platform` | stdlib | Host platform info |
| `dataclasses` | stdlib | `@dataclass` and `asdict` |
| `typing.Protocol` | stdlib | Structural type checking |
| `datetime` | stdlib | UTC timestamp |
| `os` | stdlib | Environment variable access |
