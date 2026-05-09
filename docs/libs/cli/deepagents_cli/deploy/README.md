# `libs/cli/deepagents_cli/deploy/`

> Deployment support for turning a local deepagents project into a LangGraph
> deployable bundle.

## Position in the system

The deploy command path reads `deepagents.toml`, `AGENTS.md`, optional `skills/`,
optional `mcp.json`, optional `user/` memory, optional subagents, and optional
frontend configuration. It validates that local project, writes a build
directory, and then hands the bundle to `langgraph deploy`.

## Files

- [`config.md`](./config.md) parses and validates `deepagents.toml`.
- [`bundler.md`](./bundler.md) writes `_seed.json`, generated graph/auth files,
  LangGraph config, pyproject dependencies, and optional frontend assets.
- [`commands.md`](./commands.md) wires `deepagents deploy init/dev/deploy` parser
  actions to config loading, bundling, LangSmith Hub seeding, and deploy shell
  commands.
- [`context_hub.md`](./context_hub.md) provides the Hub-backed memory backend
  vendored into deployed bundles.
- [`templates.md`](./templates.md) stores generated Python/TOML template strings.

## Static Frontend

`frontend_dist/` is a pre-built frontend bundle. It is intentionally summarized
rather than documented file-by-file because the source of truth is the frontend
project that produced the minified assets.
