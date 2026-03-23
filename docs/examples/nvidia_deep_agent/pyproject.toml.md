# `nvidia_deep_agent/pyproject.toml`

## Package Identity

| Field | Value |
|-------|-------|
| Name | `nemotron-deep-agent` |
| Version | `0.1.0` |
| Description | General-purpose deep agent: frontier orchestrator + Nemotron Super subagents + NVIDIA GPU skills |
| Python requirement | `>=3.11` |
| Build backend | `setuptools` |
| Packages | `["src"]` |

## Dependencies

| Package | Version Constraint | Purpose |
|---------|--------------------|---------|
| `deepagents` | `>=0.3.0` | Core agent framework |
| `langchain` | `>=0.3.0` | LLM orchestration |
| `langchain-anthropic` | `>=0.3.0` | Claude model integration |
| `langgraph` | `>=0.4.0` | Graph-based agent execution |
| `tavily-python` | `>=0.5.0` | Web search API client |
| `httpx` | `>=0.28.0` | HTTP client for webpage fetching |
| `markdownify` | `>=1.2.0` | HTML-to-markdown conversion |
| `python-dotenv` | `>=1.0.0` | Environment variable loading |
| `langgraph-cli[inmem]` | `>=0.1.55` | LangGraph dev server |
| `modal` | `>=0.73.0` | Serverless GPU sandbox |
| `langchain-modal` | `>=0.0.2` | LangChain wrapper for Modal |
| `langchain-nvidia-ai-endpoints` | `>=1.1.0` | NVIDIA NIM / ChatNVIDIA model client |

## Linting Configuration

- `ban-relative-imports = "all"` — All imports must be absolute (Ruff linting rule).
