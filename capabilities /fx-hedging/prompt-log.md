## 2026-11-06 — Stage 2 model specification (haul-trucks)

**Tool:** Claude Opus 5.5 (Anthropic), claude.ai
**Asked for:** A Stage 2 spec (`capabilities/fx-hedging/spec.md`) meeting the Stage 2 checklist, built from `template-spec.md`, my Stage 1 brief and Scenario 35; plus the capability README.
**AI role:** Drafted v0.1 of the spec, then reviewed it against the scenario's numbers to find gaps. I reviewed the fixes before committing.

**Prompts (summary):**
1. Pasted the Stage 2 page and asked for help with the stage.
2. "Create a spreadsheet from your last answer" (named-range input table, used to check placeholder values).
3. Pasted the deliverables checklist: "make sure these are checked off for the deliverables."

**Iteration — gap found and fixed (v0.1 → v0.2):**

- **Gap 1: the parity validation rule would fail by design.**
  - *Before (v0.1, §7):* "MM proceeds must equal forward proceeds within rounding."
  - *Found:* With the scenario's placeholder forward (1.1807) and today's spot and rates, the CIP forward is 1.1265, so forward proceeds ($21,252,600) exceed MM proceeds ($20,277,830) by 4.8%. A correct build would fail the check, and the Stage 3 audit couldn't tell a formula error from a data gap.
  - *After (v0.2, §5.3 and §7):* Split into V1, `USD_MM` must equal `FC_AMT × F_IMPLIED` to under $1 (formula check, always passes), and V2, `PARITY_GAP = F0_in / F_IMPLIED − 1` flagged beyond ±0.5% (data check, expected to flag until Stage 4).
- **Gap 2: premium timing was inconsistent.**
  - *Before:* Put proceeds = `FC_AMT × MAX(S_T, K_PUT) − FC_AMT × PREM_PUT`, which subtracts a today-dollar premium from settlement-date proceeds.
  - *After:* Premiums carried forward at `R_USD` (`FV_PREM_PUT`, `FV_PREM_CALL`), matching `template-spec.md`; stated in §4.
- **Accepted / rejected:** Accepted both. Considered adding a `BASIS` named range from the template but rejected it, because the Stage 2 page fixes the contract at ten names and ACT/360 for both legs; the 360 basis is stated in §4 instead.

**What I checked before committing:** _(edit to what you actually checked)_ Placeholder sources and dates (ECB reference rate 6 Nov 2026; FRED DTB1YR 3 Nov 2026; 12-month Euribor Nov 2026), base-case check figures recomputed, no cell addresses in §5.

## 2026-11-13 — Stage 3 AI-assisted build (haul-trucks)

**Tool:** Claude Opus 5.5 (Anthropic), claude.ai
**Asked for:** Build `capabilities/fx-hedging/model.xlsx` from my committed Stage 2 spec, meeting the seven-item Stage 3 build contract.
**Context given:** The Stage 3 page link, then the GitHub link to my committed spec folder (no copy-paste).
**AI role:** Generated the workbook from the spec with a Python/openpyxl build script, recalculated it in LibreOffice, and ran an automated check (named-range attachment, hardcodes in formula/output cells, formulas without named ranges, error cells, §7 figures vs an independent recompute).

**Prompts:**
1. Pasted the Stage 3 page link (no instruction).
2. "this is what I have from stage 2" + GitHub link to `capabilities /fx-hedging`.

**Spec defects found while building (fixed in spec v0.3, then regenerated):**
- Legend in §3 (yellow fill / blue text / black / green link / gray) contradicted the Stage 3 contract (yellow / blue / green / gray fills). Aligned to the contract.
- No home for live checks: added a **Checks** tab.
- Values the build needed but the spec never named: `PARITY_TOL` (0.5% was buried in a formula), `N_STEPS`, `S_T_EVAL`; grid index made a formula so no row is hand-typed.

**Repo fix:** The capability folder was named `capabilities ` with a trailing space, so `capabilities/fx-hedging/` did not exist. Moved the files and put this log at the repo root.

**Audit fix committed separately:** The first build passed every §7 check but 90 formulas on Sensitivity and Checks used cell addresses only (e.g. `=D6-C6`), breaking the formulas-reference-named-ranges rule. Rewritten against named ranges; values unchanged.

**Kept / changed / rejected:** Kept all §7 figures (independent recompute matched to the dollar). Kept `K_PUT`/`K_CALL` typed rather than linked, per spec §2, even though the Stage 2 named-ranges sheet linked them to `S0_in`.
