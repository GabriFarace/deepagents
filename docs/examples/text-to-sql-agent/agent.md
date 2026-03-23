# `text-to-sql-agent/agent.py`

## High-Level Purpose

The main entry point for the Text-to-SQL Deep Agent. Connects to the Chinook SQLite database, creates a `SQLDatabaseToolkit` with SQL inspection and query tools, and wraps everything in a `create_deep_agent` with file-based memory, skills, and a `FilesystemBackend`. Exposes a CLI for running single natural language queries.

## Functions

### `create_sql_deep_agent() -> CompiledStateGraph`

**Purpose:** Assemble and return the configured SQL Deep Agent.

**Return Value:** A LangGraph `CompiledStateGraph` ready for `invoke()` or `stream()`.

**Key Logic:**
1. Resolves `base_dir` to the script's directory for absolute path portability.
2. Connects to `chinook.db` using `SQLDatabase.from_uri()` (SQLite) with `sample_rows_in_table_info=3` so the model sees example rows when exploring schema.
3. Initializes `ChatAnthropic(model="claude-sonnet-4-5-20250929", temperature=0)` — temperature 0 for deterministic SQL generation.
4. Creates `SQLDatabaseToolkit(db=db, llm=model)` and retrieves its tools via `toolkit.get_tools()`. This yields: `list_tables`, `get_schema`, `query_checker`, `execute_query`.
5. Calls `create_deep_agent` with:
   - `model`: Claude Sonnet 4.5
   - `memory=["./AGENTS.md"]` — Agent identity and general SQL principles, always loaded
   - `skills=["./skills/"]` — Specialized workflows for query writing and schema exploration, loaded on-demand
   - `tools=sql_tools` — SQL toolkit tools
   - `subagents=[]` — No subagents
   - `backend=FilesystemBackend(root_dir=base_dir)` — Persistent file storage

---

### `main() -> None`

**Purpose:** CLI entry point. Parses a natural language question from `sys.argv`, invokes the agent, and displays the result using Rich panels.

**Key Logic:**
1. Uses `argparse` to accept a single positional `question` argument.
2. Displays the question in a cyan-bordered Rich `Panel`.
3. Calls `create_sql_deep_agent()` to build the agent.
4. Calls `agent.invoke({"messages": [{"role": "user", "content": args.question}]})`.
5. Extracts `result["messages"][-1].content` as the final answer.
6. Displays the answer in a green-bordered Rich `Panel`, or an error in red on exception (exits with code 1).

**CLI Usage:**
```bash
python agent.py "What are the top 5 best-selling artists?"
python agent.py "Which employee generated the most revenue by country?"
```

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| `create_deep_agent` | `deepagents` | Agent factory |
| `FilesystemBackend` | `deepagents.backends` | Local file access backend |
| `ChatAnthropic` | `langchain_anthropic` | Claude LLM client |
| `SQLDatabaseToolkit` | `langchain_community.agent_toolkits` | SQL inspection + query tools |
| `SQLDatabase` | `langchain_community.utilities` | SQLAlchemy-backed database abstraction |
| `rich` | `rich` | Console output with panels |
| `load_dotenv` | `python-dotenv` | Load `.env` file for API keys |
