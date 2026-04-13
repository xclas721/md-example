# C 階段第一批改碼清單（可執行）

> 目的：把 B 設計轉成第一批小步改碼任務。  
> 原則：先試點 Overview，最小風險、可回滾。

## 0. 批次目標

- 移除 Overview 能力中的混流入口依賴（不一次移除全域）。
- 建立 Alert / Dashboard 雙 resolver 骨架並在試點使用。
- 確保既有 dashboard 顯示與 alert 觸發無回歸。

## 1. 涉及範圍（第一批）

### ACS
- `.../grafana/utils/GrafanaUtils.java`（只做減責，不一次刪除）
- `.../grafana/controller/GrafanaRealTimeOverviewAdvApiController.java`
- 新增：`.../grafana/resolver/AlertRequestContextResolver.java`
- 新增：`.../grafana/resolver/DashboardRequestContextResolver.java`

### 3DSS
- `.../grafana/utils/GrafanaUtils.java`（只做減責，不一次刪除）
- `.../grafana/controller/GrafanaRealTimeOverviewAdvApiController.java`
- 新增：`.../grafana/resolver/AlertRequestContextResolver3dss.java`
- 新增：`.../grafana/resolver/DashboardRequestContextResolver3dss.java`

## 2. 任務拆解

1. 建立 resolver 與 context model（不改 controller）。
2. 為 Overview alert 4 支 API 改用 alert resolver。
3. 為 Overview dashboard 相關 API 改用 dashboard resolver。
4. 將 controller 中模式判斷移除。
5. 保留舊工具方法但標記 deprecated（供下一批移除）。

## 3. 驗證清單（必做）

- Alert path：`X-API-Key` 有效/無效測試。
- Dashboard path：token 有效/無效 + RBAC 測試。
- 時間參數一致性：from/to/interval 在 backend 與 Grafana query 一致。
- E2E smoke：Overview 頁面可正常載入、告警可正常查詢。

## 4. 回滾策略

- 每個能力域使用 feature flag（或 bean 切換）控制新 resolver。
- 若試點異常，僅回切 Overview，其他能力域不受影響。

## 5. 完成定義（DoD）

- Overview 相關 controller 不再呼叫混流入口方法。
- 新增 resolver 覆蓋 alert/dashboard 兩種流程。
- 測試與 smoke 驗證通過。
- `04-api-inventory.md`、`05-progress-status.md` 完成同步更新。
