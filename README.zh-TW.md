# AgentForge Protocol

<p align="center">
  <strong>給自主編程 Agent 使用的驗證優先編碼協議。</strong>
</p>

<p align="center">
  <em>適用於 Hermes、OpenClaw、Claude Code、Codex CLI，以及任何能夠讀取、編輯、測試、委派和驗證的 Agent。</em>
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

LLM 編程 Agent 很快，但快不等於正確。

常見失敗模式通常很無聊，卻很昂貴：

- 不看代碼庫就猜需求
- 加入沒人要求的抽象
- 因為旁邊代碼看起來不順眼就順手改掉
- 實作之後再補測試，然後把這叫信心
- 靠堆疊隨機補丁來 debug
- 讓 subagent 產生未驗證的副作用
- 還沒真正跑過任何東西就說「完成了」

這個 skill 把一套避免這些陷阱的工作流打包起來。

它在謹慎能提高正確性時保持謹慎，但不會把重流程強加到琐碎修改上。

---

## 它組合了什麼

`agentforge-protocol` 是這些 skill 之上的路由器和操作層：

- **karpathy-guidelines**，極簡、假設管理、精準 diff、驗證。
- **grill-plan**，在實作前釐清模糊或高風險需求。
- **writing-plans**，把清晰需求寫成可執行的實作計劃。
- **test-driven-development**，為行為改動執行 RED → GREEN → REFACTOR。
- **systematic-debugging**，先找根因，再修復。
- **subagent-driven-development**，每個任務一個新 subagent，並分階段 review。
- **requesting-code-review**，commit 或 ship 前的驗證門禁。
- **spike**，在可行性不確定時做可丟棄實驗。

這套 workflow 是 Hermes-native 的：

- 當前進度進入 `todo` tool
- 非琐碎計劃進入 `.hermes/plans/`
- 長期用戶偏好和環境事實進入 memory
- 可復用流程和坑點沉澱為 skill
- 只有當倉庫本身已有約定時，才使用項目本地的 `tasks/lessons.md`

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

啟動新的 Hermes session，讓 skill loader 看到它：

```bash
hermes --skills agentforge-protocol
```

或在 Hermes 裡載入：

```text
/skill agentforge-protocol
```

---

## 快速開始

當 coding 任務不只是一個顯而易見的小改動時使用它：

```text
Use agentforge-protocol. Add email validation to the signup flow.
```

面對模糊 feature：

```text
Use agentforge-protocol. Design and implement workspace-level permissions.
```

面對 bug：

```text
Use agentforge-protocol. The export job passes locally but fails in CI with a timezone assertion.
```

面對可行性問題：

```text
Use agentforge-protocol. Spike whether we can stream partial PDF extraction results to the UI.
```

---

## 路由器

這個 skill 會先對任務分類。

### 琐碎且顯而易見的改動

只走輕量路徑。

```text
inspect → minimal patch → cheap verification → stop
```

不強制計劃。不強制 subagent。不做流程表演。

### 清晰的行為改動

默認使用 TDD。

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

先使用 grill-plan。

```text
inspect code/docs/tests/logs
→ ask only what cannot be inspected
→ resolve decisions one by one
→ save `.hermes/plans/...md`
→ review plan
→ implement from the plan
```

### 多任務實作

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

### Bug 或測試失敗

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

## 編碼前預期

對於非琐碎 production code，這套 workflow 會在實作前記錄：

```md
## Pre-coding expectations

### Hypothesis
我認為系統裡什麼是真的，以及為什麼這個改動應該有效。

### Success criteria
能夠證明任務完成的可觀察檢查。

### Failure signals
說明方案錯誤或不安全的獨立信號。
這些不能只是「沒有滿足成功標準」。

### Ablations and expected observations
如果改變某個關鍵假設或方案，預期會看到什麼。

### Minimal verification path
能夠證明改動的最便宜測試、命令、API 調用、UI 操作或日誌檢查。
```

這一段的作用是防止 Agent 憑感覺寫代碼。

---

## 計劃形態

非琐碎計劃保存在 `.hermes/plans/`，並使用 action → verification 步驟。

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

像「讓它工作」這種弱計劃會被拒絕。每一步都必須知道如何證明自己。

---

## Subagent 規則

Subagent 很有用，但不能替代判斷。

<p align="center">
  <img src="assets/agent-stack.svg" alt="Main agent and focused subagent roles" width="100%" />
</p>

這套 workflow 會這樣使用它們：

- 一個 subagent 只做一個聚焦任務
- prompt 裡給出精確路徑、命令、約束和預期輸出
- implementer subagent 不 commit
- spec review 先於 code quality review
- 副作用由主 agent 驗證
- 主 agent 負責綜合、判斷和最終正確性

常用 subagent 角色：

- repository scout
- implementation worker
- spec compliance reviewer
- code quality reviewer
- debugging investigator
- integration reviewer

---

## 完成門禁

在 Agent 說「完成」之前，workflow 會檢查：

- 測試或 smoke check 實際運行過
- 如果無法運行測試，原因要明確
- diff 最小，且能追溯到用戶請求
- 沒有無關重構或格式漂移
- 沒有自己造成的 orphan imports、文件、配置或 TODO
- 日誌、API 回應、UI 行為或測試輸出支持結論
- 高風險改動經過獨立 review
- 可復用經驗保存到了正確層級

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

## Skill 佈局

```text
skills/
└── software-development/
    └── agentforge-protocol/
        └── SKILL.md
```

這個倉庫刻意保持很小。它發布的是一個組合式 workflow skill，不是一堆框架代碼。

---

## Philosophy

好的 agentic coding 不是讓模型變慢。

而是讓模型更難被欺騙：

- 更難被模糊需求欺騙
- 更難被沒有證明力的 passing tests 欺騙
- 更難被看似合理的 subagent 匯報欺騙
- 更難被掩蓋症狀的補丁欺騙
- 更難被看起來很有效率的大 diff 欺騙

小事足夠小時就保持小。事情可能傷到你時，就系統化處理。

---

## License

MIT
