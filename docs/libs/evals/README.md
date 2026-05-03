# `libs/evals/` — deepagents-evals Package

`deepagents-evals` is the evaluation suite for the deepagents framework. It provides tooling for running agents against benchmark tasks, measuring performance metrics, and comparing models or configurations.

---

## Structure

```
libs/evals/
├── deepagents_evals/        ← Core evaluation library
│   └── radar.py             ← Main evaluation runner
├── deepagents_harbor/       ← Harbor integration (remote eval sandbox)
├── EVAL_CATALOG.md          ← List of available evaluation tasks
├── MODEL_GROUPS.md          ← Model groupings for comparative evals
└── CONTRIBUTING.md          ← How to add new eval tasks
```

---

## Key Concepts

**Eval task:** A benchmark task with a defined setup, agent prompt, and expected outcome (graded by the evaluator).

**Harbor:** A sandboxed execution environment for running evals safely. The `deepagents_harbor` module handles communication with Harbor.

**Radar:** The main evaluation runner. Loads tasks from the catalog, runs each against the agent, grades outcomes, and produces a report.

---

## Running Evals

```bash
cd libs/evals
uv run deepagents-evals run --model claude-sonnet-4-6
```

Or from the root:
```bash
make evals MODEL=claude-sonnet-4-6
```

---

## See Also

- [EVAL_CATALOG.md](../../libs/evals/EVAL_CATALOG.md) — available eval tasks
- [MODEL_GROUPS.md](../../libs/evals/MODEL_GROUPS.md) — model groupings
