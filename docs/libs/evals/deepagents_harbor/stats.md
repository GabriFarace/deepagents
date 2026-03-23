# `deepagents_harbor/stats.py`

## High-Level Purpose

Statistical utilities for reporting eval scores with confidence intervals. Provides Wilson score confidence intervals and minimum detectable effect estimation, both recommended for small-sample binomial proportions.

## Functions

### `wilson_ci(successes: int, total: int, *, z: float = 1.96) -> tuple[float, float]`

**Purpose:** Compute the Wilson score confidence interval for a binomial proportion.

**Parameters:**
- `successes`: Number of successes (e.g., tasks passed).
- `total`: Total number of trials.
- `z`: Z-score for the desired confidence level. Default `1.96` = 95% CI.

**Return Value:** `(lower_bound, upper_bound)` as proportions in `[0, 1]`.

**Key Logic:** Uses the Wilson score formula, which is more accurate than the normal approximation for small samples and proportions near 0 or 1:
```
center = (p + z²/(2n)) / (1 + z²/n)
margin = (z / (1 + z²/n)) * sqrt(p(1-p)/n + z²/(4n²))
```
Returns `(0.0, 0.0)` when `total == 0`.

---

### `format_ci(successes: int, total: int, *, z: float = 1.96) -> str`

**Purpose:** Format a success rate with its Wilson confidence interval as a human-readable string.

**Return Value:** String like `"72.3% [68.1%, 76.2%] (95% CI, n=90)"`.

Returns `"N/A (no trials)"` when `total == 0`.

---

### `min_detectable_effect(total: int, *, z: float = 1.96, p: float = 0.5) -> float`

**Purpose:** Estimate the minimum detectable effect size for a given sample count, at the given confidence level.

**Parameters:**
- `total`: Tasks per run (assumes both runs have the same count).
- `z`: Z-score for confidence level.
- `p`: Assumed base proportion. Defaults to `0.5` (most conservative — maximizes variance).

**Return Value:** MDE as a proportion (e.g., `0.042` = 4.2 percentage points).

**Formula:** `MDE = z * sqrt(2 * p * (1-p) / n)` — the two-sample proportion test standard error.

**Use Case:** If two eval runs score 72% and 78% but the MDE is 14pp, the 6pp difference is statistically indistinguishable from noise.

Returns `1.0` when `total == 0`.

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| `math` | stdlib | `sqrt`, `erf` for statistical computations |
