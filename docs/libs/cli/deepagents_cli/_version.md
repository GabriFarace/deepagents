# `_version.py`

## High-Level Purpose

This module contains version information and lightweight URL constants for `deepagents-cli`. It is intentionally minimal with no heavy imports, as it is loaded very early in the startup sequence.

## Constants

| Constant | Value | Description |
|---|---|---|
| `__version__` | `"0.0.47"` | The current CLI package version (managed by release-please) |
| `DOCS_URL` | `"https://docs.langchain.com/oss/python/deepagents/cli"` | URL for CLI documentation |
| `PYPI_URL` | `"https://pypi.org/pypi/deepagents-cli/json"` | PyPI JSON API endpoint for version checks |
| `CHANGELOG_URL` | `"https://github.com/langchain-ai/deepagents/blob/main/libs/cli/CHANGELOG.md"` | Full changelog URL |
| `USER_AGENT` | `f"deepagents-cli/{__version__} update-check"` | User-Agent header for PyPI update requests |

## Usage

These constants are imported across the codebase for:
- Displaying version in `--version` output
- Constructing the welcome banner
- Building update-check HTTP requests
- Opening documentation/changelog URLs from `/docs` and `/changelog` slash commands
