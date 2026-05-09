# `libs/cli/deepagents_cli/command_registry.py`

> Slash command registry and dynamic skill command expansion.

## Position in the system

The input widget, autocomplete, and app command dispatcher all read command
metadata from this module.

## Functions and classes

### `BypassTier`

Enum describing which commands are allowed to bypass normal input submission in
special UI states.

### `SlashCommand`

Metadata object for a slash command: name, help text, usage, handler identity,
argument expectations, and bypass behavior.

### `_build_bypass_set(tier)`

Builds the command-name set for a bypass tier.

### `CommandEntry`

Display/autocomplete entry used when mixing built-in commands and skill
commands.

### `parse_skill_command(command)` and `build_skill_commands(...)`

Parse skill command names and turn loaded skill metadata into command entries.

## Gotchas

Slash commands are local CLI controls unless a handler explicitly submits text
to the agent.
