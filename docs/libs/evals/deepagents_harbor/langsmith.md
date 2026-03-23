# `deepagents_harbor/langsmith.py`

## High-Level Purpose

LangSmith integration for Harbor evals. Provides functions to:
1. Create deterministic example IDs from task instructions (for stable dataset linking).
2. Create and idempotently ensure LangSmith datasets populated with Harbor tasks.
3. Create LangSmith experiment sessions tied to a dataset.
4. Add Harbor reward feedback scores to LangSmith traces after a job completes.

## Module-Level Constants

| Constant | Description |
|----------|-------------|
| `LANGSMITH_API_URL` | LangSmith API base URL, from `LANGSMITH_ENDPOINT` env var or `https://api.smith.langchain.com` |
| `HEADERS` | Auth headers dict using `LANGSMITH_API_KEY` env var |

## Functions

### `create_example_id_from_instruction(instruction: str, seed: int = 42) -> str`

**Purpose:** Create a deterministic, stable UUID from an instruction string.

**Parameters:**
- `instruction`: Task instruction text.
- `seed`: Integer seed (default 42) mixed in to avoid collisions.

**Return Value:** UUID string derived from SHA-256 hash of `seed bytes + normalized instruction`.

---

### `create_dataset(dataset_name: str, version: str = "head", overwrite: bool = False) -> None`

**Purpose:** Download Harbor tasks and create a LangSmith dataset from them.

**Parameters:**
- `dataset_name`: Name for both the Harbor dataset and the LangSmith dataset.
- `version`: Harbor dataset version tag.
- `overwrite`: Whether to overwrite cached downloaded tasks.

**Key Logic:**
1. Downloads the Harbor dataset using `RegistryClientFactory`.
2. For each task, reads `instruction.md`, `task.toml`, and optionally `solution/solve.sh`.
3. Creates a LangSmith dataset and bulk-creates examples with deterministic IDs.

---

### `ensure_dataset(dataset_name: str, version: str = "head", overwrite: bool = False) -> None`

**Purpose:** Create the dataset only if it does not already exist in LangSmith.

---

### `create_experiment(dataset_name: str, experiment_name: str | None = None, *, metadata: dict | None = None) -> str`

**Purpose:** Synchronous wrapper for `create_experiment_async`. Creates a LangSmith experiment session linked to the given dataset.

**Return Value:** The experiment name (auto-generated from dataset name + timestamp if not provided).

---

### `create_experiment_async(dataset_name, experiment_name, *, metadata) -> str`

**Purpose:** Async implementation of `create_experiment`. Looks up the dataset by name, then POSTs to `/sessions` to create an experiment session.

**Return Value:** The experiment name. Diagnostic output (URL, session ID) is printed to stderr.

---

### `add_feedback(job_folder: Path, project_name: str, dry_run: bool = False) -> None`

**Purpose:** Walk a Harbor job folder, find each trial's LangSmith trace, and add a `harbor_reward` feedback score.

**Parameters:**
- `job_folder`: Path to the Harbor job output folder.
- `project_name`: LangSmith project name to search for traces.
- `dry_run`: If `True`, print what would be done without making API calls.

**Key Logic:**
1. Lists all subdirectories of `job_folder` as trial directories.
2. For each trial, calls `_process_trial()` to find and update its trace.
3. Reports counts of success, fallback, skipped, and error outcomes.

---

### `_process_trial(client, trial_dir, project_name, dry_run) -> dict[str, str]`

**Purpose:** Process a single trial: find its LangSmith trace by `trial_name` metadata, extract the reward from `result.json`, check for existing feedback (dedup), and submit `harbor_reward` feedback.

**Return Value:** Status dict with `"status"` and `"message"` keys. Status values: `"success"`, `"fallback"`, `"skipped"`, `"error"`.

---

### `_extract_reward(trial_dir: Path) -> tuple[float, str | None]`

**Purpose:** Read `result.json` from a trial directory and extract the reward value.

**Return Value:** `(reward, comment)` where `comment` is `None` on success or an explanation string when falling back to `0.0`.

**Raises:** `FileNotFoundError` or `ValueError` on missing or malformed files.

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| `langsmith.Client` | `langsmith` | Dataset/experiment/feedback operations |
| `harbor.registry.client` | `harbor` | Downloading Harbor datasets |
| `aiohttp` | `aiohttp` | Async HTTP for experiment creation |
| `toml` | `toml` | Parsing task metadata |
| `hashlib`, `uuid` | stdlib | Deterministic ID generation |
