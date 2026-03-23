# `examples/`

## What This Directory Contains

Runnable example agents demonstrating different capabilities and architectures of the deepagents framework. Each example is a self-contained Python package with its own dependencies.

## Examples

| Directory | Description |
|-----------|-------------|
| `content-builder-agent/` | File-driven content creation agent with blog and social media workflows, image generation, and YAML-defined subagents |
| `deep_research/` | Multi-agent deep research system with an orchestrator delegating parallel research tasks to sub-agents |
| `nvidia_deep_agent/` | Multi-model agent with NVIDIA Nemotron Super for research and RAPIDS GPU sandbox for data processing/ML |
| `text-to-sql-agent/` | Natural language to SQL agent using SQLDatabaseToolkit against the Chinook database |

## Architecture Comparison

| Example | Models | Subagents | Backend | Key Tools |
|---------|--------|-----------|---------|-----------|
| `content-builder-agent` | Claude (default) | researcher (Tavily) | FilesystemBackend | generate_cover, generate_social_image |
| `deep_research` | Claude Sonnet 4.5 | research-agent (up to 3 parallel) | (none) | tavily_search, think_tool |
| `nvidia_deep_agent` | Claude Sonnet 4.6 + Nemotron Super | researcher + data-processor | ModalSandbox (A10G GPU) | tavily_search, GPU skills |
| `text-to-sql-agent` | Claude Sonnet 4.5 | (none) | FilesystemBackend | SQLDatabaseToolkit |

## Common Patterns Across Examples

- **`create_deep_agent`**: All examples use the same factory function from the `deepagents` package.
- **Memory files (AGENTS.md)**: Agent identity and general instructions, always loaded into context.
- **Skills**: On-demand workflow documentation loaded from `skills/` directories.
- **Tavily search**: Three of four examples use Tavily for web research with full-page content fetching via httpx + markdownify (not just snippets).
- **FilesystemBackend or ModalSandbox**: Examples use either a local filesystem backend or a Modal cloud sandbox as the execution environment.

## Related Docs

- [`content-builder-agent/README.md`](content-builder-agent/README.md)
- [`deep_research/README.md`](deep_research/README.md)
- [`nvidia_deep_agent/README.md`](nvidia_deep_agent/README.md)
- [`text-to-sql-agent/README.md`](text-to-sql-agent/README.md)
