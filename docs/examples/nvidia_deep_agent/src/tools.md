# `nvidia_deep_agent/src/tools.py`

## High-Level Purpose

Provides the web search tool for the NVIDIA Deep Agent's research capabilities. Uses Tavily for URL discovery and fetches full webpage content (not just snippets) converted to markdown. Identical in pattern to `deep_research/research_agent/tools.py`.

## Functions

### `fetch_webpage_content(url: str, timeout: float = 10.0) -> str`

**Purpose:** Fetch a webpage and convert its HTML to markdown.

**Parameters:**
- `url`: URL to fetch.
- `timeout`: HTTP request timeout in seconds (default 10.0).

**Return Value:** Webpage content as markdown string, or an error message string on failure.

**Key Logic:** Uses `httpx.get()` with a browser User-Agent header. Calls `markdownify(response.text)` on successful responses. Returns an error string (does not raise) on any exception.

---

### `tavily_search(query: str, max_results: int = 1, topic: Literal["general", "news", "finance"] = "general") -> str`

**Purpose:** Search the web using Tavily and return full webpage content (not just snippets).

**Decorator:** `@tool(parse_docstring=True)`

**Parameters:**
- `query`: Search query to execute.
- `max_results`: Maximum number of URLs to fetch (default 1). Annotated as `InjectedToolArg` — not visible to the model.
- `topic`: Topic filter. Annotated as `InjectedToolArg` — not visible to the model.

**Return Value:** Formatted string with result count and all results as markdown sections with title, URL, and full page content.

**Key Logic:**
1. Calls `TavilyClient.search()` to discover relevant URLs.
2. For each URL in the results, calls `fetch_webpage_content()` to retrieve full page content.
3. Formats each result as a markdown section with title, URL, and content separated by `---`.
4. Returns a combined string with result count and all sections joined.

## Module-Level Objects

- `tavily_client = TavilyClient()` — Module-level Tavily client instance (uses `TAVILY_API_KEY` env var automatically).

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| `httpx` | `httpx` | HTTP fetching |
| `markdownify` | `markdownify` | HTML-to-markdown conversion |
| `TavilyClient` | `tavily` | Tavily search API client |
| `InjectedToolArg` | `langchain_core.tools` | Parameter injection annotation |
| `tool` | `langchain_core.tools` | Tool decorator |
