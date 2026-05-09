# `evals/deepagents_harbor/stats.py`

> Statistical utilities for eval score reporting.

## Position in the system

This module belongs to the Harbor integration layer. It adapts Harbor environments and LangSmith metadata into interfaces that the deepagents SDK and eval tooling can consume.

## Imports and module-level state

This file imports `__future__, math`.

## Functions and classes

### `wilson_ci(successes: int, total: int, *, z: float=1.96)`

Compute Wilson score confidence interval for a binomial proportion. More accurate than the normal approximation for small samples and proportions near 0 or 1. Recommended by Anthropic's infrastructure noise research for eval score reporting. Args: successes: Number of successes (e.g., passed tasks). total: Total number of trials. z: Z-score for desired confidence level (1.96 = 95% CI). Returns: Tuple of `(lower_bound, upper_bound)` as proportions in `[0, 1]`. Key arguments are `successes`, `total`, `z`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `sqrt`, `max`, `min`. It runs synchronously in the caller and returns directly.

### `format_ci(successes: int, total: int, *, z: float=1.96)`

Format a success rate with Wilson confidence interval. Args: successes: Number of successes. total: Total number of trials. z: Z-score for desired confidence level. Returns: Formatted string like `'72.3% [68.1%, 76.2%] (95% CI, n=90)'`. Key arguments are `successes`, `total`, `z`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `wilson_ci`, `erf`, `sqrt`. It runs synchronously in the caller and returns directly.

### `min_detectable_effect(total: int, *, z: float=1.96, p: float=0.5)`

Estimate minimum detectable effect size for a given sample count. The MDE is the smallest difference in success rates between two runs that can be considered statistically significant. If two runs score 72% and 78% but the MDE is 14pp, that 6pp gap is indistinguishable from noise at the chosen confidence level. Derived from the standard error of the difference between two independent proportions: `MDE = z * sqrt(2 * p * (1-p) / n)`. Assumes equal sample sizes in both runs. Defaults to `p=0.5` because that maximizes `p*(1-p)`, giving the most conservative (widest) estimate. Args: total: Number of tasks per run (assumes both runs have the same count). z: Z-score for desired confidence level (1.96 = 95% CI). p: Assumed base proportion. 0.5 is the conservative default since it maximizes variance. Returns: Minimum detectable difference as a proportion (e.g., `0.042 = 4.2pp`). Key arguments are `total`, `z`, `p`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `sqrt`. It runs synchronously in the caller and returns directly.

## Gotchas

Most behavior is delegated through framework protocols, so preserve method signatures when editing even if an argument appears unused locally.
