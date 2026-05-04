---
name: hermes-coding-workflow
description: Use when doing non-trivial coding with Hermes Agent. Orchestrates Karpathy-style minimal-change discipline, grill-plan intake, TDD, systematic debugging, subagent-driven implementation, spikes, and pre-commit review into one end-to-end workflow.
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [coding, workflow, planning, tdd, debugging, subagents, review]
    related_skills: [karpathy-guidelines, grill-plan, writing-plans, test-driven-development, systematic-debugging, subagent-driven-development, requesting-code-review, spike]
---

# Hermes Coding Workflow

## Overview

This skill is an end-to-end coding workflow for Hermes Agent. It combines several focused skills into one operating system:

- `karpathy-guidelines` for judgment, minimalism, and surgical changes.
- `grill-plan` for clarifying ambiguous or high-risk work before implementation.
- `writing-plans` for turning clear requirements into executable plans.
- `test-driven-development` for behavior changes and bug fixes.
- `systematic-debugging` for root-cause debugging instead of random patches.
- `subagent-driven-development` for parallel, reviewed implementation.
- `requesting-code-review` for the pre-commit or pre-ship gate.
- `spike` for disposable feasibility experiments before real builds.

The goal is not ceremony. The goal is fewer wrong assumptions, smaller diffs, less overengineering, and more work that is actually verified before it is called done.

## When to Use

Use this skill when:

- Writing, changing, reviewing, or refactoring production code.
- Fixing bugs or test failures.
- Planning a feature, migration, or architecture change.
- Breaking down a multi-step implementation.
- Delegating work to subagents.
- Preparing to commit, push, ship, or open a PR.
- You need one workflow that decides which coding skill to load next.

Do not use the full workflow for:

- One-line obvious edits.
- Pure documentation tweaks.
- Mechanical formatting requested explicitly by the user.
- Throwaway experiments that are not meant to become production code. Use only `spike` there.

For trivial work, auto-downgrade. Make the smallest safe change, run the cheapest meaningful verification, and stop.

## The Router

Start every coding task by classifying it.

### Trivial edit

Examples:

- typo fix
- one-line config correction
- small comment or README wording change
- obvious local-only patch with no behavior change

Use:

- `karpathy-guidelines`

Process:

1. Inspect the relevant file.
2. Make the smallest edit.
3. Run a cheap check if available.
4. Report the changed file and verification.

Do not force a plan or subagents.

### Clear behavior change

Examples:

- add validation
- add a small endpoint
- change one function's behavior
- fix a bug with obvious scope

Use:

- `karpathy-guidelines`
- `test-driven-development`
- `systematic-debugging` if it is a bug or failure

Process:

1. Read the existing pattern and tests.
2. State hypothesis, success criteria, independent failure signals, and minimal verification path.
3. Write or update the failing test first when behavior changes.
4. Run the test and verify RED.
5. Implement the smallest code to make it pass.
6. Run the specific test and relevant regression tests.
7. Clean only your own orphan imports, variables, or helper code.

### Ambiguous or architectural work

Examples:

- unclear requirements
- public API decisions
- migration or data model change
- module-boundary changes
- multiple reasonable implementations

Use:

- `grill-plan`
- `karpathy-guidelines`
- `writing-plans` if a detailed execution plan is needed

Process:

1. Inspect repository, tests, docs, logs, and existing plans before asking questions.
2. Ask only questions that cannot be answered from available context.
3. Ask one focused question at a time, with a recommended answer.
4. Resolve goal, non-goals, user-visible behavior, boundaries, risks, and tests.
5. Save the plan under `.hermes/plans/YYYY-MM-DD_HHMMSS-<slug>.md`.
6. Review the plan before implementation.

### Multi-task implementation

Examples:

- feature spanning multiple modules
- planned refactor
- parallelizable implementation tasks
- context-heavy work

Use:

- `writing-plans` or a plan produced by `grill-plan`
- `subagent-driven-development`
- `test-driven-development`
- `requesting-code-review` at the end

Process:

1. Read the saved plan once.
2. Extract all tasks and create a todo list.
3. Dispatch one implementer subagent per independent task with full task context.
4. Require TDD instructions in implementer context when behavior changes.
5. Run spec compliance review first.
6. Run code quality review only after spec passes.
7. Verify side effects yourself. Do not trust subagent self-reports.
8. After all tasks pass, run integration review and final verification.
9. Commit only after the pre-commit gate passes.

### Bug, test failure, or production issue

Use:

- `systematic-debugging`
- `test-driven-development`
- `karpathy-guidelines`
- subagents if the system has independent components to inspect

Process:

1. Read the full error, stack trace, logs, and failing test output.
2. Reproduce consistently.
3. Check recent changes and git diff.
4. Trace data flow across component boundaries.
5. Form one root-cause hypothesis.
6. Test the hypothesis minimally.
7. Write a regression test that fails for the bug.
8. Fix the root cause, not the symptom.
9. Run the regression test and broader tests.
10. If multiple fixes fail, stop and reassess the architecture.

### Feasibility unknown

Use:

- `spike`
- subagents for parallel comparison spikes when useful

Process:

1. Decompose the idea into 2-5 feasibility questions.
2. Test the riskiest question first.
3. Build disposable proof-of-concept code under `spikes/`.
4. Record verdict: `VALIDATED`, `PARTIAL`, or `INVALIDATED`.
5. Only turn validated knowledge into production plans.

## Pre-Coding Expectations

Before writing production code for non-trivial work, record this section in the response or in the plan:

```md
## Pre-coding expectations

### Hypothesis

What I believe is true about the system and why this change should work.

### Success criteria

Observable checks that prove the task is done.

### Failure signals

Independent signs that the approach is wrong or unsafe. Do not phrase these as merely "success criteria not met".

### Ablations and expected observations

For each meaningful alternative or variable, what I expect to observe.

### Minimal verification path

The cheapest test, command, API call, UI action, or log check that proves the change.
```

Skip the full section for trivial tasks, but still know what would count as success.

## Plan Shape

A non-trivial plan should include:

```md
# <Task> Implementation Plan

## Goal

## Non-goals

## Context discovered from code/docs/logs

## Pre-coding expectations

### Hypothesis
### Success criteria
### Failure signals
### Ablations and expected observations
### Minimal verification path

## Confirmed decisions

## Rejected alternatives

## Implementation steps

1. <Action> -> verify: <check>
2. <Action> -> verify: <check>
3. <Action> -> verify: <check>

## Files likely to change

## Tests and validation

## Risks and rollback

## Review notes
```

Every implementation step must be written as action -> verification. Weak plans like "make it work" are not acceptable.

## Subagent Protocol

Use subagents to keep the main context clean and to parallelize independent work. Do not use them to outsource judgment.

Rules:

- One subagent, one focused task.
- Pass exact file paths, commands, constraints, and expected output.
- Give the subagent enough context so it does not have to rediscover the entire plan.
- Do not let implementer subagents commit.
- Do not ask a subagent to make irreversible external changes unless the user explicitly approved that scope.
- Verify any claimed side effect yourself.
- Main agent owns synthesis, tradeoffs, final verification, and user-facing conclusions.

Useful subagent roles:

- repository scout
- implementation worker
- spec compliance reviewer
- code quality reviewer
- debugging investigator
- integration reviewer

## Review and Completion Gate

Before saying done:

- Required tests or smoke checks have run.
- If tests could not run, the reason is explicit and the next best verification was done.
- Diff is minimal and directly traceable to the user's request.
- No unrelated refactor or formatting drift.
- No orphan imports, variables, files, configs, or TODOs created by the change.
- Logs, API responses, UI behavior, or test output support the claim.
- Security-sensitive changes got an independent review.
- Reusable lessons were saved to the right place: memory, skill, or project-local lesson docs.

Before commit, push, ship, or PR:

1. Run targeted tests.
2. Run broader tests where reasonable.
3. Inspect `git diff` and `git status`.
4. Run `requesting-code-review` when risk is meaningful.
5. Scan for secrets and local-only data.
6. Commit only after verification passes.

## Persistence Layers

Put information in the right place:

- Current-turn progress -> todo tool.
- Non-trivial implementation plan -> `.hermes/plans/`.
- User preference or durable environment fact -> memory.
- Reusable workflow, command sequence, or trap -> skill.
- Project-specific lesson -> existing project docs such as `tasks/lessons.md`, only if the repo uses that convention.

Do not create process litter. A one-off task does not need new `tasks/` files in a repo that does not already use them.

## Common Pitfalls

1. **Doing the workflow by appearance only.** A plan that does not guide verification is theater.
2. **Asking before inspecting.** If code, tests, docs, or logs can answer it, read them first.
3. **Overbuilding.** Do not add configurability, abstractions, or edge-case handling that the task does not require.
4. **Refactoring nearby code.** If it is unrelated, mention it but do not touch it.
5. **Tests after implementation.** For behavior changes, test-first is the default because tests-after are biased by the implementation.
6. **Random debugging.** Three stacked guesses are worse than one careful root-cause pass.
7. **Trusting subagent self-reports.** Verify files, diffs, commands, URLs, and test results yourself.
8. **Calling it done without proof.** If no command, test, log, API response, or UI check supports the claim, it is not done.

## Quick Reference

| Situation | Load | Output |
| --- | --- | --- |
| Tiny obvious edit | `karpathy-guidelines` | minimal patch + cheap verification |
| Clear behavior change | `karpathy-guidelines`, `test-driven-development` | failing test, minimal code, passing tests |
| Ambiguous feature | `grill-plan`, `karpathy-guidelines` | clarified decisions + `.hermes/plans/...md` |
| Clear multi-step feature | `writing-plans`, `karpathy-guidelines` | executable implementation plan |
| Multi-task execution | `subagent-driven-development`, `test-driven-development` | reviewed task implementations |
| Bug or failure | `systematic-debugging`, `test-driven-development` | root cause + regression test + fix |
| Feasibility unknown | `spike` | disposable prototype + verdict |
| Before commit/push/ship | `requesting-code-review` | security/quality/test gate |

## Verification Checklist

- [ ] Task was correctly classified.
- [ ] Relevant companion skills were loaded.
- [ ] Non-trivial coding had hypothesis, success criteria, independent failure signals, ablations when relevant, and minimal verification path.
- [ ] Plan, if needed, was saved under `.hermes/plans/`.
- [ ] Behavior changes used TDD or an explicit justified alternative.
- [ ] Bugs were debugged to root cause.
- [ ] Subagents, if used, had focused tasks and their outputs were verified.
- [ ] Final response cites actual verification performed.
