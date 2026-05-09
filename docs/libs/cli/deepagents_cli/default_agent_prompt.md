# `libs/cli/deepagents_cli/default_agent_prompt.md`

> Markdown prompt or skill payload shipped with the CLI.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Prompt or skill text

The file is executable documentation for the agent runtime, so the full text is quoted here.

```markdown
# Project Notes

This file is for tracking project-specific context as you work.
You can update this file to remember decisions, patterns, and context about this project.

## Architecture Notes

(Add notes about project structure, key files, patterns as you discover them)

## Decisions

(Track important decisions and their rationale)
```

## Gotchas

Treat this file as behavioral code: changing wording can alter model behavior, skill activation, or deployment defaults even when no Python tests fail.
