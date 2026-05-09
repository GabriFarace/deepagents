# `evals/scripts/generate_radar.py`

> Generate radar charts from eval results.

## Position in the system

This is an executable maintenance/reporting script for the eval package. It is normally called from the command line or from the eval CLI wrappers rather than imported by the SDK.

## Imports and module-level state

This file imports `__future__, argparse, json, os, sys, pathlib, deepagents_evals.radar`.

## Functions and classes

### `main()`

Entry point for radar chart generation. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `ArgumentParser`, `add_mutually_exclusive_group`, `add_argument`, `parse_args`, `set`, `extend`. It runs synchronously in the caller and returns directly.

## Gotchas

Most behavior is delegated through framework protocols, so preserve method signatures when editing even if an argument appears unused locally.
