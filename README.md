# AGY terminal subagent

## Purpose

The AGY terminal subagent skill provides a delegation protocol for AI agents. Primary agents offload bounded, independent tasks to an Antigravity CLI subagent. Workers execute as headless subagents in isolated terminal processes without access to prior conversation context or shared agent memory. This CLI agent delegation pattern isolates noisy tool executions, enables parallel autonomous terminal agent runs, and prevents context window bloat during AI terminal automation workflows.

## Architecture

The workflow isolates subagent execution through a single-pass command line boundary.

```
+-------------------------------------------------------+
|                     Primary agent                     |
+-------------------------------------------------------+
                           |
                           | 1. Evaluates task eligibility
                           | 2. Bundles complete context
                           | 3. Launches agy command
                           v
+-------------------------------------------------------+
|             Antigravity CLI worker process            |
|  * Runs as a headless single-pass process             |
|  * Uses an independent context window                 |
|  * Executes local terminal and file tools             |
+-------------------------------------------------------+
                           |
                           | 1. Writes temporary execution log
                           | 2. Emits structured result or NEEDS_INPUT
                           v
+-------------------------------------------------------+
|                     Primary agent                     |
|  * Audits execution log                               |
|  * Verifies returned artifacts                        |
+-------------------------------------------------------+
```

Worker lifecycle follows three rules:
1. Complete self-containment. Each worker prompt contains the entire task, explicit file paths, and required output schemas. Workers cannot ask interactive questions.
2. Isolated execution logs. Every worker writes actions, errors, and created artifacts to a temporary log file. The worker returns this file path upon completion.
3. Explicit failure signaling. If essential input is missing, the worker returns `NEEDS_INPUT: <missing items>`. The primary agent gathers the data and spawns a new worker.

## Setup

Install the skill into your project or agent environment using the skills package runner.

Install with npx:
```bash
npx skills install https://github.com/devsanthoshmk/agy-subagent.git
```

Install with pnpx:
```bash
pnpx skills install https://github.com/devsanthoshmk/agy-subagent.git
```

Prerequisites:
- Antigravity CLI binary installed on the system PATH
- Active API credentials configured for Antigravity

## Usage

The primary agent spawns a worker by running the `agy` binary in headless print mode with `-p` and specifying a target model.

Command syntax:
```bash
agy -p "<complete self-contained prompt>" --model "<model-identifier>" --dangerously-skip-permissions
```

Example command:
```bash
agy -p "Inspect docs/api.md and extract all deprecated endpoints into a markdown table. Write significant actions to /tmp/opencode/worker-audit.log and return the path." --model "gemini-3.8-flash-high" --dangerously-skip-permissions
```

### Parallel execution

When a workflow contains multiple independent tasks, launch separate concurrent workers instead of combining tasks into one prompt. For example, run two workers simultaneously to analyze separate modules:

```bash
agy -p "Audit auth/jwt.py for token expiration handling. Return markdown findings and the audit log path." --model "gemini-3.8-flash-high" --dangerously-skip-permissions &
agy -p "Audit storage/s3.py for bucket permission checks. Return markdown findings and the audit log path." --model "gemini-3.8-flash-high" --dangerously-skip-permissions &
wait
```

### Verification

The primary agent validates worker results against logs and artifacts. For high-stakes operations, verify results using one of these methods:
- Run a second pass with a different model tier
- Run deterministic checks such as unit tests or regex queries
- Inspect the recorded execution log file directly

## Configuration

### Model selection

Select the smallest model that meets task complexity.

| Model identifier | Target workload |
|---|---|
| `gemini-3.8-flash-low` | Trivial extraction, regex filtering, and single-file lookups |
| `gemini-3.8-flash-medium` | General code search, diff inspection, and log filtering |
| `gemini-3.8-flash-high` | Structured summarization, multi-file code comparisons, and script analysis |
| `claude-sonnet-4-6` | Moderate multi-step refactoring and code audits |
| `claude-opus-4-6-thinking` | Complex debugging, architectural decisions, and high-accuracy evaluations |

### CLI flags

The skill integrates with standard Antigravity CLI flags:
- `-p`: Runs the worker in non-interactive print mode
- `--model`: Sets the model tier
- `--print-timeout`: Limits execution duration
- `--output-format`: Sets output to text, json, or stream-json
- `--dangerously-skip-permissions`: Skips interactive permission prompts for automated workflows

### Skill specification

Review `agy-terminal-subagent/SKILL.md` for prompt templates, boundary conditions, and delegation rules.
