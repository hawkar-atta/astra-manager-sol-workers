# Astra Manager, Sol Workers

A Codex skill that keeps GPT-6 Astra High in the manager and code-review role while GPT-5.6 Sol High subagents inspect, implement, test, and repair code.

## What it does

- The Astra manager plans the work and reviews the actual diff.
- All implementation is delegated to Sol High workers.
- Saying "create multiple agents for the work" starts as many independently useful workers as the current Codex runtime allows.
- Each worker receives a separate area to prevent overlapping edits.
- Review findings go back to the worker that owns the affected area.
- Workers cannot create more agents.

The skill does not change the model of an already-running Codex task. Select GPT-6 Astra with High reasoning for the main task before invoking it. The skill requests `gpt-5.6-sol` with High reasoning for every worker.

## Install

Ask Codex:

```text
Install the skill from https://github.com/hawkar-atta/astra-manager-sol-workers/tree/main/skills/astra-manager-sol-workers
```

Or copy `skills/astra-manager-sol-workers` from this repository into your Codex skills directory:

- Windows: `%USERPROFILE%\.codex\skills\astra-manager-sol-workers`
- macOS or Linux: `~/.codex/skills/astra-manager-sol-workers`

Start a new Codex task after installation so the skill is discovered.

## Use

For one worker:

```text
$astra-manager-sol-workers Implement the requested change. You remain the manager and reviewer.
```

For parallel workers:

```text
$astra-manager-sol-workers Create multiple agents for the work: implement the requested feature.
```

The manager uses only independently useful worker slots. Dependent or overlapping work is queued or assigned to the same owner.

## Usage note

This pattern reduces the amount of implementation handled by Astra. It does not guarantee a particular reduction in Codex limits because every Sol worker still consumes usage. Running more workers in parallel can finish independent work sooner while consuming usage faster.

## Repository layout

```text
skills/
  astra-manager-sol-workers/
    SKILL.md
    agents/
      openai.yaml
```
