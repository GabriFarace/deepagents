# `nvidia_deep_agent/src/backend.py`

## High-Level Purpose

Configures the Modal sandbox backend for the NVIDIA Deep Agent. Defines two Docker images (GPU RAPIDS and CPU-only), handles sandbox lookup/creation based on runtime context, and seeds newly created sandboxes with skill and memory files from the local filesystem.

## Module-Level Configuration

| Variable | Value | Description |
|----------|-------|-------------|
| `MODAL_SANDBOX_NAME` | `"nemotron-deep-agent"` | Base name for the Modal app and sandbox |
| `modal_app` | `modal.App.lookup(...)` | Modal App instance (created if missing) |
| `rapids_image` | RAPIDS 25.02 + numba-cuda>=0.28 + matplotlib + seaborn | GPU image from NVIDIA container registry |
| `cpu_image` | Debian slim + pandas + numpy + scipy + scikit-learn + matplotlib + seaborn | CPU fallback image |
| `SKILLS_DIR` | `Path("skills")` | Local directory containing skill SKILL.md files |
| `MEMORY_FILE` | `Path("src/AGENTS.md")` | Local agent memory/persona file |

**GPU image note:** RAPIDS 25.02 ships `numba-cuda 0.2.0` which has a broken device enumeration; the image upgrades to `numba-cuda>=0.28` to fix `IndexError` in `.to_pandas()` and `.describe()`.

## Functions

### `_seed_sandbox(backend: ModalSandbox) -> None`

**Purpose:** Upload local skill and memory files into a freshly created sandbox.

**Parameters:**
- `backend`: A `ModalSandbox` instance that has just been created.

**Return Value:** None.

**Key Logic:**
1. Iterates over subdirectories in `SKILLS_DIR`; for each that contains a `SKILL.md`, queues the file for upload to `/skills/<skill_dir_name>/SKILL.md` inside the sandbox.
2. If `MEMORY_FILE` exists locally, queues it for upload to `/memory/AGENTS.md`.
3. Creates parent directories inside the sandbox via `backend.execute("mkdir -p ...")`.
4. Uploads all queued files via `backend.upload_files(files)`.

**Production note:** In production, replace local file reads with a storage layer (S3, database, etc.).

---

### `create_backend(runtime) -> ModalSandbox`

**Purpose:** Backend factory function called by `create_deep_agent` to produce the sandbox for each agent run.

**Parameters:**
- `runtime`: Runtime object providing `runtime.context` dict.

**Return Value:** A `ModalSandbox` wrapping the Modal sandbox.

**Key Logic:**
1. Reads `sandbox_type` from `runtime.context` (defaults to `"gpu"`).
2. Attempts `modal.Sandbox.from_name()` to reuse an existing sandbox named `{MODAL_SANDBOX_NAME}-{sandbox_type}`.
3. On `NotFoundError`, creates a new sandbox:
   - GPU: uses `rapids_image` with `gpu="A10G"`, 1-hour timeout, 30-minute idle timeout.
   - CPU: uses `cpu_image` with same timeouts, no GPU.
4. Wraps the sandbox in `ModalSandbox(sandbox=sandbox)`.
5. If sandbox was newly created, calls `_seed_sandbox()` to upload skills and memory.

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| `modal` | `modal` | Modal serverless compute SDK |
| `ModalSandbox` | `langchain_modal` | LangChain wrapper for Modal sandboxes |
| `Path` | `pathlib` | Filesystem path manipulation |
