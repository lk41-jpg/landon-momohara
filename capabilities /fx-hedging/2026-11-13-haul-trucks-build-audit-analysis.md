# Stage 3 Build Audit — haul-trucks

**Author:** Landon Momohara · **Date:** 2026-11-13
**Workbook:** `capabilities/fx-hedging/model.xlsx` · **Spec:** `capabilities/fx-hedging/spec.md` v0.3

## Check results

| Check | Result | Value |
|---|---|---|
| V1 MM ties to CIP | PASS | `USD_MM` 20,277,830 |
| V2 Forward vs MM | FLAG | +4.81% (placeholder `F0_in`) |
| V3 MM loan repaid | PASS | 18,000,000 EUR |
| V4 Put kink | PASS | 19,637,221 |
| V5 Put continuity | PASS | +200,970 per step |
| V6 Grid symmetry | PASS | 1.060675 to 1.172325 |
| V7 No error cells | PASS | 0 |
| V8/V9 Formulas, names | By inspection | See Finding 1 |

## Finding 1 — Formulas without named ranges

- **Checked:** Scanned every formula for named-range references.
- **Found:** About 90 cells on Sensitivity and Checks used cell addresses only (e.g. `=D6-C6`).
- **Did:** Rewrote against `USD_FWD`, `USD_MM`, `FC_AMT`, `STRAT_LABELS`, `HEDGE_LABELS`; values unchanged. Commit 6440036.

**Why it matters:** Passing surface-level number checks doesn't guarantee a resilient model; relying on fragile cell addresses instead of named ranges hides structural weaknesses that break easily when the sheet changes.

## Finding 2 — Spec legend contradicted the build contract

- **Checked:** Spec §3 legend against Stage 3 contract item 4.
- **Found:** Spec said yellow fill / blue text / black / green link; the contract says yellow / blue / green / gray.
- **Did:** Fixed the spec (v0.3) and regenerated. Commit d191579.

**Why it matters:** Because an AI builds strictly to its instructions, an outdated specification will reliably produce a flawed model unless the spec itself is validated against the latest contract rules first.

## Finding 3 — V2 parity flag (data, not formula)

- **Checked:** `F0_in` against the CIP-implied forward.
- **Found:** 1.1807 vs 1.1265, a +4.81% gap; as a result the forward "wins" every row of the sensitivity grid.
- **Did:** Left the flag failing on purpose; `F0_in` is replaced with a live forward at Stage 4.

**Why it matters:** Accurate formulas still produce misleading advice when fed placeholder inputs, so keeping validation flags visible is essential to prevent decision-makers from acting on fake data.
