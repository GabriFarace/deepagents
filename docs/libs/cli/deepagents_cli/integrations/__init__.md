# `integrations/__init__.py`

## High-Level Purpose

This is the package initializer for the `integrations` subpackage. It contains only a single docstring and no exports. Submodules must be imported directly.

## Package Description

The `integrations` package provides adapters for external sandbox execution environments used by the deepagents CLI. It defines:

- `sandbox_provider.py` — Abstract `SandboxProvider` interface
- `sandbox_factory.py` — Concrete provider implementations and sandbox lifecycle management

## Usage

```python
from deepagents_cli.integrations.sandbox_factory import create_sandbox, get_default_working_dir
from deepagents_cli.integrations.sandbox_provider import SandboxProvider, SandboxError
```
