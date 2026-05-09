# `libs/`

> Package map for the monorepo. The SDK sits at the center; the CLI, ACP
> adapter, evals, REPL, and partner packages wrap or exercise it.

## Position in the system

```
deepagents SDK
  ├─ consumed by deepagents-cli
  ├─ exposed through deepagents-acp
  ├─ tested and benchmarked by deepagents-evals
  ├─ extended by partner sandbox backends
  └─ shares interpreter pieces with langchain-repl
```

The docs under this directory mirror the source tree under `libs/`. Start with
[`deepagents/`](./deepagents/README.md), then use the roadmap to decide when to
move into CLI, adapters, partners, and examples.

## Packages

| Source package | Documentation | Role |
|---|---|---|
| `libs/deepagents/` | [`deepagents/`](./deepagents/README.md) | Core SDK: `create_deep_agent()`, backend protocol, middleware stack, model/provider profiles. |
| `libs/cli/` | [`cli/`](./cli/README.md) | Textual TUI and LangGraph server lifecycle. |
| `libs/acp/` | [`acp/`](./acp/README.md) | ACP server adapter for a deep agent graph. |
| `libs/evals/` | [`evals/`](./evals/README.md) | Evaluation suite and Harbor benchmark glue. |
| `libs/partners/` | [`partners/`](./partners/README.md) | External sandbox backend implementations. |
| `libs/repl/` | [`repl/`](./repl/README.md) | LangChain REPL interpreter and middleware. |
| `libs/code/` | [`code/`](./code/README.md) | Placeholder package in this tree. |

## Reading order

Read the SDK first. The CLI and adapters make much more sense once you know
that filesystem access goes through `BackendProtocol`, that agent behavior is
mostly middleware, and that `create_deep_agent()` ultimately returns a compiled
LangGraph agent.
