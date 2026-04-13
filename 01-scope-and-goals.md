# 重構範圍與目標

## 背景

目前 Grafana 相關程式分散於多個 repo（ACS/3DSS 前後端 + 測試專案），存在同職責多版本實作、命名與行為細差異，導致可讀性與維護成本高。

## 本階段目標

1. 重新盤點 dashboard / panel / alert 使用到的 backend API。
2. 將 API 責任切清楚，降低耦合，避免同邏輯重複實作。
3. 統一查詢上限策略，避免效能與結果失真。
4. 統一時間參數與時間區間語意（from/to、時區、邊界）。
5. 統一權限檢查行為（前端路由、後端切面、API 驗證）。
6. 建立「架構一致、功能可不同」準則：ACS 與 3DSS 不必功能等同，但程式分層、命名、責任邊界與維護方式要一致。

## 非目標（目前）

- 不先大改 Grafana Dashboard JSON 視覺配置。
- 不先做大範圍 UI 改版（除非為了配合 API 契約）。
- 不先優化所有報表演算法，只先處理結構一致性與可讀性。
- 不要求 ACS 與 3DSS 的告警數量或功能項完全相同（兩系統業務目標不同）。

## 交付物（文件先行）

- API 現況盤點表（`04-api-inventory.md`）
- 工程規範（`03-engineering-standards.md`）
- 決策與待確認清單（`06-decisions-and-open-questions.md`）
- 進度與里程碑（`05-progress-status.md`）

## 完成定義（Definition of Ready for coding）

符合以下條件才進入大量改碼：

- 主要 API 已盤點完成（至少 80%）
- 查詢上限與時間語意有明確規格
- 權限規則有單一來源說明
- 至少一輪跨 ACS/3DSS 差異比對完成
