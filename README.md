# AGY Terminal Subagent

A reusable AI skill for delegating independent, self-contained terminal tasks to the AGY CLI. It helps AI agents offload bounded terminal work that can be completed from a single prompt without requiring conversation context, follow-up interaction, or shared state.

## Features

- Delegates independent tasks to AGY terminal workers
- Encourages self-contained prompts
- Includes model selection guidance
- Requires execution logs for transparency
- Documents delegation best practices

## Installation

Using **npx**:

```bash
npx skills install https://github.com/devsanthoshmk/agy-subagent.git
```

Using **pnpx**:

```bash
pnpx skills install https://github.com/devsanthoshmk/agy-subagent.git
```

## Usage

Once installed, the skill can be invoked automatically by compatible AI agents when a task is suitable for delegation.

See [`agy-terminal-subagent/SKILL.md`](agy-terminal-subagent/SKILL.md) for the complete documentation.
