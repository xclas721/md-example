# 架構地圖（跨專案）

## 工作區根目錄

`c:\HiTRUST\3workspace`

## 專案角色

| 路徑 | 角色 |
|------|------|
| `threeds-server-v3/` | 3DS Server 後端（含 admin/core/data/metrics） |
| `threeds-acs-v3/` | ACS 後端（含 acs-admin-backend） |
| `threedsserver-admin-frontend/` | 3DSS 管理前端 |
| `acs-admin-frontend/` | ACS 管理前端 |
| `Grafana-Test-data/` | 測試資料與情境模擬 |
| `dev-es-grafana/` | 本機 Grafana / ES 環境 |

## 後端地圖

### 3DSS Admin Backend

主要路徑：

`threeds-server-v3/threedsserver-admin-backend/src/main/java/com/hitrust/emv/threeds/threedsserver/admin/insightedge/grafana/`

重點類別群：

- `controller/GrafanaRealTime*.java`
- `req/GrafanaBaseReq.java`
- `service/GrafanaApiServiceImpl.java`
- `service/GrafanaUserService.java`
- `utils/GrafanaUtils.java`
- `aspect/GrafanaAccessControlAspect.java`
- `constants/GrafanaApiConst.java`

非 `grafana` 子套件但強相關：

- `.../insightedge/controller/key/GrafanaKeyController.java`
- `.../insightedge/controller/system/GrafanaOrgController.java`

### ACS Admin Backend

主要路徑：

`threeds-acs-v3/acs-admin-backend/src/main/java/com/hitrust/emv/threeds/acs/admin/insightedge/grafana/`

重點類別群：

- `controller/Grafana*.java`
- `req/*.java`（含 `GrafanaBaseReq.java`）
- `service/GrafanaApiServiceImpl.java`
- `service/GrafanaUserService.java`
- `service/GrafanaTemplateVariableService.java`
- `utils/GrafanaUtils.java`
- `aspect/GrafanaAccessControlAspect.java`

## 前端地圖

### 3DSS Frontend

- `threedsserver-admin-frontend/src/utils/grafanaUtil.ts`
- `threedsserver-admin-frontend/src/stores/grafana.ts`
- `threedsserver-admin-frontend/src/views/redirect/GrafanaRedirectView.vue`
- `threedsserver-admin-frontend/src/views/redirect/GrafanaViewerView.vue`
- `threedsserver-admin-frontend/src/plugins/permission.ts`

### ACS Frontend

- `acs-admin-frontend/src/utils/grafanaUtil.ts`
- `acs-admin-frontend/src/stores/grafana.ts`
- `acs-admin-frontend/src/views/redirect/GrafanaRedirectView.vue`
- `acs-admin-frontend/src/views/redirect/GrafanaViewerView.vue`
- `acs-admin-frontend/src/plugins/permission.ts`

## 關鍵重複點（重構優先）

- `GrafanaUtils`：ACS / 3DSS 同名工具，行為需對齊。
- `GrafanaAccessControlAspect`：權限與 API Key 驗證行為需對齊。
- `GrafanaApiServiceImpl`：呼叫 Grafana API 的共用流程可抽象化。
- 前端 `grafanaUtil.ts` / `stores/grafana.ts`：可建立一致 API 客戶端層。

## 對齊原則（重要）

- ACS 與 3DSS 是不同系統、不同客戶目標，功能清單可不同。
- 對齊重點是「架構一致」：
  - 同層級責任一致（Controller / Service / Utils / Aspect）
  - 參數命名與驗證流程一致（在各自領域語意下）
  - 文件、測試與回歸流程一致
- 不做的事：為了表面一致而硬補不存在的業務功能。
