# `evals/deepagents_evals/categories.json`

> Category registry consumed by eval listing and reporting code.

## Position in the system

`deepagents_evals.cli` loads this JSON file to define known evaluation categories. Scripts that generate catalogs and radar charts rely on the same category names for stable grouping.

## Contents

The file currently defines 3 categories:

- `categories`
- `radar_categories`
- `labels`

## Gotchas

Changing a category name can break historical summaries, catalog generation, and chart labels unless older outputs are migrated or aliased.
