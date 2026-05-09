# `libs/cli/deepagents_cli/skills/`

> CLI skill discovery, metadata display, command handling, and invocation prompt
> wrapping.

## Position in the system

The SDK has middleware for skills, but the CLI also needs filesystem-oriented
commands: list, create, inspect, delete, and invoke. These files bridge user and
project skill directories to the command surface and to the first human message
sent into the agent.

## Files

- [`load.md`](./load.md) discovers skills from built-in, user, project,
  `.agents`, and experimental `.claude` locations with override precedence.
- [`invocation.md`](./invocation.md) resolves allowed roots and builds the
  wrapped prompt that includes the selected `SKILL.md`.
- [`commands.md`](./commands.md) implements the `deepagents skills` subcommands
  and validates names, paths, deletion confirmation, template generation, and
  display formatting.

## Gotchas

Skill paths can be symlinks, so containment checks use resolved paths and an
allow-list. Keep that behavior intact when changing command output or
invocation flow.
