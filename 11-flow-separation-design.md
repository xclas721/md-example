# Alert / Dashboard 分流設計稿（B 階段輸入）

> 文件定位：B 階段設計基準（不含實作碼）。  
> 目標：移除「同一工具自動判斷 alert/token 模式」混流作法，建立可維護的雙流程架構。

## 1. 問題定義

目前在 ACS / 3DSS 皆存在類似模式：

- `GrafanaUtils#getIssuerOidForGrafana(...)`（ACS）
- `GrafanaUtils#getRequestorOidForGrafana(...)`（3DSS）

這類方法同時處理：

1. Dashboard token 流程（使用者上下文）
2. Alert API Key 流程（背景任務）

導致：

- 流程邊界不清（controller 不知道自己在哪個模式）
- 測試難切分（同一方法要覆蓋兩種上下文）
- 變更風險高（改 alert 影響 dashboard，反之亦然）

## 2. 設計原則

依 `D-009` 與 `03-engineering-standards.md`：

- Alert 與 Dashboard 必須是兩條流程。
- 禁止以單一工具方法內 `if apiKey/token` 分支作主流程入口。
- 共用僅限純函式（時間格式、字串正規化），不含流程判斷。

## 3. 目標架構（概念）

### 3.1 ACS

- `AlertRequestContextResolver`
  - 輸入：`X-API-Key`, `X-BANK-ID`
  - 輸出：`AlertContext(issuerOid, hostedFlag, authType=API_KEY)`
- `DashboardRequestContextResolver`
  - 輸入：`hitrustSsoToken`, request params
  - 輸出：`DashboardContext(issuerOid, userId, authType=TOKEN)`

### 3.2 3DSS

- `AlertRequestContextResolver3dss`
  - 輸入：`X-API-Key`, `X-REQUESTOR-ID`
  - 輸出：`AlertContext(requestorOid, hostedFlag, authType=API_KEY)`
- `DashboardRequestContextResolver3dss`
  - 輸入：`hitrustSsoToken`, request params
  - 輸出：`DashboardContext(requestorOid, userId, authType=TOKEN)`

### 3.3 共用純函式（可跨流程/跨系統）

- `OidNormalizer`
- `TimeRangeNormalizer`
- `GrafanaUrlParamBuilder`（僅組參數，不做身份判斷）

## 4. Controller 端使用規則

- Alert controller：
  - 只注入 Alert resolver
  - 不可呼叫 Dashboard resolver
- Dashboard controller：
  - 只注入 Dashboard resolver
  - 不可呼叫 Alert resolver
- 禁止在 controller 做「如果有 API key 就走 alert」分支。

## 5. 遷移步驟（安全拆分）

1. **建立新 resolver 類別**（先不改舊流程）
2. **挑一個能力域試點**（建議 Overview alert + Overview dashboard）
3. **以 feature flag 或小範圍切換**至新 resolver
4. **回歸驗證**（API 回傳、iframe 參數、權限失敗行為）
5. **移除舊混流入口**（`get*ForGrafana()` 類）

## 6. 驗收標準

- 程式中不存在新的混流入口（單一方法同時判斷 alert/token 模式）。
- Alert / Dashboard controller 各自只依賴對應 resolver。
- 既有 dashboard 顯示結果與 alert 觸發行為無回歸。
- `04-api-inventory.md` 的權限與時間欄位不需靠「模式自動判斷」解釋。

## 7. 風險與緩解

| 風險 | 影響 | 緩解 |
|------|------|------|
| 切換 resolver 後識別欄位取值錯誤 | 查詢結果錯誤 | 先以 Overview 做灰度驗證 |
| Alert 流程誤走 token | 告警中斷 | Alert controller 加單元測試與契約測試 |
| Dashboard 流程誤走 API key | 權限漏洞 | 強制依賴規則 + code review checklist |

## 8. B 階段輸出要求

B 階段提案至少要回答：

1. 第一批拆分的 controller 名單
2. 新舊 resolver 對應關係
3. 回歸測試清單與觀測指標
4. 回滾方案（切回舊 resolver 的條件與方法）
