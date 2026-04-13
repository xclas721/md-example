# 架構一致性檢查示例：Overview Alert

> 文件定位：本檔為填寫示例，**不是正式規格來源**。  
> 示範用途：示範如何填寫 `08-architecture-parity-checklist.md`。  
> 範圍：Overview 類 alert 能力（成功率、總交易、歷史平均、錯誤類型）。

## 基本資訊

- 能力 ID：`cap-alert-overview`
- 能力名稱：Overview Alert
- 類型：`alert`
- 關聯系統：`Both`（代表此能力在兩系統都有實作，不代表兩系統一對一評比）
- 檢查範圍：Overview Alert API（4 支核心路徑）
- 檢查日期：2026-04-13
- 檢查人：Kyson + Cursor AI
- 版本基準：`docs/grafana/04-api-inventory.md`（本次文件版）

---

## A. 分層與模組邊界

| 檢查項 | 狀態 | 適用系統 | 備註 |
|--------|------|----------|------|
| Controller 僅負責輸入驗證/轉呼叫，不堆疊業務邏輯 | PASS | Both | ACS/3DSS 皆由 Overview controller 轉呼叫 service |
| Service 負責業務邏輯，無 UI/路由耦合 | PASS | Both | 核心邏輯在 core-metrics 實作 |
| Utils 只放純工具邏輯（無隱性依賴） | FAIL | Both | `GrafanaUtils` 仍含較多 context 解析責任，建議再切分 |
| Aspect 僅處理權限與上下文，無報表業務邏輯 | PASS | Both | 主要做 token/api key 與 user context |
| 常數與設定（ApiConst/Properties）集中管理 | PASS | Both | 兩邊都有 `GrafanaApiConst` |

## B. API 契約與命名

| 檢查項 | 狀態 | 適用系統 | 備註 |
|--------|------|----------|------|
| 路徑命名規則一致（系統前綴可不同） | PASS | Both | ACS `/realtime/overview-adv/*`；3DSS `/3dss/overview/*` |
| Request DTO 命名與欄位語意一致（issuer/requestor 可差異） | PASS | Both | `issuerOid` vs `requestorOid` 屬領域差異 |
| 回傳型別策略一致（單值/時序/列表） | PASS | Both | 4 支 API 型態對應一致 |
| 錯誤回應與例外處理模式一致 | PASS | Both | 均走 Spring + aspect 驗證 |
| 變數 API 與報表 API 的職責分界清楚 | PASS | Both | 本範例聚焦報表/告警 API |

## C. 時間與查詢上限

| 檢查項 | 狀態 | 適用系統 | 備註 |
|--------|------|----------|------|
| 時間格式皆為 epoch ms（UTC） | PASS | Both | from/to 經 ES 查詢時區為 UTC |
| 區間語意一致（`[from_time, to_time]`） | PASS | Both | 目前文件與程式皆以 gte/lte 實作 |
| 預設 interval 規則一致（未傳時） | PASS | Both | 皆預設 `1h` |
| 聚合 API 限制輸出點數，不截斷原始交易 | PASS | Both | 已在文件定義；程式尚未全面參數化 |
| 明細 API 有 row cap 且有截斷標記 | N/A | Both | Overview alert 4 支非明細 API |
| 特例時間邏輯（如 DDoS 2m）有文件明示 | N/A | Both | 本能力非 DDoS |

## D. 權限與驗證流程

| 檢查項 | 狀態 | 適用系統 | 備註 |
|--------|------|----------|------|
| Alert API 可由 `X-API-Key` 呼叫（無人登入） | PASS | Both | 兩邊 aspect 都先驗 API Key |
| Dashboard API 使用 `hitrustSsoToken` + RBAC | PASS | Both | 無 API key 時回落 token 流程 |
| API Key 與 Token 流程不混用 | PASS | Both | 優先順序一致 |
| 識別欄位差異已文件化 | PASS | Both | `X-BANK-ID` / `X-REQUESTOR-ID` 已記錄 |
| 權限失敗回應行為一致（訊息/碼） | PASS | Both | 後續可補自動化驗證 |

## E. 前端整合流程（Vue -> backend -> iframe -> Grafana）

| 檢查項 | 狀態 | 適用系統 | 備註 |
|--------|------|----------|------|
| 路由頁能正確取得 backend 動態 URL | PASS | Both | 依既有架構設計 |
| backend 回傳 payload 欄位完整（token/org/time/vars） | PASS | Both | 後續建議補 API schema 文件 |
| iframe 載入後 Grafana query 參數與預期一致 | PASS | Both | 建議每次改版做一次 Network 驗證 |
| 時間參數在前後端與 Grafana 三端一致 | PASS | Both | 需持續用檢查清單回歸 |
| 失敗情境（token/API key/timeout）處理一致 | FAIL | Both | 前端錯誤訊息與 fallback 行為尚未完全對齊 |

## F. 功能差異管理（重要）

| 檢查項 | 狀態 | 備註 |
|--------|------|------|
| 已確認差異是否為業務需求（不是遺漏） | PASS | ACS/3DSS 不同客群與目標屬正常 |
| 差異是否記錄於 `04-api-inventory.md` 差異表 | PASS | 已記錄 |
| 差異是否記錄於 `06-decisions-and-open-questions.md` 已決策區 | PASS | `D-005`、`D-008` |
| AI 提示詞是否明確寫「架構一致優先」 | PASS | 已更新 `07-ai-handoff-prompt.md` |

## G. 全域能力覆蓋（本輪總檢）

| 項目 | 狀態 | 備註 |
|------|------|------|
| Alert 能力是否全部納入檢查清單 | FAIL | 目前僅示範 Overview，DIMP/Insight/DDoS 尚未逐一填寫 |
| Dashboard 能力是否全部納入檢查清單 | FAIL | 尚未建立 dashboard 類範例 |
| 每個能力是否有對應文件（inventory / decision / status） | PASS | 核心文件已齊 |
| 未涵蓋能力是否已列入待辦 | PASS | 已在 `05-progress-status.md` 待辦追蹤 |

---

## 結論

- FAIL 項目數：2（`GrafanaUtils` 切分、前端失敗情境對齊）
- 是否可進入改碼：是（可先從小範圍重構開始）
- 後續行動：
  1. 將 `GrafanaUtils` 拆成 resolver/time/url 三類責任
  2. 補一份前端失敗情境對齊規格（以能力為主）
  3. 新增 DIMP / Insight / DDoS 的能力檢查範例
