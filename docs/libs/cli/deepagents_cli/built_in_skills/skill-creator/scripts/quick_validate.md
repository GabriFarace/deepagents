# `libs/cli/deepagents_cli/built_in_skills/skill-creator/scripts/quick_validate.py`

> Quick validation script for skills - minimal version.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Functions and classes

### `validate_skill(skill_path)`

Basic validation of a skill.

Additional notes from the source docstring:

```text
Returns:
    Tuple of (is_valid, message) where is_valid is bool and message
        describes result.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
