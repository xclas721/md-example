# 工程規範（重構時必遵守）

本檔是重構期間的「單一規範來源」。未決議項目請先登記到 `06-decisions-and-open-questions.md`。

## 1) API 職責與命名

- 每支 API 僅對應單一明確用途（如特定 dashboard/panel/alert）。
- 避免在 Controller 內堆疊資料轉換邏輯；共用邏輯下沉到 service/util。
- 相同語意使用同一命名，避免 `overview-adv` / `overview_adv` 混用。

## 2) 查詢上限（統一策略）

- 每支查詢 API 必須有明確上限，但**不得因上限犧牲統計準確度**。
- 聚合類 API（alert / timeseries）：限制「輸出點數（bucket）」而非限制原始交易筆數。
- 明細類 API：可做 row cap，但需回傳「是否截斷」訊號。
- 大時間窗需自動調整 interval（auto interval）避免回傳爆量。
- 上限策略需同步寫入 `04-api-inventory.md`（現況與目標）。

## 3) 時間處理一致性

- Grafana 相關 API 時間格式統一為 epoch millis（UTC）。
- 區間語意統一為 `[from_time, to_time]`（含端點）。
- 預設 `interval = 1h`（未指定時）。
- 允許特例（例如 DDoS 固定最近 2 分鐘），但必須在 API 盤點表明確標示。
- Java 產 iframe URL、Vue 傳參、Grafana 查詢 body 三端必須一致。

## 4) 權限與驗證一致性

- 前端路由權限、後端 RBAC、Aspect 驗證邏輯要能對映。
- Grafana 背景告警（無人登入）與使用者操作（SSO JWT）流程要分開描述。
- `X-API-Key` 專供 alert 背景呼叫，不走 dashboard render 流程。
- `X-API-Key`、`hitrustSsoToken`、`X-BANK-ID` / `X-REQUESTOR-ID` 的優先順序需一致且可測。

## 5) 共用邏輯抽取原則

- 抽共用以「相同行為 + 相同輸入輸出」為前提，避免過度抽象。
- 先整理差異再抽，不要直接合併同名類別。
- 共用層要有最小單元測試，避免回歸。
- ACS / 3DSS 以「架構同型」為目標，不以「功能同型」為目標。

## 5.1) 架構一致性檢查（ACS vs 3DSS）

- 同類職責是否都在相同層級（controller/service/utils/aspect）。
- 同類流程是否有對應測試策略（即使功能內容不同）。
- API 權限流程是否都能清楚區分 alert API key 與 dashboard token。
- 文件欄位與命名規則是否一致（例如 inventory 表欄位、狀態定義）。

## 6) 文件同步規則

- 改任一 API：同步更新 `04-api-inventory.md`。
- 改任一規範：同步更新本檔與 `05-progress-status.md`。
- 有未定案事項：先記錄 `06-decisions-and-open-questions.md` 再實作。

## 7) 端到端對齊檢查清單（改動必跑）

- Vue 路由頁是否仍正確呼叫 backend URL 產生 API。
- backend URL 回傳的 token、org、time、template vars 是否完整。
- iframe 載入 Grafana 後，查詢 body 的 `from/to/interval` 是否與預期一致。
- alert API（`X-API-Key`）與 dashboard API（`hitrustSsoToken`）是否未混用。
