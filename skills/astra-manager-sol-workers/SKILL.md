---
name: astra-manager-sol-workers
description: Use GPT-6 Astra High as the manager and code reviewer while GPT-5.6 Sol High subagents inspect, implement, test, and repair code. Use for coding changes when the user says "create multiple agents for the work" or clearly asks for manager-worker orchestration, Astra-managed delegation, Sol workers, multiple agents, or reduced Astra usage through subagents. Do not use for ordinary questions or review-only requests that need no implementation.
---

# Astra manager with Sol workers

Keep the main task in a manager and reviewer role. The main task may inspect the repository, run read-only commands, run verification commands, write the plan, and review evidence. It must not edit implementation files, write migrations, or repair the code itself.

Only the main task may spawn agents. Every worker brief must prohibit the worker from delegating or spawning another agent. When the user says "create multiple agents for the work" or a clear equivalent, use as many useful workers as current runtime concurrency allows while keeping the main task as manager and reviewer. Split work into independent, non-overlapping ownership; do not spawn workers merely to fill available slots. Queue overlapping or dependent work until its prerequisite is complete or assign it to the worker that owns that area.

No actor may commit, push, deploy, mutate a database, or change an external system unless the user explicitly authorized that action. Verification commands must respect the same boundary.

The session or app configuration selects the main model. This skill cannot change the model of an already-running task. When model metadata is available, verify that the main task uses `gpt-6-astra` with `high` reasoning. If it does not, tell the user once that a new task or manual model switch is needed. If metadata is unavailable, continue without claiming that the main model was verified.

## Workflow

1. Read the user's request and repository instructions. Inspect only enough current code and worktree state to define scope, risks, and acceptance checks.
2. Write a short implementation plan. Separate independent work only when separate workers will save time or reduce risk, and give each worker non-overlapping ownership.
3. Delegate all code changes to workers with `model: "gpt-5.6-sol"`, `reasoning_effort: "high"`, and `fork_turns: "none"`. Give each worker a self-contained brief because it receives no parent history. When multiple agents were requested, start as many independently useful workers as current runtime concurrency permits and queue dependent or overlapping assignments.
4. Wait for the workers. Review the actual working-tree diff and relevant source, not only the workers' summaries. Check the request, repository rules, correctness, regressions, security-sensitive behavior, and test evidence.
5. If the review finds a fixable problem, send precise findings back to the worker that owns that area with `followup_task`. The manager must not make the repair. Reuse the responsible worker instead of spawning a replacement.
6. Re-review the repaired diff. Complete only when the implementation and proportionate checks pass, or report a concrete blocker.

## Worker brief

Include only what the worker needs:

- the requested outcome and acceptance criteria
- exact repository root and relevant paths or symbols when known
- required repository instructions and existing dirty-worktree boundaries
- files or areas the worker owns and must not touch
- proportionate tests or verification commands
- authorization boundaries for commits, pushes, deployments, databases, and external systems

Tell the worker to inspect, implement, and verify the change without delegating or spawning another agent. Require a concise return containing changed files, checks run with results, remaining risks, and any blocker. The worker must preserve unrelated user changes and must not commit, push, deploy, mutate a database, or change an external system unless the user explicitly authorized that action.

## Usage discipline

- Use one Sol worker by default unless the user requests multiple agents. For "create multiple agents for the work" or a clear equivalent, use as many independently useful Sol workers as current runtime concurrency allows, without displacing the main manager task.
- Give workers independent, non-overlapping ownership. Do not spawn workers just to fill slots; queue overlapping or dependent work.
- Do not spawn an agent for each file or each review finding.
- Keep worker prompts self-contained and compact. Avoid full-history forks.
- Reuse the original worker for repair passes because it already knows the implementation.
- Use one repair pass by default. Use a second only when the remaining finding is material and narrowly fixable. After two unsuccessful repair passes, stop and report the blocker.
- Do not repeat broad exploration or tests when current evidence already answers the review question. Verify risky or ambiguous claims directly.
- Do not claim a guaranteed percentage reduction in daily or weekly limits. Sol subagent work still consumes Codex usage; this pattern reduces how much implementation work Astra performs.

## Completion report

Report the implemented outcome, files changed, validation results, the manager's review decision, and any unresolved risk. State the worker model and effort that were requested when spawning it. Do not claim the main model was changed or verified unless the runtime exposed that fact.
