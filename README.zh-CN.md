# AgentForge Protocol

<p align="center">
  <strong>给自主编程 Agent 使用的验证优先编码协议。</strong>
</p>

<p align="center">
  <em>适用于 Hermes、OpenClaw、Claude Code、Codex CLI，以及任何能够读取、编辑、测试、委派和验证的 Agent。</em>
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

## 为什么需要它

LLM 编程 Agent 很快，但快不等于正确。

常见失败模式通常很无聊，却很昂贵：

- 不看代码库就猜需求
- 添加没人要求的抽象
- 因为旁边代码看起来不顺眼就顺手改掉
- 实现之后再补测试，然后把这叫信心
- 靠堆叠随机补丁来 debug
- 让 subagent 产生未经验证的副作用
- 还没真正跑过任何东西就说“完成了”

这个 skill 把一套避免这些陷阱的工作流打包起来。

它在谨慎能提高正确性时保持谨慎，但不会把重流程强加到琐碎修改上。

---

## 它组合了什么

`agentforge-protocol` 是这些 skill 之上的路由器和操作层：

- **karpathy-guidelines**，极简、假设管理、精准 diff、验证。
- **grill-plan**，在实现前澄清模糊或高风险需求。
- **writing-plans**，把清晰需求写成可执行的实现计划。
- **test-driven-development**，为行为改动执行 RED → GREEN → REFACTOR。
- **systematic-debugging**，先找根因，再修复。
- **subagent-driven-development**，每个任务一个新 subagent，并分阶段 review。
- **requesting-code-review**，commit 或 ship 前的验证门禁。
- **spike**，在可行性不确定时做可丢弃实验。

这套 workflow 是 Hermes-native 的：

- 当前进度进入 `todo` tool
- 非琐碎计划进入 `.hermes/plans/`
- 长期用户偏好和环境事实进入 memory
- 可复用流程和坑点沉淀为 skill
- 只有当仓库本身已有约定时，才使用项目本地的 `tasks/lessons.md`

<p align="center">
  <img src="assets/protocol-loop.svg" alt="Plan, test, build, review, verify loop" width="100%" />
</p>

---

## 安装

克隆仓库：

```bash
git clone https://github.com/Yat-mo/agentforge-protocol.git
```

把 skill 复制到 Hermes skills 目录：

```bash
mkdir -p ~/.hermes/skills/software-development
cp -R agentforge-protocol/skills/software-development/agentforge-protocol \
  ~/.hermes/skills/software-development/
```

启动新的 Hermes session，让 skill loader 看到它：

```bash
hermes --skills agentforge-protocol
```

或在 Hermes 里加载：

```text
/skill agentforge-protocol
```

---

## 快速开始

当 coding 任务不只是一个显而易见的小改动时使用它：

```text
Use agentforge-protocol. Add email validation to the signup flow.
```

面对模糊 feature：

```text
Use agentforge-protocol. Design and implement workspace-level permissions.
```

面对 bug：

```text
Use agentforge-protocol. The export job passes locally but fails in CI with a timezone assertion.
```

面对可行性问题：

```text
Use agentforge-protocol. Spike whether we can stream partial PDF extraction results to the UI.
```

---

## 路由器

这个 skill 会先对任务分类。

### 琐碎且显而易见的改动

只走轻量路径。

```text
inspect → minimal patch → cheap verification → stop
```

不强制计划。不强制 subagent。不做流程表演。

### 清晰的行为改动

默认使用 TDD。

```text
read existing pattern
→ define hypothesis / success / failure / minimal verification
→ write failing test
→ run RED
→ implement minimal code
→ run GREEN
→ run relevant regression
```

### 模糊或架构类任务

先使用 grill-plan。

```text
inspect code/docs/tests/logs
→ ask only what cannot be inspected
→ resolve decisions one by one
→ save `.hermes/plans/...md`
→ review plan
→ implement from the plan
```

### 多任务实现

使用 subagent-driven development。

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

### Bug 或测试失败

使用 systematic debugging。

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

### 可行性未知

使用 spike。

```text
decompose feasibility questions
→ test highest risk first
→ build disposable prototype
→ record VALIDATED / PARTIAL / INVALIDATED
→ only then plan production work
```

---

## 编码前预期

对于非琐碎 production code，这套 workflow 会在实现前记录：

```md
## Pre-coding expectations

### Hypothesis
我认为系统里什么是真的，以及为什么这个改动应该有效。

### Success criteria
能够证明任务完成的可观察检查。

### Failure signals
说明方案错误或不安全的独立信号。
这些不能只是“没有满足成功标准”。

### Ablations and expected observations
如果改变某个关键假设或方案，预期会看到什么。

### Minimal verification path
能够证明改动的最便宜测试、命令、API 调用、UI 操作或日志检查。
```

这一段的作用是防止 Agent 凭感觉写代码。

---

## 计划形态

非琐碎计划保存在 `.hermes/plans/`，并使用 action → verification 步骤。

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

像“让它工作”这种弱计划会被拒绝。每一步都必须知道如何证明自己。

---

## Subagent 规则

Subagent 很有用，但不能替代判断。

<p align="center">
  <img src="assets/agent-stack.svg" alt="Main agent and focused subagent roles" width="100%" />
</p>

这套 workflow 会这样使用它们：

- 一个 subagent 只做一个聚焦任务
- prompt 里给出精确路径、命令、约束和预期输出
- implementer subagent 不 commit
- spec review 先于 code quality review
- 副作用由主 agent 验证
- 主 agent 负责综合、判断和最终正确性

常用 subagent 角色：

- repository scout
- implementation worker
- spec compliance reviewer
- code quality reviewer
- debugging investigator
- integration reviewer

---

## 完成门禁

在 Agent 说“完成”之前，workflow 会检查：

- 测试或 smoke check 实际运行过
- 如果无法运行测试，原因要明确
- diff 最小，且能追溯到用户请求
- 没有无关重构或格式漂移
- 没有自己造成的 orphan imports、文件、配置或 TODO
- 日志、API 响应、UI 行为或测试输出支持结论
- 高风险改动经过独立 review
- 可复用经验保存到了正确层级

在 commit、push、ship 或 PR 前：

```text
targeted tests
→ broader tests where reasonable
→ git diff / git status
→ secret and local-data scan
→ independent review when risk is meaningful
→ commit only after verification passes
```

---

## Skill 布局

```text
skills/
└── software-development/
    └── agentforge-protocol/
        └── SKILL.md
```

这个仓库刻意保持很小。它发布的是一个组合式 workflow skill，不是一堆框架代码。

---

## Philosophy

好的 agentic coding 不是让模型变慢。

而是让模型更难被欺骗：

- 更难被模糊需求欺骗
- 更难被没有证明力的 passing tests 欺骗
- 更难被看似合理的 subagent 汇报欺骗
- 更难被掩盖症状的补丁欺骗
- 更难被看起来很有效率的大 diff 欺骗

小事足够小时就保持小。事情可能伤到你时，就系统化处理。

---

## License

MIT
