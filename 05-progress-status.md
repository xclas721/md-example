# 目前進度（文件先行階段）

## 當前階段

`文件整備（可進入盤點）`

## 已完成

- [x] 建立統一入口：`docs/grafana-architecture-context.md`
- [x] 建立文件中心：`docs/grafana/README.md`
- [x] 建立重構範圍：`docs/grafana/01-scope-and-goals.md`
- [x] 建立架構地圖：`docs/grafana/02-architecture-map.md`
- [x] 建立工程規範：`docs/grafana/03-engineering-standards.md`
- [x] 建立 API 盤點主表：`docs/grafana/04-api-inventory.md`
- [x] 建立決策與待確認清單：`docs/grafana/06-decisions-and-open-questions.md`
- [x] 建立 AI 提示詞模板：`docs/grafana/07-ai-handoff-prompt.md`
- [x] 建立架構一致性檢查表：`docs/grafana/08-architecture-parity-checklist.md`
- [x] 建立檢查表填寫範例：`docs/grafana/09-parity-example-overview-alert.md`
- [x] 建立檢查表填寫範例：`docs/grafana/10-parity-example-dimp-alert.md`
- [x] 建立分流設計稿：`docs/grafana/11-flow-separation-design.md`
- [x] 建立 B 階段進場門檻：`docs/grafana/12-b-phase-readiness-check.md`
- [x] ACS 首批 9 支 alert API：補齊上限現況/目標草案、時間參數、權限檢查點
- [x] ACS DIMP 5 支 alert API：補齊路徑、Controller、上限草案與權限檢查點
- [x] 3DSS 對應 API 與 ACS 差異表（首版）
- [x] 不一致清單（上限/時間/權限/產品能力）首版
- [x] 依使用者決策完成 Q-001~Q-004、Q-101、Q-102 定稿（文件層）
- [x] 補齊 dashboard 能力盤點（能力/路徑前綴層級）
- [x] 新增 Alert/Dashboard 強制分流決策（禁止模式自動判斷工具）

## 進行中

- [x] 補齊首批 ACS alert API（9 支）的 `Controller#method`、時間參數、權限檢查點
- [x] 補齊上述 9 支 API 的查詢上限（現況/目標草案）
- [x] 統一查詢上限草案（先 ACS，再 3DSS）
- [x] 時間語意與權限流程差異表（文件版）
- [ ] 將目標上限改成可設定參數並落地到程式
- [ ] 建立端到端對齊驗證腳本（Vue route -> backend URL -> iframe query）
- [ ] 移除「自動判斷 alert/token」混流工具，改為分流 resolver/service
- [x] 依 `12-b-phase-readiness-check.md` 完成第一次 Go/No-Go 評估（文件版，Go 條件式）

## 下一個里程碑

1. 完成 `04-api-inventory.md` 第一輪盤點（高優先 API 至少 80%）。
2. 完成 `06-decisions-and-open-questions.md` 中 P0 議題決策。
3. 才開始第一波程式重構（先共用層，再 API 分拆）。

## 風險

- 舊文件與新規範可能暫時不一致。
- 3DSS 告警規則目前資料較少，需額外盤點確認。
