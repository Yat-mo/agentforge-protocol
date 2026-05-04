# Hermes Coding Workflow Skill

<p align="center">
  <strong>A complete, verification-first coding workflow for Hermes Agent.</strong>
</p>

<p align="center">
  <em>Karpathy-style engineering discipline, grill-plan intake, TDD, systematic debugging, subagent orchestration, spikes, and pre-commit review in one reusable skill.</em>
</p>

---

## Why this exists

LLM coding agents are fast, but fast is not the same as correct.

The common failure modes are boring and expensive:

- guessing requirements instead of inspecting the codebase
- adding abstractions nobody asked for
- touching nearby code because it looks ugly
- writing tests after the implementation and calling that confidence
- debugging by stacking random patches
- letting subagents produce unverified side effects
- saying “done” before anything actually ran

This skill packages a workflow designed to avoid those traps.

It is cautious when caution buys correctness, but it does not force heavy ceremony onto trivial edits.

---

## What it combines

`hermes-coding-workflow` acts as a router and operating layer over these skills:

- **karpathy-guidelines** — minimalism, assumptions, surgical diffs, verification.
- **grill-plan** — clarify ambiguous or high-risk requirements before implementation.
- **writing-plans** — turn clear requirements into executable implementation plans.
- **test-driven-development** — RED → GREEN → REFACTOR for behavior changes.
- **systematic-debugging** — root-cause debugging before fixes.
- **subagent-driven-development** — fresh subagent per task, reviewed in stages.
- **requesting-code-review** — pre-commit / pre-ship verification gate.
- **spike** — disposable experiments for feasibility unknowns.

The workflow is Hermes-native:

- current progress goes to the `todo` tool
- non-trivial plans go to `.hermes/plans/`
- durable user/environment facts go to memory
- reusable workflows and traps become skills
- project-local `tasks/lessons.md` is used only when the repo already has that convention

---

## Install

Clone this repository:

```bash
git clone https://github.com/Yat-mo/hermes-coding-workflow-skill.git
```

Copy the skill into your Hermes skills directory:

```bash
mkdir -p ~/.hermes/skills/software-development
cp -R hermes-coding-workflow-skill/skills/software-development/hermes-coding-workflow \
  ~/.hermes/skills/software-development/
```

Start a new Hermes session so the skill loader sees it:

```bash
hermes --skills hermes-coding-workflow
```

Or inside Hermes:

```text
/skill hermes-coding-workflow
```

---

## Quick start

Use it whenever a coding task is more than a tiny obvious edit:

```text
Use hermes-coding-workflow. Add email validation to the signup flow.
```

For an ambiguous feature:

```text
Use hermes-coding-workflow. Design and implement workspace-level permissions.
```

For a bug:

```text
Use hermes-coding-workflow. The export job passes locally but fails in CI with a timezone assertion.
```

For a feasibility question:

```text
Use hermes-coding-workflow. Spike whether we can stream partial PDF extraction results to the UI.
```

---

## The router

The skill starts by classifying the work.

### Tiny obvious edit

Use only the lightweight path.

```text
inspect → minimal patch → cheap verification → stop
```

No forced plan. No subagents. No theater.

### Clear behavior change

Use TDD by default.

```text
read existing pattern
→ define hypothesis / success / failure / minimal verification
→ write failing test
→ run RED
→ implement minimal code
→ run GREEN
→ run relevant regression
```

### Ambiguous or architectural work

Use grill-plan first.

```text
inspect code/docs/tests/logs
→ ask only what cannot be inspected
→ resolve decisions one by one
→ save `.hermes/plans/...md`
→ review plan
→ implement from the plan
```

### Multi-task implementation

Use subagent-driven development.

```text
read saved plan once
→ extract tasks
→ implementer subagent per task
→ spec compliance review
→ code quality review
→ integration review
→ final verification
→ pre-commit gate
```

### Bug or test failure

Use systematic debugging.

```text
read full error
→ reproduce
→ inspect recent changes
→ trace data flow
→ form one hypothesis
→ write regression test
→ fix root cause
→ verify
```

### Feasibility unknown

Use spike.

```text
decompose feasibility questions
→ test highest risk first
→ build disposable prototype
→ record VALIDATED / PARTIAL / INVALIDATED
→ only then plan production work
```

---

## Pre-coding expectations

For non-trivial production code, the workflow records this before implementation:

```md
## Pre-coding expectations

### Hypothesis
What I believe is true about the system and why this change should work.

### Success criteria
Observable checks that prove the task is done.

### Failure signals
Independent signs that the approach is wrong or unsafe.
These are not just "success criteria not met".

### Ablations and expected observations
What should happen if we vary a meaningful assumption or approach.

### Minimal verification path
The cheapest test, command, API call, UI action, or log check that proves the change.
```

This is the piece that keeps the agent from coding on vibes.

---

## Plan shape

Non-trivial plans are saved under `.hermes/plans/` and use action → verification steps.

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

Weak plans like “make it work” are rejected. Every step must know how it will prove itself.

---

## Subagent rules

Subagents are useful, but they are not a replacement for judgment.

The workflow uses them like this:

- one subagent, one focused task
- exact paths, commands, constraints, and expected output in the prompt
- implementer subagents do not commit
- spec review happens before code quality review
- side effects are verified by the main agent
- the main agent owns synthesis and final correctness

Good subagent roles:

- repository scout
- implementation worker
- spec compliance reviewer
- code quality reviewer
- debugging investigator
- integration reviewer

---

## Completion gate

Before the agent says “done”, the workflow checks:

- tests or smoke checks actually ran
- if tests could not run, the reason is explicit
- diff is minimal and traceable to the request
- no unrelated refactor or formatting drift
- no orphan imports, files, configs, or TODOs from the change
- logs, API responses, UI behavior, or test output support the claim
- high-risk changes got independent review
- reusable lessons were saved to the right layer

Before commit, push, ship, or PR:

```text
targeted tests
→ broader tests where reasonable
→ git diff / git status
→ secret and local-data scan
→ independent review when risk is meaningful
→ commit only after verification passes
```

---

## Skill layout

```text
skills/
└── software-development/
    └── hermes-coding-workflow/
        └── SKILL.md
```

This repository is intentionally small. It ships one compositional workflow skill, not a pile of framework code.

---

## Philosophy

Good agentic coding is not about making the model slower.

It is about making the model harder to fool:

- harder to fool by ambiguous requirements
- harder to fool by passing tests that prove nothing
- harder to fool by a plausible subagent report
- harder to fool by a patch that hides the symptom
- harder to fool by a big diff that feels productive

Small when small is enough. Systematic when the work can hurt you.

---

## License

MIT
