# AgentForge Protocol

<p align="center">
  <strong>A coding protocol for agents that need proof, not vibes.</strong>
</p>

<p align="center">
  <em>Built for Hermes, OpenClaw, Claude Code, Codex CLI, and any agent that can read files, edit code, run tests, delegate work, and check its own claims.</em>
</p>

<p align="center">
  <a href="README.md">English</a> ·
  <a href="README.zh-CN.md">简体中文</a> ·
  <a href="README.zh-TW.md">繁體中文</a>
</p>

<p align="center">
  <img src="assets/agentforge-hero.svg" alt="AgentForge Protocol hero" width="100%" />
</p>

---

## Why this exists

Coding agents are quick. That is the fun part, and also the dangerous part.

They can write a patch before they understand the repo. They can produce a plan that sounds tidy but proves nothing. They can spin up subagents, accept their reports, and quietly ship a mess. Anyone who has used these tools for real work has seen some version of this.

AgentForge Protocol is a compact operating routine for avoiding that failure mode.

It tells the agent when to stay lightweight, when to slow down, when to write tests first, when to debug instead of guessing, and when to bring in subagents without letting them drive the car.

The point is simple: every meaningful step should leave evidence behind.

---

## What it combines

`agentforge-protocol` sits on top of a few smaller skills and decides which one should lead:

- **karpathy-guidelines**, for small diffs, fewer assumptions, and less cleverness.
- **grill-plan**, for vague or risky work that needs decisions before code.
- **writing-plans**, for turning clear requirements into steps an agent can actually execute.
- **test-driven-development**, for behavior changes that need a failing test before a fix.
- **systematic-debugging**, for bugs where guessing will only make the hole deeper.
- **subagent-driven-development**, for splitting work without losing control of the result.
- **requesting-code-review**, for the final gate before commit, push, or ship.
- **spike**, for questions where a disposable experiment beats another paragraph of speculation.

The workflow uses Hermes' own layers instead of inventing extra paperwork:

- current progress goes to the `todo` tool
- non-trivial plans go to `.hermes/plans/`
- stable user or environment facts go to memory
- repeatable procedures and traps become skills
- project-local `tasks/lessons.md` is used only when the repo already works that way

<p align="center">
  <img src="assets/protocol-loop.svg" alt="Plan, test, build, review, verify loop" width="100%" />
</p>

---

## Install

Clone the repo:

```bash
git clone https://github.com/Yat-mo/agentforge-protocol.git
```

Copy the skill into your Hermes skills directory:

```bash
mkdir -p ~/.hermes/skills/software-development
cp -R agentforge-protocol/skills/software-development/agentforge-protocol \
  ~/.hermes/skills/software-development/
```

Start a fresh Hermes session so the skill loader picks it up:

```bash
hermes --skills agentforge-protocol
```

Or load it inside Hermes:

```text
/skill agentforge-protocol
```

---

## Quick start

Use it when the task is bigger than a tiny, obvious edit:

```text
Use agentforge-protocol. Add email validation to the signup flow.
```

For an unclear feature:

```text
Use agentforge-protocol. Design and implement workspace-level permissions.
```

For a bug:

```text
Use agentforge-protocol. The export job passes locally but fails in CI with a timezone assertion.
```

For a feasibility check:

```text
Use agentforge-protocol. Spike whether we can stream partial PDF extraction results to the UI.
```

---

## The router

The first job is to classify the work. A typo does not need a ceremony. A migration does.

### Tiny obvious edit

Keep it light.

```text
inspect → minimal patch → cheap verification → stop
```

No forced plan. No subagents. No theatre.

### Clear behavior change

Use TDD unless there is a real reason not to.

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

Use grill-plan before touching production code.

```text
inspect code/docs/tests/logs
→ ask only what cannot be inspected
→ resolve decisions one by one
→ save `.hermes/plans/...md`
→ review plan
→ implement from the plan
```

### Multi-task implementation

Use subagents, but keep the main agent responsible.

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

Debug first. Patch second.

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

Spike it. Do not turn uncertainty into production architecture.

```text
decompose feasibility questions
→ test highest risk first
→ build disposable prototype
→ record VALIDATED / PARTIAL / INVALIDATED
→ only then plan production work
```

---

## Pre-coding expectations

For non-trivial production code, write down the expectations before coding:

```md
## Pre-coding expectations

### Hypothesis
What I believe is true about the system and why this change should work.

### Success criteria
The checks that would make me comfortable saying this is done.

### Failure signals
Independent signs that the approach is wrong or unsafe.
These cannot just be "the success criteria did not pass".

### Ablations and expected observations
What I expect to see if a meaningful assumption or approach changes.

### Minimal verification path
The cheapest test, command, API call, UI action, or log check that proves the change.
```

This is the part that keeps the agent honest. Not fancy, just useful.

---

## Plan shape

Non-trivial plans live under `.hermes/plans/` and use action → verification steps.

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

A plan that says "make it work" is not a plan. Each step needs a way to prove itself.

---

## Subagent rules

Subagents are useful. They are also very good at sounding confident.

<p align="center">
  <img src="assets/agent-stack.svg" alt="Main agent and focused subagent roles" width="100%" />
</p>

Use them like this:

- one subagent gets one focused task
- include exact paths, commands, constraints, and expected output
- implementer subagents do not commit
- spec review happens before code quality review
- the main agent verifies side effects
- the main agent owns synthesis, judgment, and final correctness

Good subagent roles:

- repository scout
- implementation worker
- spec compliance reviewer
- code quality reviewer
- debugging investigator
- integration reviewer

---

## Completion gate

Before the agent says "done", check the boring stuff:

- tests or smoke checks actually ran
- if tests could not run, the reason is clear
- the diff is small and tied to the request
- there is no unrelated refactor or formatting drift
- the change did not leave behind orphan imports, files, configs, or TODOs
- logs, API responses, UI behavior, or test output support the claim
- risky changes got independent review
- reusable lessons were saved in the right place

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
    └── agentforge-protocol/
        └── SKILL.md
```

The repo is intentionally small. It ships one workflow skill, not a framework.

---

## Philosophy

Good agentic coding is not about making the model slower.

It is about making the model harder to fool.

Harder to fool with vague requirements. Harder to fool with tests that pass but prove nothing. Harder to fool with a plausible subagent report. Harder to fool with a patch that hides the symptom. Harder to fool with a big diff that feels productive.

Small when small is enough. Systematic when the work can hurt you.

---

## License

MIT
