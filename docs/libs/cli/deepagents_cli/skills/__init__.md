# `libs/cli/deepagents_cli/skills/__init__.py`

> Skills module for deepagents CLI.

## Position in the system

This file supports the CLI skill system: discovering `SKILL.md` files, presenting them in slash/CLI commands, and wrapping a selected skill body into the first user message so the agent follows those instructions.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from deepagents_cli.skills.commands import execute_skills_command, setup_skills_parser`


## Functions and classes

This module has no public functions or classes. It exists for package discovery, typing markers, constants, side-effect imports, or re-export behavior described above.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
