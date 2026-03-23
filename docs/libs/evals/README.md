# `libs/evals` — deepagents-evals

## What This Package Does

`deepagents-evals` is the evaluation and benchmarking library for the Deep Agents framework. It contains two sub-packages:

- **`deepagents_harbor`**: The Harbor integration layer — running Deep Agents in Harbor sandbox environments, collecting trajectories, integrating with LangSmith, and classifying failures.
- **`deepagents_evals`**: Shared eval utilities — currently the radar chart generator and category definitions used to visualize multi-model benchmark comparisons.

It also contains a large test suite (`tests/evals/`) with both capability evals (run against live agent instances) and a `tests/unit_tests/` folder for pure unit tests.

## Directory Layout

```
libs/evals/
├── deepagents_harbor/
│   ├── __init__.py            # Re-exports key public symbols
│   ├── backend.py             # HarborSandbox — sandbox backend for eval execution
│   ├── deepagents_wrapper.py  # DeepAgentsWrapper — Harbor BaseAgent implementation
│   ├── failure.py             # FailureCategory enum and classify_failure()
│   ├── langsmith.py           # LangSmith dataset/experiment/feedback integration
│   ├── metadata.py            # InfraMetadata — sandbox infrastructure capture
│   └── stats.py               # Wilson CI and MDE statistical utilities
├── deepagents_evals/
│   ├── __init__.py            # Empty (package marker)
│   ├── categories.json        # Canonical eval category names and display labels
│   └── radar.py               # Radar chart generation from ModelResult objects
├── tests/
│   ├── evals/                 # Capability eval tests (require live agent)
│   │   ├── conftest.py        # Fixtures and agent setup
│   │   ├── external_benchmarks.py  # BFCL / FRAMES / Nexus benchmark runners
│   │   ├── llm_judge.py       # LLM-as-judge evaluator
│   │   ├── pytest_reporter.py # Custom pytest reporter for eval output
│   │   ├── utils.py           # Shared test helpers
│   │   ├── test_file_operations.py
│   │   ├── test_followup_quality.py
│   │   ├── test_hitl.py
│   │   ├── test_memory.py
│   │   ├── test_memory_multiturn.py
│   │   ├── test_skills.py
│   │   ├── test_subagents.py
│   │   ├── test_summarization.py
│   │   ├── test_system_prompt.py
│   │   ├── test_tool_selection.py
│   │   ├── test_tool_usage_relational.py
│   │   ├── test_external_benchmarks.py
│   │   ├── memory_agent_bench/   # MemoryAgentBench evaluation
│   │   └── tau2_airline/         # TAU2 Airline domain evaluation
│   └── unit_tests/
│       ├── test_category_tagging.py
│       ├── test_external_benchmark_helpers.py
│       ├── test_imports.py
│       ├── test_infra.py
│       ├── test_langsmith.py
│       └── test_radar.py
├── scripts/
│   ├── analyze.py             # Analyze eval results from JSON
│   ├── generate_radar.py      # CLI for generating radar charts
│   └── harbor_langsmith.py    # CLI for Harbor–LangSmith integration
├── pyproject.toml
└── Makefile
```

## How Files Relate

### Harbor integration flow
1. **`deepagents_wrapper.py`** is the entry point Harbor calls. It instantiates a `HarborSandbox` from **`backend.py`**, creates a Deep Agent, runs it, and saves a trajectory.
2. **`metadata.py`** is called at the start of each trial to capture sandbox specs.
3. **`failure.py`** is used post-trial to classify whether a failure was infrastructure noise or model capability.
4. **`langsmith.py`** connects Harbor trial results to LangSmith: creating datasets from Harbor task registries, creating experiment sessions, and uploading reward feedback scores.
5. **`stats.py`** provides Wilson CI and MDE utilities for result reporting.

### Radar chart flow
1. **`deepagents_evals/categories.json`** defines the canonical categories and labels.
2. **`deepagents_evals/radar.py`** reads that file at import time and exposes `generate_radar()`, `generate_individual_radars()`, and `load_results_from_summary()`.
3. **`scripts/generate_radar.py`** provides a CLI wrapper around the radar module.

### Eval tests
The `tests/evals/` capability tests run against actual deployed agent instances and are parameterized by model. The `tests/unit_tests/` folder tests pure Python logic without agent invocations.

## Key Concepts

### Harbor
Harbor is an external evaluation harness that spins up sandbox environments (Docker, Modal, etc.), runs agents, and collects trajectories. `DeepAgentsWrapper` is the bridge between Harbor and Deep Agents.

### Trajectory (ATIF v1.2)
The ATIF (Agent Trajectory Interchange Format) is the JSON format Harbor uses to store agent execution traces. `DeepAgentsWrapper._save_trajectory()` converts LangChain message history into this format.

### LangSmith Experiment Linking
When `LANGSMITH_EXPERIMENT` is set, agent runs are traced to a named LangSmith experiment. After the job, `add_feedback()` attaches Harbor reward scores to the corresponding LangSmith traces.

### Eval Categories
The canonical list (defined in `categories.json`) includes: `file_operations`, `skills`, `hitl`, `memory`, `summarization`, `subagents`, `system_prompt`, `tool_usage`, `followup_quality`, `external_benchmarks`, `tau2_airline`, `memory_agent_bench`.
