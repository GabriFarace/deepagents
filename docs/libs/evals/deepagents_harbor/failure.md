# `deepagents_harbor/failure.py`

## High-Level Purpose

Classifies trial failures as infrastructure-related (OOM, timeout, sandbox crash) or model-capability-related. Used to filter out noisy infrastructure failures when analyzing eval results.

## Classes

### `FailureCategory`

**Purpose:** Enum enumerating the possible failure categories for a trial.

**Inherits from:** `enum.Enum`

| Member | Value | Description |
|--------|-------|-------------|
| `CAPABILITY` | `"capability"` | Model produced wrong answer or logic error |
| `INFRA_OOM` | `"infra_oom"` | Out-of-memory kill (exit code 137 / SIGKILL) |
| `INFRA_TIMEOUT` | `"infra_timeout"` | Command or task exceeded time limit (exit code 124) |
| `INFRA_SANDBOX` | `"infra_sandbox"` | Sandbox crash, network failure, or environment error |
| `UNKNOWN` | `"unknown"` | Could not determine failure category |

**Property:**

##### `is_infrastructure -> bool`
Returns `True` if this failure is `INFRA_OOM`, `INFRA_TIMEOUT`, or `INFRA_SANDBOX`.

## Module-Level Pattern Tables

| Variable | Description |
|----------|-------------|
| `_OOM_EXIT_CODES` | `{137}` — Linux OOM killer exit code |
| `_TIMEOUT_EXIT_CODES` | `{124}` — GNU coreutils `timeout` exit code |
| `_OOM_PATTERNS` | Case-insensitive substrings in exception text indicating OOM |
| `_TIMEOUT_PATTERNS` | Case-insensitive substrings indicating timeout |
| `_SANDBOX_PATTERNS` | Case-insensitive substrings indicating sandbox/network failure |

## Functions

### `extract_exit_codes(trajectory_json: str) -> list[int]`

**Purpose:** Extract non-zero exit codes from an ATIF trajectory's observation results (tool outputs).

**Parameters:**
- `trajectory_json`: Raw JSON text of the ATIF trajectory.

**Return Value:** List of non-zero integer exit codes.

**Key Logic:** Parses the JSON structurally and searches only within observation result `content` fields to avoid false positives from model-generated text discussing exit codes. Falls back to raw regex scanning if JSON parsing fails.

---

### `classify_failure(*, exception_text: str | None = None, exit_codes: list[int] | None = None) -> FailureCategory`

**Purpose:** Classify a trial failure given optional exception text and exit codes.

**Parameters:**
- `exception_text`: Content of `exception.txt` from the trial directory, if present.
- `exit_codes`: Non-zero exit codes observed during the trial.

**Return Value:** The best-matching `FailureCategory`.

**Priority Order:**
1. Exit codes checked first: `137` → `INFRA_OOM`; `124` → `INFRA_TIMEOUT`.
2. Exception text pattern matching: OOM patterns → `INFRA_OOM`; timeout patterns → `INFRA_TIMEOUT`; sandbox patterns → `INFRA_SANDBOX`.
3. Exception present but no infra patterns matched → `UNKNOWN`.
4. No exception, no infra exit codes → `CAPABILITY`.

---

### `_extract_observation_texts(trajectory_json: str) -> list[str] | None`
Internal helper. Parses ATIF JSON and returns the `content` strings from all observation results. Returns `None` if parsing fails.

### `_extract_exit_codes_raw(text: str) -> list[int]`
Internal helper. Regex-scans text for `exit code: NNN` patterns and returns non-zero values.

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| `json`, `re`, `logging` | stdlib | Parsing and pattern matching |
| `enum.Enum` | stdlib | Enum base class |
