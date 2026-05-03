# `deepagents/middleware/skills.py`

## High-Level Purpose

`SkillsMiddleware` loads "skills" — reusable prompt templates stored as Markdown files — from backend paths, and injects a catalog of them into the agent's system prompt. The agent can browse and invoke skills, which provide structured guidance for specific task types (e.g., "how to write a blog post", "how to debug a memory leak").

---

## Key Class

### `SkillsMiddleware(AgentMiddleware)`

**Constructor parameters:**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `sources` | `list[str \| tuple[str, str]]` | required | Paths to skills directories, or `(path, label)` tuples |
| `backend` | `BackendProtocol` | required | Backend used to read skill files |

**Sources:** Each source is a path like `/skills/user/` or a `(path, label)` pair. The label appears in the skill catalog. Special case: `"built_in_skills"` is displayed as `"Built-in"`.

---

## Skill File Format

Skill files are Markdown with YAML frontmatter, stored at `{source_path}/{skill-name}/SKILL.md`:

```markdown
---
name: web-research
description: |
  Systematic multi-step web research. Use when asked to research a topic
  comprehensively. Provide the topic; returns a structured report.
license: MIT
---

## Web Research Skill

You will conduct a systematic web research session following these steps:

1. Identify 3-5 key questions about the topic
2. Search for each question using web_search
...
```

**Frontmatter fields:**

| Field | Required | Max length | Description |
|---|---|---|---|
| `name` | Yes | 64 chars | Alphanumeric + hyphens only |
| `description` | Yes | 1024 chars | Used in the skill catalog |
| `license` | No | — | Optional license info |

---

## System Prompt Injection

Skills are injected as an XML block in the system prompt:

```xml
<skills>
  <skill name="web-research" source="User Skills">
    Systematic multi-step web research...
    [full skill content]
  </skill>
  <skill name="code-review" source="Built-in">
    ...
  </skill>
</skills>
```

The agent is instructed to consult skills when relevant and to follow the skill's guidance rather than improvising.

---

## Skill Priority

If two sources define a skill with the same `name`, the **last source wins** (later sources override earlier ones). This lets project-level skills override user-level skills, and user-level skills override built-ins.

---

## CLI Skill Commands

In the CLI, skills are invocable via `/skill:<name> [args]` command. The CLI's `SkillsMiddleware` is configured with:
1. Built-in skills (bundled with CLI package)
2. User skills: `~/.deepagents/{agent}/skills/`
3. Project skills: `./.deepagents/skills/`

---

## See Also

- [README.md](README.md) — middleware stack overview
- [../graph.md](../graph.md) — `skills_paths` parameter
- [../../../cli/deepagents_cli/command_registry.md](../../../cli/deepagents_cli/command_registry.md) — `/skill:` slash command
