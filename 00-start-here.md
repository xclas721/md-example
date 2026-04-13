# Start Here（最小入口）

> 給「要快速進 B 階段」使用。  
> 原則：先用最小集合，不夠再擴充。

## 最小閱讀集合（正式規格）

1. `docs/grafana/01-scope-and-goals.md`
2. `docs/grafana/03-engineering-standards.md`
3. `docs/grafana/04-api-inventory.md`
4. `docs/grafana/06-decisions-and-open-questions.md`

## B 階段最小指令（建議先用）

```text
@docs/grafana/00-start-here.md
@docs/grafana/01-scope-and-goals.md
@docs/grafana/03-engineering-standards.md
@docs/grafana/04-api-inventory.md
@docs/grafana/06-decisions-and-open-questions.md
@docs/grafana/11-flow-separation-design.md

請提出 B 階段設計方案（不改碼）。
限制：
1) Alert 與 Dashboard 必須分流，不可單一工具自動判斷模式。
2) ACS/3DSS 功能可不同，但架構與維護模式需一致。
3) 不要把 09/10 當正式規格，只能當示例參考。
```

## 何時改用完整集合

- 需要 Go/No-Go 正式評估時：加讀 `12-b-phase-readiness-check.md`
- 需要檢核填表流程時：加讀 `08` + `09` + `10`
