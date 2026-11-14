# FX Hedging

Hedging a foreign-currency transaction exposure: sizing the risk, then comparing a forward, a money-market hedge and an option hedge across a range of settlement spots, in a workbook built from a written specification and repopulated with live market data.

| File | What it is |
|---|---|
| `spec.md` | Technical specification: named-range contract, tab architecture, calculation flow, sensitivity plan, validation rules |
| `model.xlsx` | The workbook built from `spec.md` (Stage 3): ten named inputs, three hedge families, ±5% sensitivity grid and chart, live checks |
| `fx-hedging-named-ranges.xlsx` | Stage 2 working sheet used to check placeholder values |

## Engagements

| Date | Engagement | Exposure | Artifacts |
|---|---|---|---|
| 2026-11-06 | Scenario 35, `haul-trucks`: U.S. haul-truck manufacturer, Portuguese copper miner | EUR 18,000,000 receivable, 365 days | [Brief](../../docs/briefs/2026-10-30-haul-trucks-hedge-brief.md) · [Spec](spec.md) · [Model](model.xlsx) |
