# `examples/content-builder-agent`

## What This Example Does

Demonstrates a file-driven content creation agent that writes blog posts and social media content. The agent's personality, brand voice, and available workflows are loaded entirely from files on disk — no code changes are needed to customize its behavior.

## Files

| File | Description |
|------|-------------|
| `content_writer.py` | Main script — agent setup, tools, and streaming CLI |
| `AGENTS.md` | Brand voice and style guide, loaded as agent memory |
| `subagents.yaml` | YAML definition of the researcher subagent |
| `skills/blog-post/SKILL.md` | Skill workflow for writing blog posts |
| `skills/social-media/SKILL.md` | Skill workflow for social media content |
| `pyproject.toml` | Package metadata and dependencies |

## Architecture

```
User Prompt
    │
    ▼
Content Writer Agent (create_deep_agent)
    ├── MemoryMiddleware → loads AGENTS.md (brand voice)
    ├── SkillsMiddleware → loads skills/ (blog-post, social-media workflows)
    ├── generate_cover tool (Gemini image generation)
    ├── generate_social_image tool (Gemini image generation)
    └── Subagents:
        └── researcher (uses web_search via Tavily)
```

## How to Run

```bash
uv run python content_writer.py "Write a blog post about AI agents"
uv run python content_writer.py "Create a LinkedIn post about prompt engineering"
```

## Environment Variables Required

| Variable | Purpose |
|----------|---------|
| `ANTHROPIC_API_KEY` | Model API key |
| `TAVILY_API_KEY` | Web search API key (for researcher subagent) |
| `GOOGLE_API_KEY` | Google Gemini API key (for image generation tools) |
