# API 盤點表（現況 / 目標）

> 用途：重構前建立全景視圖，避免各自為政。  
> 資料來源：程式碼掃描（Controller / Aspect / Req）與現行 `docs/grafana/*` 文件。

## 欄位說明

| 欄位 | 說明 |
|------|------|
| 系統 | `ACS` / `3DSS` |
| 類型 | `dashboard` / `panel` / `alert` / `config` |
| HTTP | `GET` / `POST` |
| 路徑 | API 路徑（相對路徑） |
| Controller | `Class#method`（待補） |
| 用途 | 對應哪個報表或告警 |
| 查詢上限（現況） | 現行限制（若未知填 `TBD`） |
| 查詢上限（目標） | 重構後規格 |
| 時間參數 | from/to、interval、時區語意 |
| 權限檢查 | Aspect、RBAC、API Key |
| 共用候選 | 可抽共用邏輯的名稱 |
| 狀態 | `未盤點` / `已盤點` / `待確認` / `已對齊` |

## 初始盤點（先列高價值）

| 系統 | 類型 | HTTP | 路徑 | Controller | 用途 | 查詢上限（現況） | 查詢上限（目標） | 時間參數 | 權限檢查 | 狀態 |
|------|------|------|------|------------|------|------------------|------------------|----------|----------|------|
| ACS | alert | POST | `/realtime/overview-adv/queryAlertTransactionSuccessRateTimeSeries` | `GrafanaRealTimeOverviewAdvApiController#queryAlertTransactionSuccessRateTimeSeries` | 成功率告警 | 無硬性筆數上限；`dateHistogram` 依 `(to-from)/interval` 產生（告警檔常用 `from=3600s`,`interval=1h`） | 不截斷原始交易；限制輸出 bucket（建議 `<=2000`），超出時自動放大 interval | `from_time`、`to_time`、`interval`（預設 `1h`） | `@GrafanaAcessRightCheck(REPORT_REALTIME_OVERVIEW_ADV)`；`X-API-Key` 優先，否則 `hitrustSsoToken` | 已盤點 |
| ACS | alert | POST | `/realtime/overview-adv/queryAlertTotalTransactions` | `GrafanaRealTimeOverviewAdvApiController#queryAlertTotalTransactions` | 交易量告警 | 回傳單一統計結果（`Map`） | 維持單值 | `from_time`、`to_time` | 同上（`REPORT_REALTIME_OVERVIEW_ADV`） | 已盤點 |
| ACS | alert | POST | `/realtime/overview-adv/queryHistoricalAverageTransactionCount` | `GrafanaRealTimeOverviewAdvApiController#queryHistoricalAverageTransactionCount` | 歷史平均比較 | 回傳單一數值（`value`） | 維持單值 | `currentTime`（或 fallback `to_time`）、`timeWindow` | 同上（`REPORT_REALTIME_OVERVIEW_ADV`） | 已盤點 |
| ACS | alert | POST | `/realtime/overview-adv/queryAlertErrorTypeCount` | `GrafanaRealTimeOverviewAdvApiController#queryAlertErrorTypeCount` | 錯誤類型告警 | 回傳單一數值（`value`）；內部 terms 聚合 `size=100`（`stateMachineReason`） | 維持單值；若 categories 增加，需明定聚合上限 | `from_time`、`to_time`、`errorCategory` | 同上（`REPORT_REALTIME_OVERVIEW_ADV`） | 已盤點 |
| ACS | alert | POST | `/realtime/ddos-attack-stats/queryAlertDDoSAttackStatsExceeded` | `GrafanaDDoSAttackStatsApiController#queryAlertDDoSAttackStatsExceeded` | DDoS 告警 | Controller 固定最近 `2m`；ES 查詢 `size=100` | 保持 `2m`（告警語意固定），ES `size` 改為設定值（建議 `<=100`） | API 內忽略傳入時間，使用 `now-2m ~ now` | `@GrafanaAcessRightCheck(REPORT_REALTIME_OVERVIEW_ADV)`；`X-API-Key` 優先，否則 `hitrustSsoToken` | 已盤點 |
| ACS | alert | POST | `/realtime/transactioninsight/queryMerchantBlacklistAlert` | `GrafanaRealTimeTransactionInsightApiController#queryMerchantBlacklistAlert` | 商戶黑名單告警 | terms 聚合 `size=10000`（商戶聚合） | 改為可設定上限（建議先 `3000`），並檢核是否可改為精準計數策略 | `from_time`、`to_time` | `@GrafanaAcessRightCheck(REPORT_REALTIME_TRANSACTION_INSIGHT)`；`X-API-Key` 優先，否則 `hitrustSsoToken` | 已盤點 |
| ACS | alert | POST | `/realtime/transactioninsight/queryMccBlacklistAlert` | `GrafanaRealTimeTransactionInsightApiController#queryMccBlacklistAlert` | MCC 黑名單告警 | 回傳單一統計結果（`transactionCount`） | 維持單值 | `from_time`、`to_time` | 同上（`REPORT_REALTIME_TRANSACTION_INSIGHT`） | 已盤點 |
| ACS | alert | POST | `/realtime/transactioninsight/queryCountryWatchlistAlert` | `GrafanaRealTimeTransactionInsightApiController#queryCountryWatchlistAlert` | 國家觀察名單告警 | 國家聚合上限為 watchlist 大小（`size=watchlist.size()`） | 維持 watchlist 尺寸上限，並明定上限值 | `from_time`、`to_time` | 同上（`REPORT_REALTIME_TRANSACTION_INSIGHT`） | 已盤點 |
| ACS | alert | POST | `/realtime/transactioninsight/queryHighAmountAlert` | `GrafanaRealTimeTransactionInsightApiController#queryHighAmountAlert` | 高金額告警 | ES 搜尋 `size=100`，但 API 僅回傳單筆 `count` 摘要 | 改為直接 count 聚合，避免取明細再計數 | `from_time`、`to_time` | 同上（`REPORT_REALTIME_TRANSACTION_INSIGHT`） | 已盤點 |

## ACS DIMP 告警 API（新增）

| 系統 | 類型 | HTTP | 路徑 | Controller | 用途 | 查詢上限（現況） | 查詢上限（目標） | 時間參數 | 權限檢查 | 狀態 |
|------|------|------|------|------------|------|------------------|------------------|----------|----------|------|
| ACS | alert | POST | `/realtime/mastercard-dimp/queryEdit1AuthRate` | `GrafanaRealTimeMastercardDimpApiController#queryEdit1AuthRate` | DIMP Edit1 認證率 | `dateHistogram fixedInterval=1h`，無硬性桶數上限 | 不截斷原始交易；限制輸出 bucket（建議 `<=2000`） | `from_time`、`to_time` | `@GrafanaAcessRightCheck(REPORT_REALTIME_MASTERCARD_DIMP)`；`X-API-Key` 優先，否則 `hitrustSsoToken` | 已盤點 |
| ACS | alert | POST | `/realtime/mastercard-dimp/queryEdit3ErrorRate` | `GrafanaRealTimeMastercardDimpApiController#queryEdit3ErrorRate` | DIMP Edit3 錯誤率 | 同上（1h bucket，無硬上限） | 同上（`<=2000` buckets） | `from_time`、`to_time` | 同上 | 已盤點 |
| ACS | alert | POST | `/realtime/mastercard-dimp/queryEdit73riFailureRate` | `GrafanaRealTimeMastercardDimpApiController#queryEdit73riFailureRate` | DIMP Edit7 3RI 失敗率 | 同上（1h bucket，無硬上限） | 同上（`<=2000` buckets） | `from_time`、`to_time` | 同上 | 已盤點 |
| ACS | alert | POST | `/realtime/mastercard-dimp/queryEdit8AbandonmentRate` | `GrafanaRealTimeMastercardDimpApiController#queryEdit8AbandonmentRate` | DIMP Edit8 放棄率 | 同上（1h bucket，無硬上限） | 同上（`<=2000` buckets） | `from_time`、`to_time` | 同上 | 已盤點 |
| ACS | alert | POST | `/realtime/mastercard-dimp/queryEdit11ScaExemptionRate` | `GrafanaRealTimeMastercardDimpApiController#queryEdit11ScaExemptionRate` | DIMP Edit11 SCA 豁免率 | 同上（1h bucket，無硬上限） | 同上（`<=2000` buckets） | `from_time`、`to_time` | 同上 | 已盤點 |

## 3DSS 對應 API（與 ACS 首批 9 支對照）

| ACS API | 3DSS 對應 API | 3DSS Controller | 差異摘要 |
|--------|---------------|-----------------|----------|
| `/realtime/overview-adv/queryAlertTransactionSuccessRateTimeSeries` | `/3dss/overview/queryAlertTransactionSuccessRateTimeSeries` | `GrafanaRealTimeOverviewAdvApiController#queryAlertTransactionSuccessRateTimeSeries` | ACS 用 `issuerOid` + `X-BANK-ID`；3DSS 用 `requestorOid` + `X-REQUESTOR-ID`；AccessId 分別為 `REPORT_REALTIME_OVERVIEW_ADV` vs `REPORT_REALTIME_OVERVIEW` |
| `/realtime/overview-adv/queryAlertTotalTransactions` | `/3dss/overview/queryAlertTotalTransactions` | `GrafanaRealTimeOverviewAdvApiController#queryAlertTotalTransactions` | 同上 |
| `/realtime/overview-adv/queryHistoricalAverageTransactionCount` | `/3dss/overview/queryHistoricalAverageTransactionCount` | `GrafanaRealTimeOverviewAdvApiController#queryHistoricalAverageTransactionCount` | 同上 |
| `/realtime/overview-adv/queryAlertErrorTypeCount` | `/3dss/overview/queryAlertErrorTypeCount` | `GrafanaRealTimeOverviewAdvApiController#queryAlertErrorTypeCount` | 同上 |
| `/realtime/ddos-attack-stats/queryAlertDDoSAttackStatsExceeded` | 無 | 無 | 已確認為系統差異（現階段不新增） |
| `/realtime/transactioninsight/queryMerchantBlacklistAlert` | 無直接同名；僅有 `/3dss/insight/*` 報表 API | `GrafanaRealTimeInsightAdvApiController` | 已確認為系統差異（現階段不新增） |
| `/realtime/transactioninsight/queryMccBlacklistAlert` | 無 | 無 | 已確認為系統差異（現階段不新增） |
| `/realtime/transactioninsight/queryCountryWatchlistAlert` | 無 | 無 | 已確認為系統差異（現階段不新增） |
| `/realtime/transactioninsight/queryHighAmountAlert` | 無 | 無 | 已確認為系統差異（現階段不新增） |

## 下一步

1. 將 `bucket <= 2000`、`merchant terms <= 3000` 等目標值轉為可配置參數（非硬編碼）。
2. 針對 `queryHighAmountAlert` 設計 count-only 版本，避免為了計數拉明細。
3. 依 `03-engineering-standards.md` 新增端到端時間參數對齊檢查腳本/清單。

## 不一致清單（本輪盤點）

| 編號 | 類型 | 現況 | 風險 |
|------|------|------|------|
| I-01 | 上限 | 多數 API 無硬性結果上限（依時間窗與 interval 自然限制） | 大時間窗時可能輸出過大、效能不穩定 |
| I-02 | 上限 | Transaction Insight 告警存在 `terms size=10000` 與 `search size=100` 的不一致策略 | 同類 API 效能表現差異大 |
| I-03 | 時間 | DDoS Alert API 在 Controller 強制 `now-2m ~ now`，忽略請求時間 | 呼叫端難以預期，與其他 API 行為不一致 |
| I-04 | 權限/識別 | ACS 使用 `X-BANK-ID` + `issuerOid`，3DSS 使用 `X-REQUESTOR-ID` + `requestorOid` | 跨系統共用邏輯抽取時容易混淆 |
| I-05 | 產品能力 | ACS 有 DDoS 與黑名單告警 API，3DSS 目前無對應同名告警 API | 已決議為正常差異，重點改為文件標示清楚 |
