# `built_in_skills/` — Built-in Skills

This directory contains skills that are shipped with the Deep Agents CLI package. These skills are always available to the agent at the lowest precedence level — user and project skills with the same name will always take priority.

## Overview

Built-in skills provide out-of-the-box capabilities that users can immediately use without any configuration. They are discovered automatically by `skills/load.py`.

## Skills

### `skill-creator`

A built-in skill that helps users create new reusable agent skills. It provides a structured workflow for defining a new skill, writing its instructions, and validating its format.

The skill-creator includes helper scripts in `scripts/`:
- `init_skill.py` — initializes a new skill directory with template
- `quick_validate.py` — validates a skill file's frontmatter and structure

## Directory Structure

```
built_in_skills/
├── __init__.py
└── skill-creator/
    ├── skill.md              # Skill definition (YAML frontmatter + instructions)
    └── scripts/
        ├── init_skill.py     # Skill initialization helper
        └── quick_validate.py # Skill validation helper
```

## Precedence

Built-in skills are at priority 0 (lowest). They are overridden by:
1. User skills (`~/.deepagents/{agent}/skills/`)
2. Project skills (`.deepagents/skills/`)
3. Claude-compatibility skills (experimental)

See `skills/load.py` for the full precedence chain.
