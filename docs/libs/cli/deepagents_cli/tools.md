# `libs/cli/deepagents_cli/tools.py`

> Custom tools for the CLI agent.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Functions and classes

### `_get_tavily_client()`

Get or initialize the lazy Tavily client singleton.

Additional notes from the source docstring:

```text
Returns:
    TavilyClient instance, or None if API key is not configured.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `web_search(query: str, max_results: int=5, topic: Literal['general', 'news', 'finance']='general', include_raw_content: bool=False)`

Search the web using Tavily for current information and documentation.

Additional notes from the source docstring:

```text
This tool searches the web and returns relevant results. After receiving results,
you MUST synthesize the information into a natural, helpful response for the user.

Args:
    query: The search query (be specific and detailed)
    max_results: Number of results to return (default: 5)
    topic: Search topic type - "general" for most queries, "news" for current events
    include_raw_content: Include full page content (warning: uses more tokens)

Returns:
    Dictionary containing:
    - results: List of search results, each with:
        - title: Page title
        - url: Page URL
        - content: Relevant excerpt from the page
        - score: Relevance score (0-1)
    - query: The original search query

IMPORTANT: After using this tool:
1. Read through the 'content' field of each result
2. Extract relevant information that answers the user's question
3. Synthesize this into a clear, natural language response
4. Cite sources by mentioning the page titles or URLs
5. NEVER show the raw JSON to the user - always provide a formatted response
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `fetch_url(url: str, timeout: int=30)`

Fetch content from a URL and convert HTML to markdown format.

Additional notes from the source docstring:

```text
This tool fetches web page content and converts it to clean markdown text,
making it easy to read and process HTML content. After receiving the markdown,
you MUST synthesize the information into a natural, helpful response for the user.

Args:
    url: The URL to fetch (must be a valid HTTP/HTTPS URL)
    timeout: Request timeout in seconds (default: 30)

Returns:
    Dictionary containing:
    - success: Whether the request succeeded
    - url: The final URL after redirects
    - markdown_content: The page content converted to markdown
    - status_code: HTTP status code
    - content_length: Length of the markdown content in characters

IMPORTANT: After using this tool:
1. Read through the markdown content
2. Extract relevant information that answers the user's question
3. Synthesize this into a clear, natural language response
4. NEVER show the raw markdown to the user unless specifically requested
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
