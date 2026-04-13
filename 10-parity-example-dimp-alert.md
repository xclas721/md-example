# 架構一致性檢查示例：DIMP Alert

> 文件定位：本檔為填寫示例，**不是正式規格來源**。  
> 示範用途：示範如何以「能力主軸」填寫 `08-architecture-parity-checklist.md`。  
> 範圍：Mastercard DIMP 相關 alert 能力（Edit1/3/7/8/11）。

## 基本資訊

- 能力 ID：`cap-alert-dimp`
- 能力名稱：DIMP Alert
- 類型：`alert`
- 關聯系統：`ACS`（目前 3DSS 無同能力 API，屬正常差異）
- 檢查範圍：DIMP 5 支 API 與對應 alert 規則
- 檢查日期：2026-04-13
- 檢查人：Kyson + Cursor AI
- 版本基準：`docs/grafana/04-api-inventory.md`（本次文件版）

---

## A. 分層與模組邊界

| 檢查項 | 狀態 | 適用系統 | 備註 |
|--------|------|----------|------|
| Controller 僅負責輸入驗證/轉呼叫，不堆疊業務邏輯 | PASS | ACS | `GrafanaRealTimeMastercardDimpApiController` 只轉呼叫 service |
| Service 負責業務邏輯，無 UI/路由耦合 | PASS | ACS | 邏輯在 core-metrics ES service |
| Utils 只放純工具邏輯（無隱性依賴） | FAIL | ACS | issuer 解析仍與 context 綁定，建議切分 |
| Aspect 僅處理權限與上下文，無報表業務邏輯 | PASS | ACS | `GrafanaAccessControlAspect` 僅處理驗證流程 |
| 常數與設定（ApiConst/Properties）集中管理 | PASS | ACS | `GrafanaApiConst` 使用一致 |

## B. API 契約與命名

| 檢查項 | 狀態 | 適用系統 | 備註 |
|--------|------|----------|------|
| 路徑命名規則一致（系統前綴可不同） | PASS | ACS | `/realtime/mastercard-dimp/queryEdit*` 規則一致 |
| Request DTO 命名與欄位語意一致 | PASS | ACS | 共用 `GrafanaBaseReq`，欄位一致 |
| 回傳型別策略一致（單值/時序/列表） | PASS | ACS | 5 支 API 皆回傳時序列表 |
| 錯誤回應與例外處理模式一致 | PASS | ACS | 共用 Spring 例外處理 + aspect 驗證 |
| 變數 API 與報表 API 的職責分界清楚 | PASS | ACS | DIMP 屬報表/告警查詢 API |

## C. 時間與查詢上限

| 檢查項 | 狀態 | 適用系統 | 備註 |
|--------|------|----------|------|
| 時間格式皆為 epoch ms（UTC） | PASS | ACS | ES 查詢使用 UTC |
| 區間語意一致（`[from_time, to_time]`） | PASS | ACS | gte/lte 查詢 |
| 預設 interval 規則一致（未傳時） | N/A | ACS | DIMP API 內固定 1h histogram，不吃 interval 參數 |
| 聚合 API 限制輸出點數，不截斷原始交易 | PASS | ACS | 屬聚合計算，未截原始交易 |
| 明細 API 有 row cap 且有截斷標記 | N/A | ACS | DIMP 5 支皆非明細列表 |
| 特例時間邏輯有文件明示 | PASS | ACS | 1h histogram 與門檻規則已文件化 |

## D. 權限與驗證流程

| 檢查項 | 狀態 | 適用系統 | 備註 |
|--------|------|----------|------|
| Alert API 可由 `X-API-Key` 呼叫（無人登入） | PASS | ACS | 由 aspect 優先驗 API key |
| Dashboard API 使用 `hitrustSsoToken` + RBAC | PASS | ACS | 無 API key 時回 token 流程 |
| API Key 與 Token 流程不混用 | PASS | ACS | 優先順序一致 |
| 識別欄位差異已文件化 | PASS | ACS | DIMP 使用 `issuerOid` / `X-BANK-ID` |
| 權限失敗回應行為一致（訊息/碼） | PASS | ACS | 後續可補整體自動化測試 |

## E. 前端整合流程（Vue -> backend -> iframe -> Grafana）

| 檢查項 | 狀態 | 適用系統 | 備註 |
|--------|------|----------|------|
| 路由頁能正確取得 backend 動態 URL | PASS | ACS FE | 現行流程符合架構 |
| backend 回傳 payload 欄位完整（token/org/time/vars） | PASS | ACS FE | 建議補 schema 文件 |
| iframe 載入後 Grafana query 參數與預期一致 | PASS | ACS FE | 建議每次改版抽測 |
| 時間參數在前後端與 Grafana 三端一致 | PASS | ACS FE | 需持續回歸 |
| 失敗情境（token/API key/timeout）處理一致 | FAIL | ACS FE | 錯誤提示與重試行為可再標準化 |

## F. 功能差異管理（重要）

| 檢查項 | 狀態 | 備註 |
|--------|------|------|
| 已確認差異是否為業務需求（不是遺漏） | PASS | DIMP 能力目前聚焦 ACS 屬已決策差異 |
| 差異是否記錄於 `04-api-inventory.md` 差異表 | PASS | 已記錄 3DSS 無同能力 API |
| 差異是否記錄於 `06-decisions-and-open-questions.md` 已決策區 | PASS | `D-005`、`D-008` |
| AI 提示詞是否明確寫「架構一致優先」 | PASS | 已更新 `07-ai-handoff-prompt.md` |

## G. 全域能力覆蓋（本輪總檢）

| 項目 | 狀態 | 備註 |
|------|------|------|
| Alert 能力是否全部納入檢查清單 | FAIL | 仍待補 Insight / DDoS / 系統監控等能力範例 |
| Dashboard 能力是否全部納入檢查清單 | FAIL | 尚未建立 dashboard 能力範例 |
| 每個能力是否有對應文件（inventory / decision / status） | PASS | 核心文件已齊 |
| 未涵蓋能力是否已列入待辦 | PASS | 已在 status 追蹤 |

---

## 結論

- FAIL 項目數：2（`GrafanaUtils` 責任切分、前端錯誤處理標準化）
- 是否可進入改碼：是（可先從 ACS DIMP 小步重構）
- 後續行動：
  1. 將 DIMP 相關共用解析邏輯從 `GrafanaUtils` 拆分
  2. 建立 DIMP 前端錯誤 handling 規範（與 Overview 同模板）
  3. 補第三份能力範例：`cap-alert-insight`
