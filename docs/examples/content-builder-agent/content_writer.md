# `content-builder-agent/content_writer.py`

## High-Level Purpose

A runnable content creation agent that writes blog posts and social media content. The agent's behavior (brand voice, style) is configured through files on disk (`AGENTS.md`, `skills/`), and specialized research is delegated to a subagent. It streams output using Rich terminal formatting.

## Tools

### `web_search(query: str, max_results: int = 5, topic: Literal["general", "news"] = "general") -> dict`

**Purpose:** Search the web using the Tavily API.

**Parameters:**
- `query`: The search query string.
- `max_results`: Number of results (default 5).
- `topic`: `"general"` or `"news"`.

**Return Value:** Tavily search results dict, or `{"error": "..."}` on failure.

**Dependencies:** `TavilyClient` from `tavily`, `TAVILY_API_KEY` env var.

---

### `generate_cover(prompt: str, slug: str) -> str`

**Purpose:** Generate a cover image for a blog post using Google Gemini.

**Parameters:**
- `prompt`: Image description.
- `slug`: Blog post slug; image is saved to `blogs/<slug>/hero.png`.

**Return Value:** Path string or error message.

---

### `generate_social_image(prompt: str, platform: str, slug: str) -> str`

**Purpose:** Generate an image for a social media post using Google Gemini.

**Parameters:**
- `prompt`: Image description.
- `platform`: `"linkedin"` or `"tweets"`.
- `slug`: Post slug; image is saved to `<platform>/<slug>/image.png`.

**Return Value:** Path string or error message.

## Functions

### `load_subagents(config_path: Path) -> list`

**Purpose:** Load subagent definitions from a YAML config file and resolve tool name references to actual tool objects.

**Parameters:**
- `config_path`: Path to `subagents.yaml`.

**Return Value:** List of subagent dicts with keys `name`, `description`, `system_prompt`, and optionally `model` and `tools`.

**Key Logic:** Maintains a `available_tools` mapping (`{"web_search": web_search}`) and resolves tool names from the YAML `tools` lists to actual callables.

---

### `create_content_writer() -> CompiledStateGraph`

**Purpose:** Assemble and return the content writer agent.

**Key Configuration:**
- `memory=["./AGENTS.md"]` — Brand voice / style guide loaded by `MemoryMiddleware`
- `skills=["./skills/"]` — Specialized workflows loaded by `SkillsMiddleware`
- `tools=[generate_cover, generate_social_image]` — Image generation tools
- `subagents=load_subagents(...)` — Research subagent from YAML
- `backend=FilesystemBackend(root_dir=EXAMPLE_DIR)` — Local filesystem access

---

## Classes

### `AgentDisplay`

**Purpose:** Rich-based terminal display manager that tracks how many messages have been printed and shows a spinner during waiting periods.

**Attributes:**
- `printed_count: int` — How many messages have already been rendered.
- `current_status: str` — Current spinner status text.
- `spinner: Spinner` — Rich spinner object.

**Methods:**

##### `update_status(status: str)`
Updates the spinner text.

##### `print_message(msg)`
Renders messages with panel formatting:
- `HumanMessage` → blue panel
- `AIMessage` with text content → green panel with Markdown rendering
- `AIMessage` with tool calls → inline status lines (researching, generating image, writing file, searching)
- `ToolMessage` → checkmark or error for known tool names

## Entry Point

### `main() -> None`

**Purpose:** CLI entry point. Reads the task from `sys.argv[1:]` (defaults to a blog post about AI agents). Runs the agent with streaming and uses Rich's `Live` context for spinner display.

**Key Pattern:** Streams with `stream_mode="values"`, tracking `messages` in the state and printing only newly arrived messages.

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| `create_deep_agent` | `deepagents` | Agent factory |
| `FilesystemBackend` | `deepagents.backends` | Local file access |
| `yaml` | `pyyaml` | YAML subagent config loading |
| `rich` | `rich` | Terminal display (console, live, markdown, panels, spinners) |
| `TavilyClient` | `tavily` | Web search (lazy import) |
| `google.genai` | `google-genai` | Image generation (lazy import) |
