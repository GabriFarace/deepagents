# `evals/scripts/analyze.py`

> Analyze job trials from a jobs directory.

## Position in the system

This is an executable maintenance/reporting script for the eval package. It is normally called from the command line or from the eval CLI wrappers rather than imported by the SDK.

## Imports and module-level state

This file imports `argparse, asyncio, json, dataclasses, enum, pathlib, typing, deepagents_harbor.failure, deepagents_harbor.stats, deepagents`.
Module constants worth noticing: `ANALYSIS_PROMPT`.

## Functions and classes

### `scan_dataset_for_solutions(dataset_path: Path)`

Scan a dataset directory and create a mapping from task names to solution paths. Args: dataset_path: Path to the dataset directory (e.g., terminal-bench/) Returns: Dictionary mapping task names to their solution/solve.sh paths Example: {"chess-best-move": Path("terminal-bench/7bFm.../chess-best-move/solution/solve.sh")} Key arguments are `dataset_path`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `iterdir`, `exists`, `print`, `is_dir`. It runs synchronously in the caller and returns directly.

### `find_task_directory(trial_dir: Path, task_name: str, task_source: str)`

Find the task directory for a given trial. Args: trial_dir: Path to the trial directory task_name: Name of the task (from config.json) task_source: Source of the task (e.g., "terminal-bench") Returns: Path to the task directory if found, None otherwise Key arguments are `trial_dir`, `task_name`, `task_source`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `iterdir`, `exists`, `is_dir`. It runs synchronously in the caller and returns directly.

### `parse_reward(reward_path: Path)`

Parse the reward file. Returns True if reward is 1, False otherwise. Key arguments are `reward_path`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `read_text`, `strip`. This is asynchronous and awaits I/O or framework operations before returning.

### `extract_task_metadata(trial_dir: Path)`

Extract task metadata from config.json and other files. Args: trial_dir: Path to the trial directory Returns: Dictionary containing task metadata Key arguments are `trial_dir`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `exists`, `open`, `load`, `get`. It runs synchronously in the caller and returns directly.

### `extract_task_instructions(trajectory_path: Path)`

Extract the task instructions from the trajectory file. Looks for the user message in the trajectory steps. Key arguments are `trajectory_path`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `get`, `open`, `load`. It runs synchronously in the caller and returns directly.

### `count_tool_usage(trajectory_path: Path)`

Count tool usage across all steps in a trajectory. Args: trajectory_path: Path to the trajectory.json file in ATIF format Returns: Dictionary mapping tool names to their usage counts Key arguments are `trajectory_path`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `get`, `open`, `load`. It runs synchronously in the caller and returns directly.

### `get_task_name_from_trial(trial_dir: Path)`

Extract the task name from a trial's config.json. Args: trial_dir: Path to the trial directory Returns: Task name if found, None otherwise Key arguments are `trial_dir`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `exists`, `open`, `load`, `get`. It runs synchronously in the caller and returns directly.

### `enrich_trials_with_solutions(trials: list[Trial], solution_mapping: dict[str, Path])`

Update trials with solution paths from a pre-computed solution mapping. Args: trials: List of Trial objects to enrich solution_mapping: Dictionary mapping task names to solution paths Returns: The same list of trials (modified in place) for convenience Key arguments are `trials`, `solution_mapping`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `get_task_name_from_trial`. It runs synchronously in the caller and returns directly.

### `analyze_trial(trial_dir: Path, solution_mapping: Optional[dict[str, Path]]=None)`

Analyze a single trial directory. Returns a Trial object even if trajectory or reward files are missing so incomplete trials can be reported. Status is determined as follows: - FAILED: If exception.txt exists or reward is False - COMPLETED: If reward is True - PENDING: Otherwise (no reward, no exception) Key arguments are `trial_dir`, `solution_mapping`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `exists`, `Trial`, `get_task_name_from_trial`, `count_tool_usage`, `read_text`, `classify_failure`. This is asynchronous and awaits I/O or framework operations before returning.

### `scan_jobs_directory(jobs_dir: Path, solution_mapping: Optional[dict[str, Path]]=None)`

Scan the jobs directory and extract all trial metadata. Args: jobs_dir: Path to the jobs directory containing trial subdirectories solution_mapping: Optional pre-computed mapping from task names to solution paths. If not provided, solutions will be searched for individually. Key arguments are `jobs_dir`, `solution_mapping`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `print`, `exists`, `append`, `iterdir`, `is_dir`, `analyze_trial`. This is asynchronous and awaits I/O or framework operations before returning.

### `print_summary(trials: list[Trial])`

Print a summary of the analyzed trials. Key arguments are `trials`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `print`, `sum`, `sorted`, `len`, `get`, `min_detectable_effect`. It runs synchronously in the caller and returns directly.

### `analyze_failed_trial(trial: Trial, analyze_pending: bool=False)`

Run deep agent analysis on a failed or pending trial trajectory. Args: trial: The trial to analyze analyze_pending: If True, analyze pending trials in addition to failed ones Returns: Analysis result as a string, or None if trajectory cannot be read Key arguments are `trial`, `analyze_pending`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `create_deep_agent`, `dumps`, `upper`, `invoke`, `open`, `load`. This is asynchronous and awaits I/O or framework operations before returning.

### `write_trial_analysis(trial: Trial, trial_dir: Path, output_dir: Path, summary_only: bool=False, analyze_pending: bool=False)`

Analyze a failed or pending trial and write the results to a file. Args: trial: The trial to analyze trial_dir: Path to the trial directory output_dir: Directory where analysis files should be written summary_only: If True, skip LLM analysis and only write metadata summary analyze_pending: If True, analyze pending trials in addition to failed ones Returns: Path to the written analysis file, or None if analysis was skipped Key arguments are `trial`, `trial_dir`, `output_dir`, `summary_only`, `analyze_pending`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `extract_task_metadata`, `mkdir`, `extract_task_instructions`, `open`, `write`, `get`. This is asynchronous and awaits I/O or framework operations before returning.

### `main()`

Main entry point. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `ArgumentParser`, `add_argument`, `parse_args`, `print_summary`, `print`, `scan_dataset_for_solutions`. This is asynchronous and awaits I/O or framework operations before returning.

### `TrialStatus`

Status of a trial execution. This class inherits from `Enum` and is the main object for this part of the module.

### `Trial`

Metadata for a single trial run. This class inherits from `object` and is the main object for this part of the module.

## System prompts and tool descriptions

### `ANALYSIS_PROMPT`

```text
# Trajectory Analysis Prompt

You are analyzing an agent execution trajectory. Your goal is to identify what happened during execution and, if the trial failed, determine why.

## IMPORTANT: Trial Status

The trial status will be explicitly provided to you. This status is the ground truth:
- **FAILED**: The agent did not successfully complete the task (reward = 0 or exception occurred)
- **PENDING**: The trial has not finished executing yet
- **COMPLETED**: The agent successfully completed the task (reward = 1)

**If the status is FAILED, then something went wrong, even if the agent reported success or the trajectory appears successful.** Your job is to identify what went wrong by carefully examining the details.

## Reference Solution

A reference solution script (solve.sh) will be provided when available. This script shows the correct approach to solving the task. Use this to:
- Compare the agent's approach against the known working solution
- Identify where the agent's actions diverged from the correct approach
- Understand what steps or commands the agent missed or executed incorrectly
- Determine if the agent used different tools/methods that led to failure

## Trajectory Format

The trajectory is in ATIF (Agent Trajectory Interchange Format) with sequential steps:
- `source`: Who generated the step (system/user/agent)
- `message`: The content of the step
- `tool_calls`: (if present) Tools the agent attempted to use
- `observation`: (if present) Results from tool execution

## Analysis Task

Review the trajectory with careful attention to subtle details and provide:

### 1. FAILURE IDENTIFICATION (for FAILED trials)

**Start by comparing the user's request to the agent's actual actions:**
- What exactly did the user ask for? (Quote the specific request)
- What exactly did the agent do? (Quote the actual tool calls and parameters)
- If a reference solution is provided, how does the agent's approach differ from it?
- Are there any discrepancies between what was requested and what was executed?

**Then identify:**
- **Failure Step**: Which step number failed or where did things go wrong?
- **What Failed**: Describe what went wrong (tool error, incorrect logic, incomplete execution, subtle mistakes, etc.)
- **Error Details**: Quote any error messages or failure indicators
- **Subtle Issues**: Look for problems that aren't obvious errors - small differences in parameters, values, or execution that don't match the request

**Special Case: Max Iterations Reached**
If the agent failed due to reaching the maximum iteration/recursion limit:
- **Evaluate Progress**: Was the agent making sensible progress toward the solution?
- **Direction Assessment**: Were the agent's actions moving it closer to completing the task?
- **Correctness**: Despite not finishing, were the steps taken correct and logical?
- **Compare to Solution**: If a reference solution is provided, was the agent following a similar approach?
- **Estimate Completion**: How close was the agent to completing the task when it hit the limit?
- **Root Cause**: Was the limit hit due to:
  - Agent making good progress but task simply required more steps?
  - Agent spinning in circles or repeating ineffective actions?
  - Agent pursuing a suboptimal approach that would take too many steps?
  - Agent getting stuck on a subtask or error recovery loop?

### 2. EXECUTION ANALYSIS
- **What the Agent Did**: Trace the agent's actions step by step
- **What Was Expected**: Based on the user's request and reference solution (if provided), what should have happened?
- **Where It Went Wrong**: Identify the specific point where the agent's actions diverged from what was needed
- **Tool Usage**: Examine all tool parameters carefully - verify they match what the user requested

### 3. ROOT CAUSE
Determine the underlying cause:
- Is this incorrect tool usage (wrong tool or wrong parameters)?
- Is this a logical/reasoning error (agent made wrong decision)?
- Is this a tool execution error (tool failed or returned error)?
- Is this incomplete execution (agent stopped too early)?
- Is this a resource/permission error?
- Is this agent confusion about the task requirements?
- Is this a subtle parameter mismatch (values that look correct but differ from the request)?

### 4. SUGGESTED IMPROVEMENTS
If clear from the trajectory, suggest:
- What the agent should have done differently (reference the solution script if available)
- Which component or capability needs improvement
- How to prevent this type of failure

## Guidelines

- **Pay close attention to details**: Even if the agent reported success, if the trial failed, find what went wrong
- **Use the reference solution**: When provided, compare the agent's approach systematically against it
- Look for subtle issues like path mistakes, incorrect values, or logical errors
- Be concise but specific
- Quote exact error messages when present
- Focus on actionable insights
- Identify patterns in agent behavior that led to failure
- Don't assume the agent is correct just because it reported success
```

## Gotchas

Several blocks are async and assume they run inside the owning framework event loop. Calling them from sync code needs the package CLI or an explicit async runner.
