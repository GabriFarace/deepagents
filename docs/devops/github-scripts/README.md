# GitHub Scripts

Scripts in `.github/scripts/` support the CI/CD workflows. There are Python scripts for validation and evaluation logic, and JavaScript files for PR/issue labeling logic.

## Python Scripts

| Script | Purpose | Called By |
|---|---|---|
| [`aggregate_evals.py`](./aggregate_evals.md) | Aggregates per-model eval reports into summary tables and JSON | `evals.yml` (`aggregate` job) |
| [`check_extras_sync.py`](./check_extras_sync.md) | Verifies optional extras version constraints match required deps in `pyproject.toml` | `check_extras_sync.yml`, pre-commit hook |
| [`check_version_equality.py`](./check_version_equality.md) | Verifies `pyproject.toml` version matches `_version.py` for SDK and CLI | `check_versions.yml`, pre-commit hook |
| [`models.py`](./models.md) | Single source of truth for all model definitions; resolves presets to GitHub Actions matrix JSON | `evals.yml`, `harbor.yml` |

## JavaScript Files

| File | Purpose | Called By |
|---|---|---|
| [`pr-labeler.js`](./pr-labeler.md) | Shared helper module for all PR and issue labeling logic | `pr_labeler.yml`, `pr_labeler_backfill.yml`, `tag-external-issues.yml` |
| [`pr-labeler-config.json`](./pr-labeler-config.md) | JSON configuration for size thresholds, file rules, type/scope-to-label mappings | `pr-labeler.js` |

## Design Principles

### Python Scripts

- All Python scripts are self-contained with no third-party dependencies beyond what's needed for their specific task (`tabulate` for `aggregate_evals.py`, standard library for all others).
- Scripts are callable directly from CI workflow steps and also from pre-commit hooks, ensuring local and CI behavior is identical.
- Exit codes follow Unix convention: `0` for success, `1` for failure.

### JavaScript/JSON (Labeling)

- All labeling logic is centralized in `pr-labeler.js` and `pr-labeler-config.json`.
- Workflows call `require('./.github/scripts/pr-labeler.js').loadAndInit(github, owner, repo, core)` to access the shared helpers.
- This avoids code duplication between `pr_labeler.yml`, `pr_labeler_backfill.yml`, and `tag-external-issues.yml`.

## Adding a New Package to Labeling

To add labeling support for a new partner package:
1. Add a `fileRules` entry to `pr-labeler-config.json` with the package's path prefix
2. Add the corresponding scope to `scopeToLabel` if it has a commit scope
3. Update `pr_labeler_backfill.yml` documentation and consider running the backfill
