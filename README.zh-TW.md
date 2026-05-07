# AgentForge Protocol

<p align="center">
  <strong>給編程 Agent 用的協議。重點不是寫得快，是寫完有證據。</strong>
</p>

<p align="center">
  <em>適用於 Hermes、OpenClaw、Claude Code、Codex CLI，以及任何能讀文件、改代碼、跑測試、派 subagent、檢查自己說法的 Agent。</em>
</p>

<p align="center">
  <a href="https://github.com/Yat-mo/agentforge-protocol"><img alt="GitHub repo" src="https://img.shields.io/badge/repo-agentforge--protocol-2f2a25?style=flat-square"></a>
  <img alt="License" src="https://img.shields.io/badge/license-MIT-c7a27a?style=flat-square">
  <img alt="Agents" src="https://img.shields.io/badge/agents-Hermes%20%7C%20OpenClaw%20%7C%20Claude%20Code%20%7C%20Codex%20CLI-8e6f52?style=flat-square">
</p>

<p align="center">
  <a href="README.md">English</a> ·
  <a href="README.zh-CN.md">简体中文</a> ·
  <a href="README.zh-TW.md">繁體中文</a>
</p>

<p align="center">
  <img src="assets/agentforge-hero.svg" alt="AgentForge Protocol hero" width="100%" />
</p>

> 編程 Agent 很快。這是好玩的地方，也是危險的地方。AgentForge Protocol 保留速度，但要求每一步都留下證據。

---

## 為什麼需要它

它們可以在還沒理解倉庫前就寫出 patch。可以生成一份看起來很整齊、但其實什么也證明不了的計劃。可以开几个 subagent，收下它們的匯報，然后悄悄把一坨東西 ship 出去。

真正用這些工具幹過活的人，大概都見過某個版本的這種場面。

AgentForge Protocol 是一套很小的操作習慣，用來避開這種坑。它告訴 Agent 什麼時候保持轻量，什麼時候該慢下來，什麼時候先寫測試，什麼時候應該 debug 而不是猜，什麼時候可以派 subagent，但不能把方向盤交出去。

**核心很簡單：有意義的步驟，最後都要留下證據。**

這個版本也吸收了架構驅動治理裡有用的部分：先讀 baseline，再判斷影響面；把修復軌和退役軌分開；長任務保留 checkpoint，避免中途跑偏。

---

## 一眼看懂

| 如果任務是... | 協議會這樣處理 |
| --- | --- |
| 很小、很明顯 | inspect，patch，跑最便宜的有效檢查，然後停 |
| 清晰的行為改動 | 先写失敗測試，再让它通过 |
| 模糊或架構類 | 先 inspect，再追问關鍵決定，最后保存計劃 |
| 多步驟任務 | 拆任务，用聚焦 subagent，分阶段 review |
| bug 或測試失敗 | 复现，追根因，加 regression test |
| 不確定能不能做 | 先 spike，不要直接寫進 production architecture |

<p align="center">
  <img src="assets/protocol-loop.svg" alt="Plan, test, build, review, verify loop" width="100%" />
</p>

---

## 它組合了什麼

`agentforge-protocol` 站在幾個更小的 skill 之上，決定當前該由誰主導。

| Skill | 負責什麼 |
| --- | --- |
| `karpathy-guidelines` | 小 diff，少假设，少耍聰明 |
| `grill-plan` | 需求模糊或風險高時，先做決定再寫代碼 |
| `writing-plans` | 把清晰需求變成 Agent 真能執行的步骤 |
| `test-driven-development` | 行為改動先有失敗測試，再有修復 |
| `systematic-debugging` | patch 前先找根因 |
| `subagent-driven-development` | 拆任务，但不丟掉控制權 |
| `requesting-code-review` | commit、push、ship 前的最後一道門 |
| `spike` | 不確定時先做能丟的實驗 |

它用 Hermes 自己的層級，不額外製造文檔垃圾：

- 當前進度进 `todo` tool
- 非琐碎計劃进 `.hermes/plans/`
- 穩定的用戶偏好或環境事實进 memory
- 可重複的流程和坑點变成 skill
- 只有倉庫本來就這麼做時，才使用項目裡的 `tasks/lessons.md`

---

## 安裝

```bash
git clone https://github.com/Yat-mo/agentforge-protocol.git
mkdir -p ~/.hermes/skills/software-development
cp -R agentforge-protocol/skills/software-development/agentforge-protocol \
  ~/.hermes/skills/software-development/
```

啟動新的 Hermes session，讓 skill loader 讀到它：

```bash
hermes --skills agentforge-protocol
```

或在 Hermes 里載入：

```text
/skill agentforge-protocol
```

---

## 快速開始

```text
Use agentforge-protocol. Add email validation to the signup flow.
```

```text
Use agentforge-protocol. Design and implement workspace-level permissions.
```

```text
Use agentforge-protocol. The export job passes locally but fails in CI with a timezone assertion.
```

```text
Use agentforge-protocol. Spike whether we can stream partial PDF extraction results to the UI.
```

---

## 路由器

第一步是判斷任務類型。錯字不需要儀式感，遷移就需要。

<details open>
<summary><strong>琐碎且明顯的改动</strong></summary>

輕處理。

```text
inspect → minimal patch → cheap verification → stop
```

不强行写計劃。不強行派 subagent。不演流程。

</details>

<details open>
<summary><strong>清晰的行為改動</strong></summary>

默認走 TDD，除非真的有理由不走。

```text
read existing pattern
→ define baseline / hypothesis / success / failure / evidence plan
→ 必要時追蹤修復軌 + 退役軌
→ write failing test
→ run RED
→ implement minimal code
→ run GREEN
→ run relevant regression
```

</details>

<details open>
<summary><strong>模糊或架構類任务</strong></summary>

碰 production code 前，先用 grill-plan。

```text
inspect code/docs/tests/logs
→ ask only what cannot be inspected
→ resolve decisions one by one
→ save `.hermes/plans/...md`
→ review plan
→ implement from the plan
```

</details>

<details open>
<summary><strong>多任务实现</strong></summary>

可以用 subagent，但主 Agent 仍然負責結果。

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

</details>

<details open>
<summary><strong>Bug 或測試失敗</strong></summary>

先 debug，再 patch。

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

</details>

<details open>
<summary><strong>可行性未知</strong></summary>

先 spike。不要把不確定性直接寫進 production architecture。

```text
decompose feasibility questions
→ test highest risk first
→ build disposable prototype
→ record VALIDATED / PARTIAL / INVALIDATED
→ only then plan production work
```

</details>

---

## 編碼前預期

非琐碎 production code 動手前，先寫下這些東西：

```md
## Pre-coding expectations

### Baseline read set
動手前要讀的 source of truth、架構邊界、owner、影響面、兼容約束和驗證入口。

### Hypothesis
我認為系統裡什麼是真的，以及為什麼這個改動應該有效。

### Success criteria
哪些檢查能讓我放心說，這事做完了。

### Failure signals
說明方案錯了或不安全的独立信号。
不能只是「成功標準沒通過」。

### Ablations and expected observations
如果換掉某个关键假设或做法，我預期会看到什么。

### Evidence plan
最終結論需要哪些 fresh evidence 支撐：測試、命令、日誌、API 回應、截圖或 diff review。

### Minimal verification path
最便宜的證明方式：測試、命令、API 調用、UI 操作或日誌檢查。
```

這段不是裝嚴謹。它是用來防止 Agent 憑感覺寫代碼的。

---

## 計劃形態

非琐碎計劃放在 `.hermes/plans/`，每一步都寫成 action → verification。

```md
# <Task> Implementation Plan

## Goal

## Non-goals

## Context discovered from code/docs/logs

## Pre-coding expectations

### Baseline read set
### Hypothesis
### Success criteria
### Failure signals
### Ablations and expected observations
### Evidence plan
### Minimal verification path

## Confirmed decisions

## Rejected alternatives

## Fix lane and retirement lane

## Checkpoint, resume hint, and drift check

## Implementation steps

1. <Action> -> verify: <check>
2. <Action> -> verify: <check>
3. <Action> -> verify: <check>

## Files likely to change

## Tests and validation

## Risks and rollback

## Review notes
```

“讓它工作”不是計劃。每一步都要知道怎麼證明自己。

---

## Subagent 規則

Subagent 很有用，也很會說得像真的。

<p align="center">
  <img src="assets/agent-stack.svg" alt="Main agent and focused subagent roles" width="100%" />
</p>

| 規則 | 為什麼重要 |
| --- | --- |
| 一个 subagent 只拿一个聚焦任务 | 寬泛 prompt 会產出寬泛结果 |
| 给清楚路径、命令、约束和預期输出 | fresh context 需要真實上下文 |
| implementer subagent 不 commit | 最終狀態由主 Agent 負責 |
| spec review 先于 code quality review | 先確認有沒有做對事 |
| 主 Agent 亲自验证副作用 | 自述不是證據 |

常見 subagent 角色：repository scout、implementation worker、spec compliance reviewer、code quality reviewer、debugging investigator、integration reviewer。

---

## 完成門禁

Agent 說「完成」之前，先檢查這些無聊但要命的事：

- 測試或 smoke check 真的跑過
- 如果測試跑不了，原因說清楚
- diff 小，而且能對應到用戶請求
- 沒有無關重構或格式漂移
- 沒留下自己造成的 orphan imports、文件、配置或 TODO
- fresh evidence 要明確寫出來，不能暗示
- 日誌、API 回應、UI 行為或測試輸出能支撐結論
- bug fix、重構、contract 調整要解決修復軌和退役軌，或說明剩餘風險
- 長任務或高風險任務要有 checkpoint、resume hint 和 drift check
- 高風險改動經過獨立 review
- 可復用經驗放到了正確地方

commit、push、ship 或 PR 前：

```text
targeted tests
→ broader tests where reasonable
→ git diff / git status
→ secret and local-data scan
→ independent review when risk is meaningful
→ commit only after verification passes
```

---

## Skill 佈局

```text
skills/
└── software-development/
    └── agentforge-protocol/
        └── SKILL.md
```

這個倉庫刻意保持很小。它發布的是一个 workflow skill，不是一個框架。

---

## Philosophy

好的 agentic coding 不是讓模型變慢。

而是讓模型更難被騙。

更難被模糊需求騙。更难被沒什麼證明力的 passing tests 骗。更难被看似合理的 subagent 匯報骗。更难被遮住症狀的补丁骗。更难被看起來很努力的大 diff 骗。

小事足夠小時就保持小。事情可能傷到你時，就系統化處理。

---

## License

MIT
