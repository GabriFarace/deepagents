# `evals/deepagents_harbor/failure.py`

> Failure classification for eval trial results.

## Position in the system

This module belongs to the Harbor integration layer. It adapts Harbor environments and LangSmith metadata into interfaces that the deepagents SDK and eval tooling can consume.

## Imports and module-level state

This file imports `__future__, json, logging, re, enum, typing`.
Module constants worth noticing: `_OOM_EXIT_CODES`, `_TIMEOUT_EXIT_CODES`, `_OOM_PATTERNS`, `_TIMEOUT_PATTERNS`, `_SANDBOX_PATTERNS`.

## Functions and classes

### `extract_exit_codes(trajectory_json: str)`

Extract non-zero exit codes from ATIF trajectory observation results. Parses the trajectory JSON structurally and only searches observation content (tool output) for exit code patterns, avoiding false positives from model-generated text that discusses exit codes. Args: trajectory_json: Raw JSON text of the ATIF trajectory. Returns: List of non-zero exit codes found in observation results. Key arguments are `trajectory_json`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `extend`. It runs synchronously in the caller and returns directly.

### `classify_failure(*, exception_text: str | None=None, exit_codes: list[int] | None=None)`

Classify a trial failure as infrastructure or capability. Uses exit codes and exception text to determine whether a failure was caused by infrastructure issues (OOM, timeout, sandbox crash) or by the model's capability. Pattern matching is restricted to `exception_text` only (structured, controlled output) to avoid false positives from model-generated content in trajectories. Args: exception_text: Content of `exception.txt` if present. exit_codes: List of non-zero exit codes observed during the trial. Returns: The determined failure category. Key arguments are `exception_text`, `exit_codes`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `lower`, `any`. It runs synchronously in the caller and returns directly.

### `FailureCategory`

Classification of trial failures. Distinguishes infrastructure failures from model capability failures. This class inherits from `Enum` and is the main object for this part of the module.

#### `FailureCategory.is_infrastructure(self)`

Whether this failure is caused by infrastructure rather than model capability. It does not keep durable module state; effects come from returned values or delegated calls. It runs synchronously in the caller and returns directly.

## Gotchas

Most behavior is delegated through framework protocols, so preserve method signatures when editing even if an argument appears unused locally.
