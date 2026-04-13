# AI 起手提示詞（Cursor 專用）

以下可直接複製到 Cursor Chat / Composer。

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

任務：提出重構方案，不改碼。輸出內容需包含：
1) 分層與模組邊界（Controller/Service/Utils/Aspect）
2) 共用邏輯抽取名單（先 ACS 與 3DSS 各自，再共用）
3) 風險與回滾策略
4) 需要先決議的項目（對應 Q-編號）
補充：不要以「功能等量」為目標，改以「架構同型、維運同型」為目標。
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
好ㄉㄜ好```
