# `libs/deepagents/deepagents/profiles/harness/harness_profiles.py`

> Runtime and declarative profile types for shaping deep-agent behavior.

## Position in the system

`create_deep_agent()` consumes harness profiles after the chat model has been
resolved. A profile can replace or suffix system prompts, override tool
descriptions, hide tools, remove non-required middleware, append extra
middleware, and alter the auto-added general-purpose subagent.

The same lookup pattern as provider profiles is used: exact `provider:model`
registrations layer on provider-wide registrations, and user/plugin
registrations layer on built-ins through additive merge semantics.

## Imports and module-level state

`_HARNESS_PROFILES` is the internal registry. `_HARNESS_PROFILE_CONFIG_KEYS`
and `_GENERAL_PURPOSE_SUBAGENT_KEYS` are derived from dataclass fields so
`from_dict()` stays synchronized with the dataclasses. The module lazily imports
`deepagents.graph` only when it needs the required-middleware names, avoiding a
top-level import cycle.

## Functions and classes

### `_scaffolding_violation_label(entry: object) -> str | None`

Checks whether an `excluded_middleware` entry names required scaffolding. String
entries are compared with required middleware names; class entries are compared
by class `__name__`. It returns a label for error reporting or `None` when the
entry is allowed to proceed.

### `_format_scaffolding_rejection(violations: list[str]) -> str`

Formats the shared error message used when a profile tries to exclude required
scaffolding. This keeps construction-time validation in this module aligned
with assembly-time validation in `_excluded_middleware.py`.

### `GeneralPurposeSubagentProfile`

Small immutable dataclass that edits the auto-added `general-purpose` subagent.
`enabled` is a three-state override (`None` inherit/default, `True` force,
`False` disable). `description` and `system_prompt` replace the default
description or prompt when provided.

#### `GeneralPurposeSubagentProfile.to_dict(self) -> dict[str, Any]`

Serializes only non-`None` fields, keeping config output minimal and preserving
inheritance/default semantics.

#### `GeneralPurposeSubagentProfile.from_dict(cls, data: Mapping[str, Any]) -> GeneralPurposeSubagentProfile`

Validates a plain mapping and constructs the sub-profile. Unknown keys and
wrong value types raise `TypeError`; valid omitted keys remain `None`.

### `HarnessProfileConfig`

Declarative, YAML/JSON-friendly subset of harness settings. It supports plain
prompt strings, tool-description mappings, excluded tool names, string-form
middleware exclusions, and nested `GeneralPurposeSubagentProfile` data. It does
not support runtime-only middleware instances or factories.

#### `HarnessProfileConfig.__post_init__(self) -> None`

Freezes `tool_description_overrides` with `MappingProxyType`, validates
string-form `excluded_middleware`, and rejects any attempt to exclude required
scaffolding.

#### `HarnessProfileConfig.to_dict(self) -> dict[str, Any]`

Exports only non-default fields to plain scalars, lists, and dicts suitable for
JSON/YAML. It sorts set-like fields for stable output and emits an explicit
empty `general_purpose_subagent` object when that distinction matters.

#### `HarnessProfileConfig.from_dict(cls, data: Mapping[str, Any]) -> HarnessProfileConfig`

Constructs config from a dict with strict key and type checking. Unknown keys
raise `TypeError`; invalid middleware names or required-scaffolding exclusions
raise `ValueError`.

#### `HarnessProfileConfig.to_harness_profile(self) -> HarnessProfile`

Converts declarative config to runtime `HarnessProfile`. This direction is
lossless because config contains only fields that runtime profiles can express.

#### `HarnessProfileConfig.from_harness_profile(cls, profile: HarnessProfile) -> HarnessProfileConfig`

Exports a runtime profile back to declarative config when possible. It rejects
runtime-only `extra_middleware` and class-form excluded middleware entries that
lack a public `serialized_name`, rather than silently dropping behavior.

### `HarnessProfile`

Runtime dataclass consumed by `create_deep_agent()`. It can replace/suffix
system prompts, override tool descriptions, exclude tools, exclude middleware
by class or name, add extra middleware, and adjust the default general-purpose
subagent.

#### `HarnessProfile.__post_init__(self) -> None`

Freezes mutable container fields, converts static `extra_middleware` sequences
to tuples, validates string-form middleware exclusions, and rejects
required-scaffolding exclusions. Callable middleware factories are preserved so
they can be invoked on each materialization.

#### `HarnessProfile.materialize_extra_middleware(self) -> list[AgentMiddleware]`

Returns a fresh list of extra middleware. If `extra_middleware` is a factory,
the factory is invoked for this call; otherwise the stored tuple is copied.

### `_apply_profile_prompt(profile: HarnessProfile, base_prompt: str) -> str`

Applies prompt replacement and suffix rules. `base_system_prompt` replaces the
provided base when set; `system_prompt_suffix` appends after a blank line when
set. `create_deep_agent()` uses the same helper for the main agent,
declarative subagents, and the general-purpose subagent.

### `_coerce_str_or_none(value: object, field_name: str) -> str | None`

Validates optional string fields loaded from dict-backed config.

### `_coerce_str_mapping(value: object, field_name: str) -> dict[str, str]`

Validates a string-to-string mapping and returns a plain dict. `None` becomes
an empty dict.

### `_coerce_frozen_strset(value: object, field_name: str) -> frozenset[str]`

Validates list/tuple/set/frozenset values containing only strings. `None`
becomes an empty frozenset.

### `_coerce_general_purpose_subagent(value: object) -> GeneralPurposeSubagentProfile | None`

Accepts `None` or a mapping and converts mappings through
`GeneralPurposeSubagentProfile.from_dict()`.

### `_validate_config_middleware_string(entry: object, field_name: str) -> None`

Validates string grammar for config-form middleware exclusions. Entries must be
non-empty strings, cannot contain `:`, and cannot start with `_`. Required
scaffolding and "matched something" checks happen elsewhere because they need
graph-level policy or assembled stacks.

### `_serialize_runtime_excluded_middleware_entry(entry: type[AgentMiddleware] | str) -> str`

Serializes runtime middleware exclusions to config form. Strings pass through;
class entries require a non-empty `serialized_name` attribute. Arbitrary
`module:Class` serialization is intentionally rejected.

### `_ensure_harness_profiles_loaded() -> None`

Calls the shared lazy built-in/plugin bootstrap before registry access.

### `_coerce_runtime_harness_profile(profile: HarnessProfile | HarnessProfileConfig) -> HarnessProfile`

Converts config objects to runtime profiles and passes runtime profiles
through unchanged.

### `_register_harness_profile_impl(key: str, profile: HarnessProfile | HarnessProfileConfig) -> None`

Internal registration primitive. It validates the key, coerces config to
runtime form, and merges on top of any existing profile under the same key.

### `register_harness_profile(key: str, profile: HarnessProfile | HarnessProfileConfig) -> None`

Public beta registration API. It ensures bootstrap has completed, then layers
the incoming runtime or config profile onto the registry entry for `key`.
Scalar fields prefer the incoming profile, mapping fields merge with incoming
values winning, excluded sets union, extra middleware merge by concrete type,
and general-purpose subagent settings merge field by field.

### `_has_any_harness_profile() -> bool`

Returns whether any user registration exists beyond bootstrap-provided harness
profiles. `graph.py` uses this to choose logging behavior for profile misses.

### `_get_harness_profile(spec: str) -> HarnessProfile | None`

Looks up the runtime harness profile for a spec, checking exact key first and
provider prefix second. If both match, the exact model profile is merged on top
of the provider profile. Malformed specs return `None`.

### `_resolve_middleware_seq(middleware: Sequence[AgentMiddleware] | Callable[[], Sequence[AgentMiddleware]]) -> Sequence[AgentMiddleware]`

Normalizes a static middleware sequence or zero-arg factory. Factories are
called at materialization time, allowing fresh middleware instances per stack.

### `_merge_middleware(base: Sequence[AgentMiddleware], override: Sequence[AgentMiddleware]) -> tuple[AgentMiddleware, ...]`

Merges two middleware sequences by concrete class. Base entries keep their
position unless replaced by an override of the same exact type; new override
types append at the end. This lets provider-level middleware be replaced by a
model-level/user registration without losing ordering for unrelated entries.

### `_merge_general_purpose_subagent_profiles(base: GeneralPurposeSubagentProfile | None, override: GeneralPurposeSubagentProfile | None) -> GeneralPurposeSubagentProfile | None`

Field-wise merge for the nested subagent profile. Unset override fields inherit
base fields, while explicit override values win.

### `_merge_profiles(base: HarnessProfile, override: HarnessProfile) -> HarnessProfile`

Constructs the merged runtime profile used for repeated registrations and
provider-plus-exact lookup. Scalars prefer override when set, mapping fields
merge with override values winning, exclusion sets union, middleware sequences
merge by exact type, and nested general-purpose subagent settings merge
field-wise.

### `_harness_profile_for_model(model: BaseChatModel, spec: str | None) -> HarnessProfile`

Resolves the harness profile for an already-built chat model. If the original
string `spec` is available, it is the authoritative lookup key. Otherwise the
helper extracts provider and model identifier from the model instance and tries
the canonical `provider:identifier` key, an identifier-only lookup only when
the identifier already contains `:`, and finally a provider-only lookup.

When no profile matches, it returns an empty `HarnessProfile` as a null object.
The miss is logged at warning level only when user-registered harness profiles
exist beyond bootstrap defaults, because that situation often means a profile
key did not match the pre-built model's derived identity.

## Gotchas

`excluded_middleware` is not a way to remove required scaffolding. The code
rejects both class-form and string-form attempts to exclude middleware such as
filesystem or subagent scaffolding so the assembled graph remains functional.
