# 決策與待確認議題

## 已決策

### D-001 文件拆分策略

- 決議：改用 `docs/grafana/` 多檔維護，`docs/grafana-architecture-context.md` 只做入口。
- 原因：降低單一大檔過時風險，便於多人協作與 AI 引用。

### D-002 查詢上限統一策略（高準確 + 防爆量）

- 決議：每月可達百萬筆交易時，**不允許因上限而犧牲統計準確度**。
- 策略：
  - 聚合類 API（alert / timeseries）以 ES 聚合為主，不做抽樣截斷。
  - 統一限制「輸出資料點數」而非「原始交易筆數」。
  - 大時間窗自動放大 interval（auto interval），避免回傳過多桶。
  - 明細型 API 才用 row cap，且需標示「結果已截斷」。
- 目標：準確度優先、效能可控、行為可預期。

### D-003 時間語意統一與端到端對齊

- 決議：Grafana 相關 API 時間一律以 **epoch millis + UTC** 為準。
- 規範：
  - `from_time`、`to_time`：UTC epoch ms。
  - 區間語意統一為 `[from_time, to_time]`（含端點）。
  - 預設 `interval = 1h`（未傳時套用）。
  - 若 API 有特殊時間策略（例如 DDoS 固定最近 2 分鐘），需明確標示於 API 盤點表。
- 端到端：Java 產生 iframe URL 參數、Vue 傳遞參數、Grafana query body 三者必須同規格。

### D-004 告警權限與 Token/API Key 邊界

- 決議：`X-API-Key` 專供 alert 背景呼叫，不走 dashboard render flow。
- 規範：
  - Alert API 允許 `X-API-Key` 驗證，不依賴使用者登入 token。
  - Dashboard/前台互動走既有 `hitrustSsoToken` 與 RBAC。
  - Alert API Key 視為長效金鑰（不走短期 session 到期機制）。

### D-005 ACS / 3DSS 告警能力差異

- 決議：ACS 與 3DSS 為不同系統，告警數量與類型不同屬正常。
- 現階段約束：dashboard / alert rule 暫不新增，僅做一致性整理與重構。

### D-008 架構一致性優先於功能一致性

- 決議：重構目標是讓 ACS 與 3DSS 在程式架構與維護方式一致，不強求功能項目完全一致。
- 理由：兩系統雖互通，但客戶目標與業務範圍不同（EMV 3DS 生態下角色不同）。
- 實務要求：
  - 同類邏輯採一致分層與命名模式。
  - 功能差異要被文件化，不靠程式註解零散描述。
  - 回歸驗證流程一致，驗證內容可依系統差異調整。

### D-009 Alert 與 Dashboard 流程強制分流

- 決議：Alert 與 Dashboard 為不同存取流程，程式層不得透過「單一工具自動判斷目前是 alert 還是 dashboard」來決定行為。
- 原因：混流工具會讓責任邊界模糊，導致維護與除錯困難。
- 實務要求：
  - Alert API 只走 `X-API-Key` 背景流程。
  - Dashboard API 只走使用者 token + RBAC 流程。
  - Resolver/Utils 依流程分開（例如 AlertResolver、DashboardResolver），不要以同一方法內 `if token/apiKey` 分支混用。

### D-006 共用層切分策略（Q-101）

- 決議：先做「系統內共用」，不先強推跨 ACS/3DSS 合併模組。
- 做法：
  - 先在各自系統內整理 `GrafanaUtils`、request resolver、time/parser、query builder。
  - 跨系統僅共享「規格與測試案例」，非直接共享執行模組。
  - 待兩邊行為完全對齊後，再評估抽出共同 library。

### D-007 前端 API client 對齊策略（Q-102）

- 決議：兩前端維持獨立 repo，但對齊同一份 client contract。
- 做法：
  - 統一 `grafanaUtil` / `stores/grafana` 的介面命名與錯誤處理格式。
  - 統一 iframe 啟動所需 payload 欄位（token、org、time、template vars）。
  - 建立「路由頁 -> backend URL API -> iframe」對齊檢查清單，改任一端都要回歸驗證。

## 待確認（P0）

### Q-001 查詢上限統一值

- 問題：各 API 目前上限不一致或未明訂。
- 目前狀態：已於 D-002 定義方向；尚待把「實際數值」逐條落表。
- 影響檔案：`03-engineering-standards.md`、`04-api-inventory.md`

### Q-002 時間語意統一

- 問題：`from/to`、`from_time/to_time`、`interval` 在不同 API 行為可能不同。
- 目前狀態：已於 D-003 定義格式與邊界；尚待完成全 API 稽核。
- 影響檔案：`03-engineering-standards.md`、`04-api-inventory.md`

### Q-003 權限驗證優先順序

- 問題：API Key 與 SSO JWT 的流程需要單一說明。
- 目前狀態：已於 D-004 定義用途邊界；尚待補錯誤碼與 Hosted/Issuer 對照表。
- 影響檔案：`03-engineering-standards.md`、`04-api-inventory.md`

### Q-004 ACS/3DSS 告警 API 對齊範圍

- 問題：ACS 有 DDoS 與 Transaction Insight 黑名單類告警 API，3DSS 目前只有 overview 類對應。
- 目前狀態：已於 D-005 確認為產品差異（現階段不新增）。
- 影響檔案：`04-api-inventory.md`、`01-scope-and-goals.md`

## 待確認（P1）

### Q-101 共用層切分策略

- 目前狀態：已於 D-006 決議（先系統內共用）。

### Q-102 前端 API client 對齊

- 目前狀態：已於 D-007 決議（對齊 contract，不強制合 repo）。

## 記錄格式規範

新增議題請用以下格式：

- `Q-編號`：問題
- 背景
- 候選方案
- 建議方案
- 決策人 / 日期
