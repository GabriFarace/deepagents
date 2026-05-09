# `libs/cli/deepagents_cli/built_in_skills/`

> Built-in skills shipped with the CLI.

## Position in the system

Built-in skills are the lowest-precedence skill source. The CLI lists them by
default, but user and project skill directories can override a built-in skill by
reusing its name.

## Files

- [`remember/SKILL.md`](./remember/SKILL.md) is a skill prompt for capturing
  durable preferences, best practices, and reusable workflows into memory or
  new skills.
- [`skill-creator/SKILL.md`](./skill-creator/SKILL.md) is the full skill-design
  guide used when creating, updating, or validating skills.
- [`skill-creator/scripts/init_skill.md`](./skill-creator/scripts/init_skill.md)
  documents the scaffold helper for new skill folders.
- [`skill-creator/scripts/quick_validate.md`](./skill-creator/scripts/quick_validate.md)
  documents the lightweight validator for skill structure.

## Gotchas

`SKILL.md` files are prompt payloads, not passive docs. Their frontmatter affects
skill discovery and their body affects model behavior after activation.
