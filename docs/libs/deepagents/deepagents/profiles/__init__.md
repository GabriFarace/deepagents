# `libs/deepagents/deepagents/profiles/__init__.py`

> Public beta import surface for profile APIs.

## Position in the system

This module is a thin barrel file. It re-exports provider and harness profile
classes and registration helpers so callers can import them from
`deepagents.profiles`, while the package root re-exports the same public names
from `deepagents`.

## Imports and module-level state

No registry bootstrap happens here. The comment in the source is important:
built-in provider and harness profiles are loaded lazily on first registry
access, not at import time.

## Functions and classes

This file defines no functions or classes. It is covered primarily by the
parent [`README.md`](./README.md), plus the concrete docs for
[`provider/provider_profiles.md`](./provider/provider_profiles.md) and
[`harness/harness_profiles.md`](./harness/harness_profiles.md).

