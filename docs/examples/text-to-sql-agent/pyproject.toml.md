# `text-to-sql-agent/pyproject.toml`

## Package Identity

| Field | Value |
|-------|-------|
| Name | `text2sql-deepagent` |
| Version | `0.1.0` |
| Description | A natural language to SQL query agent powered by LangChain's Deep Agents framework and Claude Sonnet 4.5 |
| Python requirement | `>=3.11` |
| Build backend | `setuptools` |
| Repository | https://github.com/langchain-ai/deepagents/tree/main/examples/text-to-sql-agent |

## Dependencies

| Package | Version Constraint | Purpose |
|---------|--------------------|---------|
| `deepagents` | `>=0.3.5` | Core agent framework |
| `langchain` | `>=1.2.3` | LLM orchestration |
| `langchain-anthropic` | `>=1.3.1` | Claude model integration |
| `langchain-community` | `>=0.3.0` | `SQLDatabaseToolkit` and `SQLDatabase` |
| `langgraph` | `>=1.0.6` | Graph-based agent execution |
| `sqlalchemy` | `>=2.0.0` | SQL database abstraction layer |
| `python-dotenv` | `>=1.0.0` | Environment variable loading |
| `tavily-python` | `>=0.5.0` | Web search API (listed but not used in main agent) |
| `rich` | `>=13.0.0` | Terminal formatting for CLI output |

## Linting Configuration

- `ban-relative-imports = "all"` — All imports must be absolute (Ruff linting rule).
