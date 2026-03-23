# `pyproject.toml` — Package Configuration

## High-Level Purpose

This file defines the `deepagents-cli` Python package configuration using the `hatchling` build backend. It specifies all dependencies, optional extras for model providers and sandbox integrations, the console entry point, and development tooling configuration.

## Build System

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

## Package Metadata

| Field | Value |
|---|---|
| Name | `deepagents-cli` |
| Current version | `0.0.34` |
| Python requirement | `>=3.11, <4.0` |
| License | MIT |
| Status | Beta (Development Status 4) |

## Core Dependencies

### Framework

| Package | Version Range | Purpose |
|---|---|---|
| `deepagents` | `==0.4.11` | Core agent SDK |
| `langchain` | `>=1.2.13,<2.0.0` | LangChain framework |
| `langgraph` | `>=1.1.2,<2.0.0` | Agent graph execution |
| `langgraph-checkpoint-sqlite` | `>=3.0.0,<4.0.0` | SQLite checkpointing |

### Client-Server Architecture

| Package | Purpose |
|---|---|
| `langgraph-sdk` | Remote LangGraph server client |
| `langgraph-cli[inmem]` | `langgraph dev` server CLI |
| `httpx` | HTTP client for server communication |

### Model Providers (Core)

| Package | Provider |
|---|---|
| `langchain-anthropic` | Anthropic (Claude models) |
| `langchain-google-genai` | Google (Gemini models) |
| `langchain-openai` | OpenAI (GPT models) |

### UI/Terminal

| Package | Purpose |
|---|---|
| `textual` | TUI framework |
| `textual-autocomplete` | Autocomplete widget support |
| `textual-speedups` | Performance optimizations |
| `prompt-toolkit` | Terminal input handling |
| `rich` | Rich text and markup |
| `markdownify` | HTML-to-Markdown conversion |

### Tools & Utilities

| Package | Purpose |
|---|---|
| `tavily-python` | Web search tool |
| `pyperclip` | Clipboard integration |
| `uuid-utils` | UUID7 generation |
| `python-dotenv` | `.env` file loading |
| `requests` | HTTP client |
| `pillow` | Image processing |
| `pyyaml` | YAML parsing (subagent frontmatter) |
| `aiosqlite` | Async SQLite access |
| `tomli-w` | TOML file writing |
| `langchain-mcp-adapters` | MCP tool loading |
| `deepagents-acp` | ACP server protocol |
| `langsmith[sandbox]` | LangSmith integration + sandbox |

## Optional Extras

### Model Providers

Optional extras install additional LangChain provider integrations:

| Extra | Provider |
|---|---|
| `anthropic` | Anthropic |
| `baseten` | Baseten |
| `bedrock` | AWS Bedrock |
| `cohere` | Cohere |
| `deepseek` | DeepSeek |
| `fireworks` | Fireworks AI |
| `google-genai` | Google Generative AI |
| `groq` | Groq |
| `huggingface` | Hugging Face |
| `ibm` | IBM WatsonX |
| `litellm` | LiteLLM |
| `mistralai` | Mistral AI |
| `nvidia` | NVIDIA NIM |
| `ollama` | Ollama (local models) |
| `openai` | OpenAI |
| `openrouter` | OpenRouter |
| `perplexity` | Perplexity AI |
| `vertexai` | Google Vertex AI |
| `xai` | xAI (Grok) |

### Sandbox Providers

| Extra | Sandbox |
|---|---|
| `agentcore` | AWS AgentCore Code Interpreter |
| `daytona` | Daytona |
| `modal` | Modal |
| `runloop` | Runloop |

## Console Script Entry Point

```toml
[project.scripts]
deepagents = "deepagents_cli.main:cli_main"
```

This makes `deepagents` available as a CLI command after installation.

## Development Tool Configuration

### Ruff (Linter + Formatter)

- Target Python version: 3.11+
- Line length: 100
- Enabled rules: E, F, I, N, W, UP, B, A, S, T, PTH, RUF, ARG, PL, SIM, PERF, TID
- Key ignores: `E501` (line length — handled by formatter), `T201` (print statements used for CLI output), `S603`/`S404` (subprocess use is intentional)

### Pytest

```toml
[tool.pytest.ini_options]
asyncio_mode = "auto"
filterwarnings = ["error"]
```

### Coverage

```toml
[tool.coverage.run]
source = ["deepagents_cli"]
omit = ["tests/*"]
```
