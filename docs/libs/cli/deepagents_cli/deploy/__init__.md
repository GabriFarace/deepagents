# `libs/cli/deepagents_cli/deploy/__init__.py`

> Deploy commands for bundling and shipping deep agents.

## Position in the system

This file belongs to the `deepagents deploy` path. The deploy package reads a project layout, validates `deepagents.toml`, bundles prompts, memories, skills, MCP config, optional frontend assets, and emits files that `langgraph deploy` can run.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from deepagents_cli.deploy.commands import execute_deploy_command, execute_dev_command, execute_init_command, setup_deploy_parsers`

- `from deepagents_cli.deploy.config import SandboxProvider, SandboxScope`


## Functions and classes

This module has no public functions or classes. It exists for package discovery, typing markers, constants, side-effect imports, or re-export behavior described above.

## Gotchas

Deploy helpers usually write files, read environment variables, or shell out through command helpers. Keep validation errors explicit because deployment failures otherwise surface late inside LangGraph Cloud tooling.
