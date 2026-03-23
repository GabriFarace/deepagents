# Workflow: `release.yml` — Package Release

## Overview

Builds and publishes deepagents packages to PyPI. Supports all packages in the monorepo. The release pipeline enforces several safety checks before publishing: branch validation, SDK pin verification (for CLI), test publication to Test PyPI, and unit test execution against the built wheel.

## Trigger

- `workflow_call` — called automatically from `release-please.yml` when a CLI release PR is merged
- `workflow_dispatch` — manual trigger for any supported package

### `workflow_dispatch` Inputs

| Input | Description |
|---|---|
| `package` | Package to release (dropdown: `deepagents`, `deepagents-cli`, `deepagents-acp`, `deepagents-evals`, `langchain-daytona`, `langchain-modal`, `langchain-quickjs`, `langchain-runloop`) |
| `dangerous-nonmain-release` | Allow releasing from a non-`main` branch (backports/hotfixes only) |
| `dangerous-skip-sdk-pin-check` | Skip CLI SDK pin validation (for intentionally pinning an older SDK) |

## Supported Packages

| Package | Working Directory |
|---|---|
| `deepagents` | `libs/deepagents` |
| `deepagents-cli` | `libs/cli` |
| `deepagents-acp` | `libs/acp` |
| `deepagents-evals` | `libs/evals` |
| `langchain-daytona` | `libs/partners/daytona` |
| `langchain-modal` | `libs/partners/modal` |
| `langchain-quickjs` | `libs/partners/quickjs` |
| `langchain-runloop` | `libs/partners/runloop` |

## Environment Variables

- `PYTHON_VERSION: "3.11"`
- `UV_NO_SYNC: "true"`, `UV_FROZEN: "true"`

## Permissions

- `contents: write` (global) — required for creating GitHub releases

## Jobs

### 1. `setup`

Maps the `package` input to its `working-dir`. Fails for unknown package names.

### 2. `build`

Runs with `contents: read` only (isolated from publish permissions for security).

- Builds the distribution with `uv build`
- Uploads the `dist/` directory as the `dist` artifact
- Extracts `pkg-name` and `version` from `pyproject.toml` via an inline Python script

Only runs from `main` unless `dangerous-nonmain-release` is set.

### 3. `release-notes`

Generates the GitHub release body by:
1. Extracting the current version's section from `CHANGELOG.md` (falls back to `git log` if not found)
2. Collecting contributors from merged PRs since the previous tag:
   - Skips bot accounts and internal contributors (PRs labeled `internal`)
   - Extracts Twitter handles and LinkedIn URLs from PR bodies
   - Formats contributor shoutouts: `@ghuser ([Twitter](url), [LinkedIn](url))`

### 4. `pre-release-checks`

Runs with `contents: read` (no cache, to catch missing dependency bugs):

1. **Verify CLI SDK pin** — for `deepagents-cli`: checks that `libs/cli/pyproject.toml` pins `deepagents==<sdk_version>`. Hard gate; can be bypassed with `dangerous-skip-sdk-pin-check`.
2. **Import dist package** — installs the built wheel into a fresh venv and imports the package
3. **Run unit tests** — installs test dependencies and runs `make test` against the built wheel

### 5. `test-pypi-publish`

Publishes to [Test PyPI](https://test.pypi.org/) using OIDC trusted publishing (no secrets needed). Uses `skip-existing: true` for CI safety. Attestations disabled (temporary workaround).

### 6. `publish`

Publishes to [PyPI](https://pypi.org/) using OIDC trusted publishing. Runs only after `pre-release-checks` and `test-pypi-publish` both succeed.

### 7. `mark-release`

Runs only after `pre-release-checks` and `publish` both succeed:

1. Creates a GitHub release using `ncipollo/release-action@v1`:
   - Tag format: `<pkg-name>==<version>`
   - Attaches the dist artifacts
   - Uses the generated release body
   - Marks as latest only for the `deepagents` (SDK) package
2. Updates the release PR label from `autorelease: pending` to `autorelease: tagged`:
   - First tries to find the PR via the commit SHA
   - Falls back to searching for a merged PR with `autorelease: pending` + `release` labels matching the package name

## PyPI Trusted Publishing Setup

For each package, configure trusted publishing on PyPI at `https://docs.pypi.org/trusted-publishers/adding-a-publisher/` pointing to this repository and workflow.

## Notes

- For the CLI, `workflow_dispatch` is intended only for recovery/hotfix scenarios. Normal releases go through `release-please.yml`.
- The build and publish jobs are intentionally separated to prevent a compromised build step from accessing PyPI or GitHub credentials.
- See `.github/RELEASING.md` for the full release runbook including recovery procedures.
