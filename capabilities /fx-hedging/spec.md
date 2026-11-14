<div style="border-top: 6px solid #024731; border-bottom: 1px solid #B2B2B2; padding: 12px 0; margin-bottom: 24px; font-family: 'Open Sans', Helvetica, Arial, sans-serif;">
  <div style="color: #024731; font-weight: 700; letter-spacing: 0.06em; text-transform: uppercase; font-size: 0.85rem;">University of Hawaiʻi at Mānoa · Shidler College of Business</div>
  <div style="color: #000000; font-weight: 700; font-size: 1.25rem; margin-top: 4px;">FIN 321: International Finance and Securities</div>
  <div style="color: #525252; font-weight: 400; font-size: 0.95rem;">FX Transaction Hedging Project — Technical Specification</div>
</div>

# Haul-Truck Manufacturer — FX Transaction Hedge Model · Technical Specification

> <span style="color:#024731; font-weight:700;">Technical specification</span> for the EUR receivable hedge model (Scenario 35, `haul-trucks`): the named-range contract, calculation flow, and validation checks, precise enough that an AI or a colleague could build the workbook from this document alone.

| Field | Value |
|------|------|
| **Created by** | Landon Momohara |
| **Updated by** | Landon Momohara |
| **Date Created** | 2026-11-06 |
| **Date Updated** | 2026-11-13 |
| **Version** | 0.3 |
| **LLM Used** | Claude Opus 5.5 (Anthropic): drafted v0.1 and reviewed it for gaps; fixes logged in `prompt-log.md` |
| **Role** | Treasury Analyst |
| **Audience** | CFO / Director of Treasury |
| **Companion Workbook** | `capabilities/fx-hedging/model.xlsx` (Stage 3) |

---

## 1. Problem Statement

We sold haul trucks to a Portuguese copper miner and invoiced them EUR 18,000,000, which we get paid in 365 days (about 2027-11-06). We report in dollars, but the invoice is locked in euros, so what we actually receive is `FC_AMT × S_T`. If the euro drops, we lose money: every 1% fall in EURUSD costs us about $201,000, and a 5% fall costs about $1.0 million. The goal is to protect the dollar value of this payment at a cost we can live with. This model compares four choices (no hedge, a forward, a money-market hedge, and a put option) across settlement rates within ±5% of today's spot, and gives us the numbers for the Stage 5 decision memo.

---

## 2. Inputs — the Named-Range Contract

All ten inputs go on the **Inputs** tab as yellow cells, and each one is set up as a named range. **Every value below is indicative and gets replaced with live market data at Stage 4.** The exceptions are `FC_AMT` and `T_DAYS`, which the scenario fixes, and the two premiums, which the scenario gives us and we keep.

| Name | Description | Placeholder | Unit | Placeholder source (2026-11-06) | Stage-4 source |
|---|---|--:|---|---|---|
| `FC_AMT` | Foreign-currency receivable | 18,000,000 | EUR | Scenario 35 (fixed) | Unchanged |
| `S0_in` | Spot rate at inception | 1.1165 | USD per EUR | ECB euro reference rate, 6 Nov 2026 | Live EURUSD close on population date (ECB reference rate) |
| `F0_in` | Forward rate, 365 days | 1.1807 | USD per EUR | Scenario 35 indicative forward (teaching placeholder) | Live 12-month dealer forward, or CIP-implied forward if none; gap noted |
| `R_USD` | USD interest rate | 4.20% | Annual %, ACT/360 | 1-yr T-bill, Fed H.15 (FRED `DTB1YR`), 3 Nov 2026 | Fed H.15 1-yr T-bill on population date |
| `R_FC` | EUR interest rate | 3.283% | Annual %, ACT/360 | 12-month Euribor, latest Nov 2026 fixing | 12-month Euribor (EMMI) on population date |
| `K_PUT` | Put strike | 1.1165 | USD per EUR | At spot, per scenario (`= S0_in`) | Reset to live `S0_in` |
| `K_CALL` | Call strike | 1.1165 | USD per EUR | At spot, per scenario (`= S0_in`) | Reset to live `S0_in` |
| `PREM_PUT` | Put premium per EUR | 0.0245 | USD | Scenario 35 | Unchanged |
| `PREM_CALL` | Call premium per EUR | 0.0448 | USD | Scenario 35 | Unchanged |
| `T_DAYS` | Days to settlement | 365 | Days | Scenario 35 (fixed) | Unchanged |

I typed `K_PUT` and `K_CALL` in as numbers equal to `S0_in` instead of linking them, so either strike can be changed on its own later. Rates are entered as decimals (4.20% = 0.042).

---

## 3. Tab Architecture

| Tab | Purpose |
|---|---|
| **Cover** | Title, scenario (35, `haul-trucks`), author, version, date, and a data-source table (each input, source, access date) |
| **Legend** | Color key (Stage 3 build contract), applied as cell fills on every tab: yellow = input; blue = assumption (hardcoded setting or reference figure); green = formula; gray = output |
| **Inputs** | The ten named inputs from §2, with unit, source and an "indicative — replaced at Stage 4" flag beside each |
| **Forward** | Forward hedge proceeds (§5.1) |
| **MoneyMarket** | Money-market hedge in three visible steps, plus the parity block and the `PARITY_TOL` assumption (§5.2–5.3) |
| **Options** | Premiums today and at settlement, put floor, breakevens, and put/call payoffs evaluated at `S_T_EVAL` (§5.4) |
| **Sensitivity** | 11-row grid of USD proceeds and hedge profit by strategy, winner labels, and the comparison chart (§6) |
| **Checks** | Live validation checks V1–V7 from §7: computed value, spec expected value, difference, PASS/FAIL/FLAG status |
| **Notes** | Assumptions from §4, open issues, and a change log |

---

## 4. Assumptions & Constraints

- **Quote convention:** USD per EUR. A higher number means a stronger euro.
- **Rate basis:** Both rates are simple annual rates on ACT/360. Every growth factor is `1 + R × T_DAYS/360`. Since `T_DAYS = 365`, that's 365/360 of a year, a bit more than one. I use the T-bill rate as published (discount basis) and ignore the small gap to its money-market yield (about 0.1 pp).
- **Parity expectation:** The money-market hedge should equal `FC_AMT × F_IMPLIED`, the forward that covered interest parity predicts. It only matches the forward hedge when `F0_in` is a real market forward. The 1.1807 placeholder is about 4.8% above `F_IMPLIED`, so I expect a gap until Stage 4. That gap is a data issue, not a formula mistake (§7).
- **Premium treatment:** We pay the premiums in USD today, quoted per 1 EUR with no multiplier. Because we pay them a year before we get paid, I grow them at `R_USD` to settlement so every outcome is compared in the same year-end dollars.
- **Left out:** Bid-ask spreads, fees, taxes, hedge accounting, and counterparty risk. I assume the customer pays in full and on time, and that we hedge the full amount once, today.
- **Options:** The premiums come from the scenario; the model doesn't price them. Options can only be exercised at settlement (European).

---

## 5. Calculation Flow

Everything below uses named ranges, never cell addresses. Each step gets its own cell with the name shown.

### 5.1 Forward hedge
- `USD_FWD` = `FC_AMT × F0_in`. The proceeds are locked in, so this number is the same in every row of the sensitivity grid.

### 5.2 Money-market hedge (three steps)
1. **Borrow EUR today.** `BORROW_FC` = `FC_AMT / (1 + R_FC × T_DAYS/360)`. I borrow less than the full EUR 18M so that the loan plus interest equals exactly what the customer pays us, and their payment closes out the loan.
2. **Convert at spot.** `USD_NOW` = `BORROW_FC × S0_in`.
3. **Invest in USD.** `USD_MM` = `USD_NOW × (1 + R_USD × T_DAYS/360)`. This is the dollar amount we lock in for settlement.

### 5.3 Parity block (on MoneyMarket)
- `F_IMPLIED` = `S0_in × (1 + R_USD × T_DAYS/360) / (1 + R_FC × T_DAYS/360)`
- `MM_TIE` = `USD_MM − FC_AMT × F_IMPLIED`; must be 0 to rounding.
- `PARITY_GAP` = `F0_in / F_IMPLIED − 1`; how far the quoted forward is above (+) or below (−) parity, in percent.
- `PARITY_TOL` = 0.5% (blue assumption).
- `PARITY_FLAG` = `"OK"` if `ABS(PARITY_GAP)` ≤ `PARITY_TOL`, otherwise `"CHECK F0_in"`.

### 5.4 Option hedges
- `FV_PREM_PUT` = `PREM_PUT × FC_AMT × (1 + R_USD × T_DAYS/360)` (a cost, shown as a positive number and subtracted).
- `FV_PREM_CALL` = `PREM_CALL × FC_AMT × (1 + R_USD × T_DAYS/360)`.
- **Put floor (the hedge).** For settlement spot `S_T`: `USD_PUT(S_T)` = `FC_AMT × MAX(S_T, K_PUT) − FV_PREM_PUT`. If the euro ends below `K_PUT`, the put pays `K_PUT − S_T` per euro, so we never get less than `USD_FLOOR_PUT` = `FC_AMT × K_PUT − FV_PREM_PUT`. If it ends above `K_PUT`, we let the put expire and sell our euros at `S_T`.
- **Call participation (informational).** `CALL_PAYOFF(S_T)` = `FC_AMT × MAX(S_T − K_CALL, 0) − FV_PREM_CALL`. A call on EUR only pays when the euro goes up. We're already receiving euros, so a call just adds more upside and does nothing if the euro falls. It isn't a hedge for us. I show it for completeness and **leave it out** of the winner labels.
- **Payoff at a chosen `S_T`.** `S_T_EVAL` = `K_PUT` by default (the kink), with `USD_PUT_EVAL` and `CALL_PAYOFF_EVAL` computed from it, so the Options tab shows both payoffs as functions of `S_T`. Premiums today are also shown in USD: `PREM_PUT_USD0` = `PREM_PUT × FC_AMT`, `PREM_CALL_USD0` = `PREM_CALL × FC_AMT`.
- **Breakevens.** `BE_PUT_FWD` = `F0_in + FV_PREM_PUT / FC_AMT`; `BE_PUT_MM` = `F_IMPLIED + FV_PREM_PUT / FC_AMT`. If the euro ends above these rates, the put does better than that locked hedge.

---

## 6. Sensitivity Plan

- **Grid:** `S_T` = `S0_in × (1 + k%)` for k = −5 to +5 in 1% steps, giving 11 rows with the base case (k = 0) in the middle. The step size is a named assumption `STEP_FRAC` = 1% and the half-width `N_STEPS` = 5, both blue cells on the Sensitivity tab. Each row's k is a formula, `ROW() − ROW(SENS_HDR) − 1 − N_STEPS`, so no row is hand-typed.
- **Columns per row:** `S_T`; `USD_NO_HEDGE` = `FC_AMT × S_T`; `USD_FWD`; `USD_MM`; `USD_PUT(S_T)`; `CALL_PAYOFF(S_T)`; hedge profit `USD_k − USD_NO_HEDGE` for forward, MM and put; `WINNER` (highest of no hedge, forward, MM, put); `BEST_HEDGE` (highest of forward, MM, put). Profit and winner formulas reference named ranges (`USD_FWD`, `USD_MM`, `FC_AMT`, `STRAT_LABELS`, `HEDGE_LABELS`), not just cell addresses.
- **Chart:** One line chart, `S_T` on the x-axis and USD proceeds on the y-axis, with four series: no hedge, forward, MM, put. Forward and MM are flat lines, no hedge is a straight line going up, and the put stays flat until `K_PUT` and then rises alongside no hedge.
- **What the CFO should see:** how much we lose unhedged if the euro falls, what certainty costs (the gap between the flat lines and no hedge at today's spot), and how far the euro has to rise before the put's upside pays back its premium.

---

## 7. Validation Rules (Check Figures)

These become the Stage 3 audit checklist and are computed live on the **Checks** tab. The workbook passes only if every check holds; a failing or flagged check is reported in the audit note, never forced to pass.

| # | Check | Pass condition | Expected at placeholders |
|---|---|---|---|
| V1 | MM ties to CIP | `ABS(MM_TIE)` < USD 1 | `USD_MM` = 20,277,830 |
| V2 | Forward vs. MM | `PARITY_GAP` computed and flagged; must be within ±0.5% after Stage 4 | +4.81%, `"CHECK F0_in"` (expected with placeholder) |
| V3 | MM loan repaid | `BORROW_FC × (1 + R_FC × T_DAYS/360)` = `FC_AMT` | 17,420,153 × 1.033286 = 18,000,000 |
| V4 | Put kink | `USD_PUT` at `S_T = K_PUT` = `FC_AMT × K_PUT − FV_PREM_PUT` | 19,637,221 |
| V5 | Put continuity | Put proceeds rise by `FC_AMT × S0_in × 1%` per step above `K_PUT` and are constant below it | +200,970 per step |
| V6 | Grid symmetry | Middle row `S_T` = `S0_in`; ends = 0.95× and 1.05× `S0_in` | 1.060675 to 1.172325 |
| V7 | No error cells | Zero `#DIV/0!`, `#NAME?`, `#REF!`, `#VALUE!` in any tab | — |
| V8 | Every output is a formula | No hardcoded number in a gray cell | — |
| V9 | Names, not addresses | Every formula references the §2 names or §5 derived names | — |

---

## 8. Outputs (gray cells)

| Output | Tab | Base value (placeholders) |
|---|---|--:|
| `USD_FWD` | Forward | 21,252,600 |
| `BORROW_FC`, `USD_NOW`, `USD_MM` | MoneyMarket | 17,420,153 EUR · 19,449,601 · 20,277,830 |
| `F_IMPLIED`, `PARITY_GAP`, `PARITY_FLAG` | MoneyMarket | 1.126546 · +4.81% · CHECK F0_in |
| `FV_PREM_PUT`, `FV_PREM_CALL` | Options | 459,779 · 840,739 |
| `USD_FLOOR_PUT` | Options | 19,637,221 |
| `BE_PUT_FWD`, `BE_PUT_MM` | Options | 1.2062 · 1.1521 |
| `USD_BASE_k` (no hedge, forward, MM, put at `S_T = S0_in`) | Sensitivity | 20,097,000 · 21,252,600 · 20,277,830 · 19,637,221 |
| `SENS_TABLE` (§6 grid) and `SENS_CHART` | Sensitivity | — |

All dollar values are USD at settlement unless marked EUR.

---

## Appendix A — Change Log

| Version | Date | Author | Change |
|---|---|---|---|
| 0.1 | 2026-11-06 | Landon Momohara (AI-drafted) | First draft from the Stage 1 brief and Scenario 35 |
| 0.2 | 2026-11-06 | Landon Momohara | Review fixes: parity rule split into V1 (formula tie) and V2 (data gap); premiums carried forward at `R_USD`; call defined as informational; inputs refreshed to 6 Nov 2026 |
| 0.3 | 2026-11-13 | Landon Momohara (AI-assisted) | Stage 3 build fixes: legend aligned to the Stage 3 colour contract; added Checks tab; named `PARITY_TOL`, `N_STEPS`, `S_T_EVAL`; formula-driven grid index; named-range rule extended to profit and winner columns |
