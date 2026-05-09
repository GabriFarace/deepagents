# `libs/cli/deepagents_cli/integrations/`

> Sandbox provider abstraction and factory logic for CLI-created backends.

## Position in the system

The CLI can run with different sandbox providers. `sandbox_provider.py` defines
the shared interface and exceptions; `sandbox_factory.py` imports optional
provider SDKs lazily, validates dependency availability, creates sandboxes, and
returns the working directory metadata the rest of the CLI expects.

## Files

- [`sandbox_provider.md`](./sandbox_provider.md) documents the provider protocol
  and sandbox-specific exceptions.
- [`sandbox_factory.md`](./sandbox_factory.md) documents provider lookup,
  dependency checks, setup commands, and concrete LangSmith, Daytona, Modal,
  Runloop, and AgentCore provider adapters.

## Gotchas

Provider dependencies are optional extras. Keep imports lazy and error messages
actionable so users who do not use a provider do not pay its import cost.
