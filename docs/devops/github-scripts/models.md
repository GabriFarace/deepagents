# Script: `models.py`

## Overview

Single source of truth for all AI model definitions used in the `evals.yml` and `harbor.yml` workflows. Resolves a model selection string (preset name or comma-separated specs) to a GitHub Actions matrix JSON. Each model is declared once in a registry with tags encoding which workflow presets it belongs to.

## Location

`.github/scripts/models.py`

## Usage in CI/CD

```bash
# For evals workflow (reads EVAL_MODELS env var)
python .github/scripts/models.py eval

# For harbor workflow (reads HARBOR_MODELS env var)
python .github/scripts/models.py harbor
```

Output is written to `$GITHUB_OUTPUT` as `matrix={"model":["provider:model", ...]}` (or printed to stdout if `GITHUB_OUTPUT` is not set).

## Data Structures

### `Model` (NamedTuple)

| Field | Type | Description |
|---|---|---|
| `spec` | `str` | Model identifier in `provider:model` format |
| `groups` | `frozenset[str]` | Set of tag strings like `eval:set0`, `harbor:anthropic` |

### `REGISTRY`

A tuple of all registered models. Tags follow the convention `{workflow}:{group}`.

#### Eval presets and their tag membership

| Preset | Tag | Included Providers |
|---|---|---|
| `set0` | `eval:set0` | Anthropic, OpenAI, Google, OpenRouter, Baseten, Fireworks |
| `set1` | `eval:set1` | Select Anthropic, OpenAI, Google, Baseten, Fireworks, Ollama |
| `set2` | `eval:set2` | Ollama, Groq, xAI, NVIDIA |
| `open` | `eval:open` | OpenRouter, Baseten (GLM-5), NVIDIA |
| `all` | `eval:*` | All models with any `eval:` tag |

#### Harbor presets and their tag membership

| Preset | Tag | Included Providers |
|---|---|---|
| `anthropic` | `harbor:anthropic` | Anthropic Claude models |
| `openai` | `harbor:openai` | OpenAI models |
| `baseten` | `harbor:baseten` | Baseten-hosted models |
| `all` | `harbor:*` | All models with any `harbor:` tag |

## Functions

### `_filter_by_tag(prefix: str, tag: str | None) -> list[str]`

Returns all model specs from `REGISTRY` that:
- Match the exact `tag` (if provided), or
- Have any group starting with `prefix` (for the `all`/`None` case)

Preserves REGISTRY order.

### `_resolve_models(workflow: str, selection: str) -> list[str]`

Resolves a selection string:
1. If `selection` matches a known preset name → `_filter_by_tag`
2. Otherwise, parses as comma-separated `provider:model` specs
3. Validates that each spec contains `:` and only safe characters (alphanumeric, `:`, `-`, `_`, `.`, `/`)
4. Raises `ValueError` for invalid input

### `main() -> None`

Entry point:
1. Reads workflow name (`eval` or `harbor`) from `sys.argv[1]`
2. Reads model selection from the appropriate env var (`EVAL_MODELS` or `HARBOR_MODELS`), defaulting to `"all"`
3. Resolves models via `_resolve_models`
4. Outputs `matrix={"model": [...]}` to `$GITHUB_OUTPUT` or stdout

## Security

Model specs are validated against `_SAFE_SPEC_RE = r"^[a-zA-Z0-9:_\-./]+"` to reject shell metacharacters (`$`, `` ` ``, `;`, `|`, `&`, `(`, `)`, etc.) before the specs are passed to GitHub Actions matrix contexts.

## Exit Codes

| Condition | Exit |
|---|---|
| Success | `0` |
| Wrong usage / unknown workflow | Exits with usage message |
| Invalid/empty model selection | Exits with `ValueError` message |

## Dependencies

- Standard library only: `json`, `os`, `re`, `sys`, `typing`
