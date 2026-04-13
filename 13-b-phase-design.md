# B 階段設計文件（定稿）

> 來源：依 `00/01/03/04/06/11/12` 收斂後的 B 階段輸出。  
> 約束：不改碼，只定義可執行設計。

## 1. 目標與範圍

- 目標：建立 Alert / Dashboard 分流架構，消除混流工具依賴。
- 目標：保持 ACS / 3DSS 架構同型，但不強迫功能同型。
- 範圍：Grafana 相關 backend flow、前端 iframe 契約、時間與上限策略。

## 2. 現況問題摘要（按能力域）

- Overview/Alert：已盤點完整，但上限策略需程式參數化。
- Dashboard：已到能力域層級，尚需 API 細項盤點。
- Flow 邊界：現有 `get*ForGrafana()` 類方法同時處理 alert/token 模式。
- 權限：邊界已定義，程式結構尚未完全反映。

## 3. 目標架構（Alert / Dashboard 分流）

- Alert path：僅 `X-API-Key` + 組織識別（背景流程）。
- Dashboard path：僅 `hitrustSsoToken` + RBAC（使用者流程）。
- 共用只允許純函式，不允許流程判斷共享。

## 4. 模組與責任切分（Controller / Service / Resolver / Utils）

- Controller：只轉呼叫，不做流程判斷。
- Service：能力邏輯（Overview/Insight/...），不做身份模式判斷。
- Resolver：`AlertRequestContextResolver` / `DashboardRequestContextResolver`。
- Utils：僅保留純函式（時間、正規化、URL 參數組裝）。

## 5. API 契約調整策略（含相容性）

- 對外路徑先維持，先做內部分流。
- 先相容後收斂：Phase1 並行、Phase2 移除混流入口、Phase3 契約硬化。
- 以能力域小步切換，保留回退開關。

## 6. 時間與上限策略落地

- 時間統一：epoch ms + UTC，區間 `[from_time,to_time]`。
- 聚合 API 限輸出 bucket；明細 API 才做 row cap 並回傳截斷訊號。
- 將目前文件目標值轉為可配置參數。

## 7. 權限與驗證流程落地

- Alert API：API key path only。
- Dashboard API：token + RBAC path only。
- 依賴規則約束：Alert controller 不可依賴 Dashboard resolver，反之亦然。

## 8. 遷移步驟（分批、可回滾）

1. 建立 resolver 與 context model（不切流量）。
2. 試點 Overview（同時涵蓋 alert+dashboard）。
3. 能力域擴展（DIMP/Insight/DDoS/...）。
4. 移除混流工具入口。
5. 完成回歸驗證與收斂。

## 9. 測試與驗證計畫

- 單元：resolver 輸入輸出與錯誤分支。
- 整合：controller+resolver+service 全路徑。
- E2E：Vue -> backend URL -> iframe query 參數一致性。

## 10. 風險、觀測指標、回滾條件

- 風險：識別欄位解析錯誤、告警中斷、iframe 參數錯配。
- 指標：4xx/5xx、告警觸發成功率、查詢延遲、空資料率。
- 回滾：按能力域回切，不做全域一次回退。

## 11. 里程碑與交付物

- M1：B 設計審核通過（本文件）。
- M2：Overview 分流試點完成。
- M3：能力域擴展完成。
- M4：混流工具入口移除，驗證全綠。
