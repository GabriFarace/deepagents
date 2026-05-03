# libs/ — Package Overview

deepagents is a Python monorepo managed with `uv`. The `libs/` directory contains all publishable packages.

---

## Package Dependency Graph

```
deepagents (SDK)          ← core; no CLI or TUI deps
    ↑
    ├── deepagents-cli    ← wraps SDK; adds Textual TUI + LangGraph server
    ├── deepagents-acp    ← wraps SDK; exposes agent as ACP server
    └── deepagents-evals  ← tests SDK behavior via CLI + Harbor benchmarks

libs/partners/*
    ├── daytona/          ← implements BackendProtocol via Daytona cloud sandbox
    ├── modal/            ← implements BackendProtocol via Modal serverless
    ├── quickjs/          ← implements BackendProtocol via in-process QuickJS
    └── runloop/          ← implements BackendProtocol via Runloop cloud sandbox
```

The SDK defines `BackendProtocol`. Every sandbox provider implements it independently, so the SDK has zero runtime dependencies on any specific cloud provider.

---

## Package Descriptions

| Package | Directory | Purpose |
|---|---|---|
| `deepagents` | `libs/deepagents/` | Core SDK — `create_deep_agent()`, backends, middleware |
| `deepagents-cli` | `libs/cli/` | CLI tool — Textual TUI, LangGraph server subprocess, sessions |
| `deepagents-acp` | `libs/acp/` | ACP adapter — exposes any compiled agent as an ACP server |
| `deepagents-evals` | `libs/evals/` | Evaluation suite — Harbor integration, benchmark metrics |
| `deepagents-daytona` | `libs/partners/daytona/` | Daytona cloud sandbox backend |
| `deepagents-modal` | `libs/partners/modal/` | Modal serverless sandbox backend |
| `deepagents-quickjs` | `libs/partners/quickjs/` | In-process QuickJS sandbox backend |
| `deepagents-runloop` | `libs/partners/runloop/` | Runloop cloud sandbox backend |

---

## Choosing Between Packages

**I want to build a custom agent programmatically:** use `deepagents` (SDK) directly. Call `create_deep_agent()` and invoke or stream the resulting `CompiledStateGraph`.

**I want an interactive chat interface:** use `deepagents-cli` — it provides a full TUI and handles the LangGraph server lifecycle automatically.

**I want to expose my agent to an ACP client:** wrap it with `deepagents-acp`. The ACP adapter handles session management and streaming.

**I need to run untrusted code safely:** add one of the `partners/*` sandbox backends. They all implement `BackendProtocol`, so you can swap them without touching agent logic.

**I want to measure agent quality:** use `deepagents-evals` to run the benchmark suite, or wire up custom evals via its Harbor integration.

---

## See Also

- [deepagents SDK docs](deepagents/deepagents/README.md)
- [CLI docs](cli/deepagents_cli/README.md)
- [ACP docs](acp/README.md)
- [Evals docs](evals/README.md)
