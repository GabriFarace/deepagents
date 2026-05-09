# `partners/quickjs/langchain_quickjs/_skills.py`

> Skill module loader for the REPL.

## Position in the system

This partner package implements or supports a BackendProtocol-compatible sandbox. The core SDK can use these classes through its backend abstraction without depending directly on the provider.

## Imports and module-level state

This file imports `__future__, logging, re, dataclasses, pathlib, typing, quickjs_rs`.
Module constants worth noticing: `SKILL_MODULE_EXTENSIONS`, `_MAX_BUNDLE_BYTES`, `_SKILL_NAME_RE`, `_SKILL_SPECIFIER_RE`.

## Functions and classes

### `load_skill(metadata: SkillMetadata, backend: BackendProtocol)`

Load one skill into a ``LoadedSkill``. Raises: InvalidSkillScopeError: Metadata has no `module` key, or the entrypoint doesn't match any file in the skill directory. SkillInstallError: Backend fetch failed, content was non-UTF-8, or the bundle exceeded the size cap. Key arguments are `metadata`, `backend`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `download_files`, `LoadedSkill`, `InvalidSkillScopeError`, `append`, `SkillInstallError`, `ModuleScope`. It runs synchronously in the caller and returns directly.

### `aload_skill(metadata: SkillMetadata, backend: BackendProtocol)`

Async sibling of :func:`load_skill`. Key arguments are `metadata`, `backend`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `LoadedSkill`, `InvalidSkillScopeError`, `adownload_files`, `append`, `SkillInstallError`, `ModuleScope`. This is asynchronous and awaits I/O or framework operations before returning.

### `scan_skill_references(source: str)`

Return the set of skill names the source imports from. Extracts every literal ``"@/skills/<name>"`` specifier the source contains. The caller is responsible for rejecting unknown names with a ``SkillNotAvailable``-style error — this is a scan, not a validator. A returned name is not proof the skill exists or installs cleanly. Dynamic imports with computed specifiers are not detected; those can only succeed if a literal reference elsewhere in the session has already triggered install. Key arguments are `source`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `frozenset`, `findall`. It runs synchronously in the caller and returns directly.

### `SkillLoadError`

Base class for skill-load failures surfaced at install time. This class inherits from `Exception` and is the main object for this part of the module.

### `InvalidSkillScopeError`

Skill directory contains nothing installable. Either no module-extension files were found, or the frontmatter ``module`` path doesn't match any of them. This class inherits from `SkillLoadError` and is the main object for this part of the module.

### `SkillInstallError`

Backend fetch failed or produced unreadable content for a skill. This class inherits from `SkillLoadError` and is the main object for this part of the module.

### `LoadedSkill`

A skill's install-ready state. Attributes: name: Spec-validated skill name (kebab-case). specifier: The bare specifier we install under. Always ``"@/skills/<name>"``. scope: A ``ModuleScope`` carrying every code file from the skill directory, with the entrypoint renamed to ``index.<ext>`` if the author picked a different name. This class inherits from `object` and is the main object for this part of the module.

## Gotchas

Several blocks are async and assume they run inside the owning framework event loop. Calling them from sync code needs the package CLI or an explicit async runner.
