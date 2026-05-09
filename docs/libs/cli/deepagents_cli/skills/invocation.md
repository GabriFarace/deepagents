# `libs/cli/deepagents_cli/skills/invocation.py`

> Helpers for loading and formatting skill invocations.

## Position in the system

This file supports the CLI skill system: discovering `SKILL.md` files, presenting them in slash/CLI commands, and wrapping a selected skill body into the first user message so the agent follows those instructions.

## Functions and classes

### `SkillInvocationEnvelope`

Structured prompt and checkpoint metadata for a skill invocation.

Additional notes from the source docstring:

```text
Attributes:
    prompt: Composed prompt that wraps `SKILL.md` content with
        invocation instructions.
    message_kwargs: Extra fields merged into the initial HumanMessage.
```

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `discover_skills_and_roots(assistant_id: str)`

Discover skills and build pre-resolved containment roots.

Additional notes from the source docstring:

```text
Args:
    assistant_id: Agent identifier used to resolve user skill directories.

Returns:
    Tuple of `(skill metadata list, pre-resolved containment roots)`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `build_skill_invocation_envelope(skill: ExtendedSkillMetadata, content: str, args: str='')`

Build the wrapped prompt and persisted metadata for a skill.

Additional notes from the source docstring:

```text
Args:
    skill: Loaded skill metadata.
    content: Raw `SKILL.md` content.
    args: Optional user request appended after the skill body.

Returns:
    A `SkillInvocationEnvelope` with the composed prompt and
        `message_kwargs` containing persisted skill metadata.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
