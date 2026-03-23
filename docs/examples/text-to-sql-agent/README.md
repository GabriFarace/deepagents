# `examples/text-to-sql-agent/`

## What This Example Does

Demonstrates a Text-to-SQL Deep Agent that translates natural language questions into SQL queries against the Chinook SQLite database. Uses `SQLDatabaseToolkit` for SQL tools, file-based memory (AGENTS.md) for agent identity, and on-demand skills for query writing and schema exploration.

## Files

| File | Description |
|------|-------------|
| `agent.py` | Main agent implementation and CLI entry point |
| `AGENTS.md` | Agent identity, core principles, and safety rules (always loaded) |
| `skills/query-writing/SKILL.md` | Workflow for writing and executing SQL queries |
| `skills/schema-exploration/SKILL.md` | Workflow for database structure discovery |
| `chinook.db` | Sample SQLite database (downloaded separately, gitignored) |
| `pyproject.toml` | Package metadata and dependencies |

## Architecture

```
User Question (CLI)
    │
    ▼
SQL Deep Agent (create_deep_agent)
    ├── memory=["./AGENTS.md"]       (agent identity, always loaded)
    ├── skills=["./skills/"]         (query-writing, schema-exploration — on-demand)
    ├── backend=FilesystemBackend    (local file storage)
    └── tools (from SQLDatabaseToolkit):
        ├── list_tables
        ├── get_schema
        ├── query_checker
        └── execute_query
    │
    ▼
Chinook SQLite Database
    │
    ▼
Formatted Answer (Rich panels)
```

## How to Run

```bash
# Install dependencies
uv sync

# Download the Chinook database
curl -L -o chinook.db https://github.com/lerocha/chinook-database/raw/master/ChinookDatabase/DataSources/Chinook_Sqlite.sqlite

# Set API key
export ANTHROPIC_API_KEY=your_key

# Run
python agent.py "What are the top 5 best-selling artists?"
```

## Environment Variables Required

| Variable | Purpose |
|----------|---------|
| `ANTHROPIC_API_KEY` | Claude API key (required) |
| `LANGCHAIN_API_KEY` | LangSmith tracing (optional) |
| `LANGCHAIN_TRACING_V2` | Enable LangSmith tracing (optional) |

## Key Design Patterns

- **Progressive disclosure**: AGENTS.md is always loaded; skills are loaded on-demand when the agent determines which skill matches the task — keeps context efficient while providing deep expertise.
- **SQLDatabaseToolkit**: Provides `list_tables`, `get_schema`, `query_checker`, and `execute_query` tools so the agent can safely explore schema before generating queries.
- **No subagents**: Simpler than the deep_research or nvidia_deep_agent examples; a single agent handles the full workflow.
- **FilesystemBackend**: Enables the agent to save intermediate work (query plans, scratch notes) to the local filesystem.

## Related Docs

- [`agent.md`](agent.md) — `create_sql_deep_agent()` and `main()` function details
