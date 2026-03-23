# `deep_research/research_agent/tools.py`

## High-Level Purpose

Provides the two tools used by the research sub-agent: a web search tool (Tavily + full page content fetching) and a strategic reflection tool.

## Functions

### `fetch_webpage_content(url: str, timeout: float = 10.0) -> str`

**Purpose:** Fetch a webpage and convert its HTML to markdown.

**Parameters:**
- `url`: URL to fetch.
- `timeout`: HTTP request timeout in seconds (default 10.0).

**Return Value:** Webpage content as markdown (via `markdownify`), or an error message string on failure.

**Key Logic:** Uses `httpx.get()` with a browser User-Agent header. Calls `markdownify(response.text)` to convert HTML to markdown.

---

### `tavily_search(query: str, max_results: int = 1, topic: Literal["general", "news", "finance"] = "general") -> str`

**Purpose:** Search the web using Tavily and return full webpage content (not just snippets).

**Decorator:** `@tool(parse_docstring=True)`

**Parameters:**
- `query`: Search query to execute.
- `max_results`: Maximum number of URLs to fetch (default 1). Annotated as `InjectedToolArg` — not visible to the model.
- `topic`: Topic filter. Annotated as `InjectedToolArg` — not visible to the model.

**Return Value:** Formatted string with search results including full markdown content of each fetched page.

**Key Logic:**
1. Calls `TavilyClient.search()` to discover relevant URLs.
2. For each URL in the results, calls `fetch_webpage_content()` to retrieve full page content.
3. Formats each result as a markdown section with title, URL, and content.
4. Returns a combined string with result count and all sections.

---

### `think_tool(reflection: str) -> str`

**Purpose:** A strategic reflection tool that records the agent's analysis of its research progress.

**Decorator:** `@tool(parse_docstring=True)`

**Parameters:**
- `reflection`: Detailed reflection text covering: current findings, missing information, quality assessment, and next steps decision.

**Return Value:** `f"Reflection recorded: {reflection}"` — the tool confirms the reflection was recorded.

**Key Logic:** This is essentially a no-op tool whose value comes from forcing the model to verbalize its reasoning before deciding whether to continue searching. Acts as a "chain-of-thought" step.

## Module-Level Objects

- `tavily_client = TavilyClient()` — Module-level client instance (uses `TAVILY_API_KEY` env var automatically).

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| `httpx` | `httpx` | HTTP fetching |
| `markdownify` | `markdownify` | HTML-to-markdown conversion |
| `TavilyClient` | `tavily` | Tavily search API client |
| `InjectedToolArg` | `langchain_core.tools` | Parameter injection annotation |
