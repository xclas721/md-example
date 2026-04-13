# Grafana 文件中心（重構前置）

本資料夾是 Grafana / InsightEdge 重構前的「唯一文件中心」，目的：

- 降低資訊分散與過時風險
- 讓 AI 與人都能用固定路徑讀到一致脈絡
- 將目標、規範、進度、決策分開維護

## 檔案索引

- `01-scope-and-goals.md`：重構目標、非目標、交付方式
- `02-architecture-map.md`：跨專案程式地圖與責任邊界
- `03-engineering-standards.md`：統一規範（查詢上限、時間、權限、共用邏輯）
- `04-api-inventory.md`：API 盤點主表（現況/目標）
- `05-progress-status.md`：進度狀態與下一步
- `06-decisions-and-open-questions.md`：已決策與待確認議題
- `07-ai-handoff-prompt.md`：給 Cursor AI 的標準起手提示詞
- `08-architecture-parity-checklist.md`：ACS/3DSS 架構一致性檢查表

## 來源文件（已整併）

舊版根目錄規格檔已移除，內容已整併到本資料夾內各分檔。  
若需追溯舊版本，請用版本控制歷史查詢。

## 更新規則

1. 新資訊優先更新這個資料夾，不再新增平行入口文件。
2. 每次改動 API / 權限 / 時間邏輯時，至少更新：
   - `03-engineering-standards.md`
   - `04-api-inventory.md`
   - `05-progress-status.md`
3. 有爭議或待確認內容，寫到 `06-decisions-and-open-questions.md`，避免口頭遺失。
