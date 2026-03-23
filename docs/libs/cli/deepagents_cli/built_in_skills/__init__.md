# `built_in_skills/__init__.py`

## High-Level Purpose

This is the package initializer for built-in skills that ship with the Deep Agents CLI. Built-in skills are always available at the **lowest precedence level** — user and project skills with the same name will override them.

## Package Description

The `built_in_skills` directory contains skill definitions (Markdown files with YAML frontmatter) that provide default capabilities to the agent. These skills are discovered automatically and loaded before user/project skills, making them the fallback when no higher-precedence skill of the same name exists.

## Skill Directory Structure

```
built_in_skills/
├── __init__.py
└── skill-creator/          # Example built-in skill
    └── scripts/
        ├── init_skill.py
        └── quick_validate.py
```

## Usage

Built-in skills are loaded automatically by `skills/load.py` via the `built_in_skills_dir` parameter of `list_skills()`. They do not need to be imported directly.
