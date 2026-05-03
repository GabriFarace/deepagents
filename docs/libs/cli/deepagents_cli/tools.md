# `libs/cli/deepagents_cli/tools.py`

## High-Level Purpose

`tools.py` defines the two built-in non-filesystem tools available to the CLI agent: `fetch_url` (web page retrieval) and `web_search` (Tavily web search). Both are LangChain `BaseTool` subclasses added to the agent's tool list in `agent.py`.

---

## Tools

### `fetch_url`

Fetches a URL and converts the HTML response to clean Markdown.

**Input schema:**
- `url: str` — the URL to fetch

**Behavior:**
1. Makes an HTTP GET request via `httpx`
2. Parses the HTML with `html2text` or `markdownify`
3. Returns the Markdown-formatted page content (truncated if too long)
4. Returns an error message if the request fails

**Always available.** No API key required.

---

### `web_search`

Performs a web search via the Tavily API and returns a summary of results.

**Input schema:**
- `query: str` — the search query
- `max_results: int` (optional, default 5)

**Behavior:**
1. Calls the Tavily search API
2. Returns a structured list of results: title, URL, and snippet for each

**Only added to the agent if `TAVILY_API_KEY` is set** in the environment. If the key is missing, `web_search` is omitted from the tool list and the agent falls back to `fetch_url` for any web access.

---

## Architecture Notes

Both tools are intentionally simple wrappers — they don't have retry logic, caching, or rate limiting. For production deployments that need more robust web access, replacing these with custom tool implementations (or MCP server equivalents) is straightforward.

---

## See Also

- [agent.md](agent.md) — where these tools are added to the agent
- [mcp_tools.md](mcp_tools.md) — additional tools loaded from MCP servers
