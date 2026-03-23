# `deep_research/pyproject.toml`

## Package Identity

| Field | Value |
|-------|-------|
| Name | `deep-research-example` |
| Version | `0.1.0` |
| Description | Deep research agent example using deepagents package |
| Python requirement | `>=3.11` |
| Build backend | `setuptools` |
| Packages | `["research_agent"]` |

## Dependencies

| Package | Version Constraint | Purpose |
|---------|--------------------|---------|
| `deepagents` | `>=0.2.6` | Core agent framework |
| `langchain-openai` | `>=1.0.2` | OpenAI model integration |
| `langchain-anthropic` | `>=1.0.3` | Claude model integration |
| `langchain-google-genai` | `>=3.1.0` | Gemini model integration |
| `langchain_tavily` | `>=0.2.13` | Tavily LangChain integration |
| `tavily-python` | `>=0.5.0` | Tavily API client |
| `httpx` | `>=0.28.1` | HTTP client for webpage fetching |
| `markdownify` | `>=1.2.0` | HTML-to-markdown conversion |
| `pydantic` | `>=2.0.0` | Data validation |
| `rich` | `>=14.0.0` | Terminal formatting |
| `jupyter` | `>=1.0.0` | Jupyter notebook support |
| `ipykernel` | `>=6.20.0` | Jupyter kernel |
| `python-dotenv` | `>=1.0.0` | Environment variable loading |
| `langgraph-cli[inmem]` | `>=0.1.55` | LangGraph dev server |

## Dependency Overrides (Security)

| Package | Override | Reason |
|---------|----------|--------|
| `nbconvert` | `>=7.17.0` | CVE-2025-53000 |
| `protobuf` | `>=6.33.5` | CVE-2026-0994 |

## Linting Configuration

- Rules: `E` (pycodestyle), `F` (pyflakes), `I` (isort), `D` (pydocstyle), `D401`, `T201`, `UP`
- Ignored: `UP006`, `UP007`, `UP035`, `D417`, `E501`
- Docstring convention: Google style
- `ban-relative-imports = "all"` — All imports must be absolute

## Optional Dependencies

- `dev`: `mypy>=1.11.1`, `ruff>=0.6.1`
