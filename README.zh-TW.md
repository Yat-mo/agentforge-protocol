# AgentForge Protocol

<p align="center">
  <strong>給編程 Agent 用的協議。重點不是寫得快，是寫完有證據。</strong>
</p>

<p align="center">
  <em>適用於 Hermes、OpenClaw、Claude Code、Codex CLI，以及任何能讀文件、改代碼、跑測試、派 subagent、檢查自己說法的 Agent。</em>
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

## 為什麼需要它

編程 Agent 很快。這是好玩的地方，也是危險的地方。

它們可以在還沒理解倉庫前就寫出 patch。可以生成一份看起來很整齊、但其實什麼也證明不了的計劃。可以開幾個 subagent，收下它們的匯報，然後悄悄把一坨東西 ship 出去。真正用這些工具幹過活的人，大概都見過某個版本的這種場面。

AgentForge Protocol 是一套很小的操作習慣，用來避開這種坑。

它告訴 Agent 什麼時候保持輕量，什麼時候該慢下來，什麼時候先寫測試，什麼時候應該 debug 而不是猜，什麼時候可以派 subagent，但不能把方向盤交出去。

核心很簡單：有意義的步驟，最後都要留下證據。

---

## 它組合了什麼

`agentforge-protocol` 站在幾個更小的 skill 之上，決定當前該由誰主導：

- **karpathy-guidelines**，小 diff，少假設，少耍聰明。
- **grill-plan**，需求模糊或風險高時，先把決定問清楚再寫代碼。
- **writing-plans**，把清晰需求變成 Agent 真能執行的步驟。
- **test-driven-development**，行為改動先有失敗測試，再有修復。
- **systematic-debugging**，bug 先找根因，不靠猜。
- **subagent-driven-development**，拆任務可以，但不能丟掉控制權。
- **requesting-code-review**，commit、push、ship 前的最後一道門。
- **spike**，不確定能不能做時，先做一個能丟的實驗，比繼續空談強。

它用 Hermes 自己的層級，不額外製造文檔垃圾：

- 當前進度進 `todo` tool
- 非琐碎計劃進 `.hermes/plans/`
- 穩定的用戶偏好或環境事實進 memory
- 可重複的流程和坑點變成 skill
- 只有倉庫本來就這麼做時，才使用項目裡的 `tasks/lessons.md`

<p align="center">
  <img src="assets/protocol-loop.svg" alt="Plan, test, build, review, verify loop" width="100%" />
</p>

---

## 安裝

克隆倉庫：

```bash
git clone https://github.com/Yat-mo/agentforge-protocol.git
```

把 skill 複製到 Hermes skills 目錄：

```bash
mkdir -p ~/.hermes/skills/software-development
cp -R agentforge-protocol/skills/software-development/agentforge-protocol \
  ~/.hermes/skills/software-development/
```

啟動新的 Hermes session，讓 skill loader 讀到它：

```bash
hermes --skills agentforge-protocol
```

或在 Hermes 裡載入：

```text
/skill agentforge-protocol
```

---

## 快速開始

當任務不只是一個很明顯的小改動時，用它：

```text
Use agentforge-protocol. Add email validation to the signup flow.
```

需求還不清楚時：

```text
Use agentforge-protocol. Design and implement workspace-level permissions.
```

遇到 bug 時：

```text
Use agentforge-protocol. The export job passes locally but fails in CI with a timezone assertion.
```

想先確認可不可行時：

```text
Use agentforge-protocol. Spike whether we can stream partial PDF extraction results to the UI.
```

---

## 路由器

第一步是判斷任務類型。錯字不需要儀式感，遷移就需要。

### 琐碎且明顯的改動

輕處理。

```text
inspect → minimal patch → cheap verification → stop
```

不強行寫計劃。不強行派 subagent。不演流程。

### 清晰的行為改動

默認走 TDD，除非真的有理由不走。

```text
read existing pattern
→ define hypothesis / success / failure / minimal verification
→ write failing test
→ run RED
→ implement minimal code
→ run GREEN
→ run relevant regression
```

### 模糊或架構類任務

碰 production code 前，先用 grill-plan。

```text
inspect code/docs/tests/logs
→ ask only what cannot be inspected
→ resolve decisions one by one
→ save `.hermes/plans/...md`
→ review plan
→ implement from the plan
```

### 多任務實作

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

### Bug 或測試失敗

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

### 可行性未知

先 spike。不要把不確定性直接寫進 production architecture。

```text
decompose feasibility questions
→ test highest risk first
→ build disposable prototype
→ record VALIDATED / PARTIAL / INVALIDATED
→ only then plan production work
```

---

## 編碼前預期

非琐碎 production code 動手前，先寫下這些東西：

```md
## Pre-coding expectations

### Hypothesis
我認為系統裡什麼是真的，以及為什麼這個改動應該有效。

### Success criteria
哪些檢查能讓我放心說，這事做完了。

### Failure signals
說明方案錯了或不安全的獨立信號。
不能只是「成功標準沒通過」。

### Ablations and expected observations
如果換掉某個關鍵假設或做法，我預期會看到什麼。

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

「讓它工作」不是計劃。每一步都要知道怎麼證明自己。

---

## Subagent 規則

Subagent 很有用，也很會說得像真的。

<p align="center">
  <img src="assets/agent-stack.svg" alt="Main agent and focused subagent roles" width="100%" />
</p>

這樣用：

- 一個 subagent 只拿一個聚焦任務
- 給清楚路徑、命令、約束和預期輸出
- implementer subagent 不 commit
- spec review 先於 code quality review
- 主 Agent 親自驗證副作用
- 主 Agent 負責綜合、判斷和最終正確性

常見 subagent 角色：

- repository scout
- implementation worker
- spec compliance reviewer
- code quality reviewer
- debugging investigator
- integration reviewer

---

## 完成門禁

Agent 說「完成」之前，先檢查這些無聊但要命的事：

- 測試或 smoke check 真的跑過
- 如果測試跑不了，原因說清楚
- diff 小，而且能對應到用戶請求
- 沒有無關重構或格式漂移
- 沒留下自己造成的 orphan imports、文件、配置或 TODO
- 日誌、API 回應、UI 行為或測試輸出能支撐結論
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

這個倉庫刻意保持很小。它發布的是一個 workflow skill，不是一個框架。

---

## Philosophy

好的 agentic coding 不是讓模型變慢。

而是讓模型更難被騙。

更難被模糊需求騙。更難被沒什麼證明力的 passing tests 騙。更難被看似合理的 subagent 匯報騙。更難被遮住症狀的補丁騙。更難被看起來很努力的大 diff 騙。

小事足夠小時就保持小。事情可能傷到你時，就系統化處理。

---

## License

MIT
