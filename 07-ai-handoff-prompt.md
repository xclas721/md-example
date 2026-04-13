# AI 起手提示詞（Cursor 專用）

以下可直接複製到 Cursor Chat / Composer。

## 0. B 階段最小版（建議）

```text
@docs/grafana/00-start-here.md
@docs/grafana/01-scope-and-goals.md
@docs/grafana/03-engineering-standards.md
@docs/grafana/04-api-inventory.md
@docs/grafana/06-decisions-and-open-questions.md
@docs/grafana/11-flow-separation-design.md

請提出 B 階段設計方案（不改碼）。
限制：
1) Alert 與 Dashboard 必須分流，不可單一工具自動判斷模式。
2) ACS/3DSS 功能可不同，但架構與維護模式需一致。
3) 不要把 09/10 當正式規格，只能當示例參考。
```

## A. 盤點模式（先不改碼）

```text
請先閱讀以下文件，依順序：
@docs/grafana-architecture-context.md
@docs/grafana/README.md
@docs/grafana/01-scope-and-goals.md
@docs/grafana/02-architecture-map.md
@docs/grafana/03-engineering-standards.md
@docs/grafana/04-api-inventory.md
@docs/grafana/06-decisions-and-open-questions.md

任務：只做盤點，不改程式碼。請補齊 04-api-inventory.md 的 Controller#method、現況查詢上限、時間參數、權限檢查點，並用表格列出「不一致清單」。
約束：不可臆測未讀到的程式內容；不確定請標記 TBD。
補充：ACS 與 3DSS 功能可不同，請以「架構一致性」為優先，不要硬湊同功能。
```

## B. 設計模式（先提方案）

```text
請先閱讀：
@docs/grafana-architecture-context.md
@docs/grafana/01-scope-and-goals.md
@docs/grafana/03-engineering-standards.md
@docs/grafana/04-api-inventory.md
@docs/grafana/06-decisions-and-open-questions.md
@docs/grafana/11-flow-separation-design.md
@docs/grafana/12-b-phase-readiness-check.md

任務：提出重構方案，不改碼。輸出內容需包含：
1) 分層與模組邊界（Controller/Service/Utils/Aspect）
2) 共用邏輯抽取名單（先 ACS 與 3DSS 各自，再共用）
3) 風險與回滾策略
4) 需要先決議的項目（對應 Q-編號）
補充：不要以「功能等量」為目標，改以「架構同型、維運同型」為目標。
限制：除非使用者明確指定，B 模式不要引用 `09-parity-example-overview-alert.md` 與 `10-parity-example-dimp-alert.md`，避免把示例內容當正式規格。
限制：設計時必須把 Alert 與 Dashboard 視為兩條流程，不可提出「單一工具自動判斷模式」方案。
限制：輸出設計方案前先給出 B 階段 Go/No-Go 判定（依 `12` 檢查表）。
```

## C. 實作模式（開始改碼）

```text
請先閱讀：
@docs/grafana-architecture-context.md
@docs/grafana/03-engineering-standards.md
@docs/grafana/04-api-inventory.md
@docs/grafana/05-progress-status.md
@docs/grafana/06-decisions-and-open-questions.md

任務：依「已決策內容」實作第一批重構（先最小可驗證範圍）。每改一類 API 請同步更新 docs/grafana/04-api-inventory.md 與 docs/grafana/05-progress-status.md。
約束：先小步提交，避免一次大改。
補充：若 ACS 與 3DSS 出現功能差異，保留差異並文件化，不強制做功能對齊。
```
