# `libs/` — Library Packages

This directory contains all independently versioned Python packages that make up the deepagents framework.

## Packages

| Package | Version | Description |
|---|---|---|
| [`deepagents`](deepagents/README.md) | 0.5.6 | Core SDK — agent factory, backends, middleware |
| [`deepagents-cli`](cli/README.md) | 0.0.47 | Interactive TUI and CLI tool |
| [`deepagents-acp`](acp/README.md) | (see package) | Agent Client Protocol (ACP) adapter |
| [`deepagents-evals`](evals/README.md) | (see package) | Evaluation suite and Harbor benchmarks |
| Partners | various | Sandbox provider integrations |

## Package Dependency Graph

```
deepagents-cli ──depends on──► deepagents (SDK)
                    │
                    └──depends on──► deepagents-acp (optional, --acp mode)

deepagents-acp ──depends on──► deepagents (SDK)

deepagents-evals ──depends on──► deepagents (SDK)
                                 deepagents-cli (for harbor)

Partners (daytona, modal, quickjs, runloop)
    └──implement──► deepagents.backends.BackendProtocol
```

## Partner Packages

Each partner package adds a sandbox integration for a specific cloud/local execution environment:

| Package | Description |
|---|---|
| [`deepagents-daytona`](partners/daytona/README.md) | Daytona cloud sandbox |
| [`deepagents-modal`](partners/modal/README.md) | Modal serverless sandbox |
| [`deepagents-quickjs`](partners/quickjs/README.md) | QuickJS in-process JS sandbox |
| [`deepagents-runloop`](partners/runloop/README.md) | Runloop cloud sandbox |

## Architecture Roles

- **`deepagents` SDK** — The foundation. Defines the agent graph, backend protocol, and middleware system. Every other package depends on it.
- **`deepagents-cli`** — User-facing layer. Wraps the SDK with a Textual TUI, session management, and MCP integration.
- **`deepagents-acp`** — Protocol adapter. Exposes an SDK agent as an ACP-compliant server for remote clients.
- **`deepagents-evals`** — Quality assurance. Runs the agent against benchmarks across multiple model providers.
- **Partner packages** — Pluggable execution environments. Implement `BackendProtocol` for different sandboxes.
