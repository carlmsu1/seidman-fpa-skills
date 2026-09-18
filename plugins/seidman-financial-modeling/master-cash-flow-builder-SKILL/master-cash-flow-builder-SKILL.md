---
name: master-cash-flow-builder
description: "Orchestrates `cash-flow-builder-ar`, `cash-flow-builder-ap`, `payroll-schedule-builder`, and optionally `capex-depreciation-builder` and `debt-amortization-builder`, into one combined monthly cash flow model on a single shared spine. Runs one consolidated interview instead of asking shared facts once per module, resolves the tab-naming collisions that arise when modules that each independently use generic names like `Forecast` and `Source_Data` are combined into one workbook, and builds a `Consolidated_Cash_Flow` roll-forward that pulls each module's actual cash line with the correct sign — some modules' lines already carry the right sign for a cash flow statement, others need an explicit flip, and getting this wrong silently cancels or doubles a real cash movement rather than erroring loudly. Also builds a light `PL_Memo` tab — sales, purchases, payroll expense, depreciation, interest — rolling to an implied EBITDA. Documents plainly that a negative closing cash balance is an expected, honest signal of a funding gap, not an error, since the debt module this hub draws on is term-loan-only with no revolver to plug one. Use whenever the user asks to build a combined or consolidated cash flow model, a full monthly cash flow forecast spanning A/R, A/P and payroll, an integrated model with capex and/or debt, or a company-wide monthly cash flow forecast pulling several schedules together. This skill REQUIRES `financial-modeling-foundation`, `cash-flow-builder-ar`, `cash-flow-builder-ap`, and `payroll-schedule-builder` always, plus `capex-depreciation-builder` and/or `debt-amortization-builder` wherever those are in scope for the engagement — load and follow each one, and ask the user to enable any that are not available."
---

# Master Cash Flow Builder

**Requires `financial-modeling-foundation`, `cash-flow-builder-ar`, `cash-flow-builder-ap`, and `payroll-schedule-builder` in every build; `capex-depreciation-builder` and `debt-amortization-builder` wherever those are in scope.** Load Foundation first, then every module skill that's in scope for this engagement. Each module's own instructions still govern that module's own mechanics in full — this skill is additive on top of all of them, not a replacement for any. Where this skill appears to contradict Foundation or a module skill, Foundation wins, then the module skill for anything specific to its own domain; this skill only governs how the modules combine.

**This is an orchestration layer, not a sixth set of financial mechanics.** It does not re-derive DSO, wage-base caps, depreciation methods, or amortization schedules — every module skill already owns its own math in full. What this skill adds is everything that only exists *because* several modules are landing in one workbook: one interview instead of five, one spine instead of five, resolved tab-name collisions, and a consolidation layer that pulls the modules together correctly.

**Build in this order.**

1. Interview the modeler — scope, then the consolidated facts
2. Build the shared spine
3. Build each in-scope module, with renamed tabs
4. Build `Consolidated_Cash_Flow`
5. Build `PL_Memo`
6. Extend the shared `Checks` tab

---

# 1. Interview the modeler

## Scope, first

Before any of the individual modules' own interviews, establish which modules this engagement needs:

> Before anything else: which of these five does this model need?
> 1. **A/R** — nearly always in scope for a company with receivables.
> 2. **A/P** — nearly always in scope for a company with trade payables.
> 3. **Payroll** — nearly always in scope for a company with employees.
> 4. **Capex & depreciation** — only if fixed asset activity is material to this engagement.
> 5. **Debt amortization** — only if the company carries term debt this forecast needs to service.
>
> A/R, A/P, and payroll together form the operating core of the cash flow; capex and debt are overlays, included only where they matter to this engagement.

Record the answer plainly in the notes block — every later step depends on knowing exactly which modules are in play.

## Consolidated Phase 1 — asked once, not once per module

Every in-scope module's own Phase 1 asks for company name, model start date, horizon, and reporting units. **Ask these once, here, and tell each module skill to inherit them rather than asking again:**

> Before I look at any data: company name, the first forecast month and how many months, and reporting units (whole dollars, thousands, or millions).

Everything else in each module's own Phase 1 — A/R's materiality threshold, A/P's payment policy fork, payroll's pay frequency and anchor pay date, capex's in-service convention, debt's day count convention — is genuinely module-specific and still gets asked, once each, in the course of building that module. This hub only removes the redundant four questions, not the real ones.

---

# 2. Build the shared spine

**Every in-scope module reads this spine rather than building its own.** Build it once, before any module.

- **`Macro_Assumptions`** — company name, units, forecast start date, period count, from the consolidated Phase 1 above.
- **The header** — `Month Number`, `Actual/Forecast`, `Month Beginning` (via `EDATE`, per Foundation), exactly as each individual module skill specifies. This lives once, not once per module.
- **The A/R-to-Cash and A/P-to-Cash footer**, only if A/R and/or A/P are in scope — `Projection Period - Date`, `Transaction Adjustment`, and whichever of the `A/R-to-Cash Collection Period` / `A/P-to-Cash Payment Period` rows the in-scope modules need. Build this once even where both are in scope; each module drives its own row against it, per that module's own instructions.

Payroll, capex, and debt each have their own internal period mechanics (the pay-period ledger, the in-service dating, the amortization schedule) that don't touch the A/R-to-Cash footer at all — they only need the header and `Macro_Assumptions`.

---

# 3. Build each in-scope module — with renamed tabs

Build each in-scope module following its own skill's instructions in full, **except for one mandatory change: rename every tab that module would otherwise name generically**, because combining modules that each independently chose the same generic name produces a real collision — Excel won't allow two sheets both named `Forecast`, and even where a surface tolerated it, a reader couldn't tell which is which.

**Two names collide across modules; the rest don't:**

| Generic name | Used by | Renamed to |
|---|---|---|
| `Forecast` | A/R, A/P, payroll all use this exact name for their primary output tab | `AR_Forecast`, `AP_Forecast`, `Payroll_Forecast` |
| `Source_Data` | Every module's catch-all for remaining extracts | `AR_Source_Data`, `AP_Source_Data`, `Payroll_Source_Data`, `Capex_Source_Data`, `Debt_Source_Data` |

Every other tab a module skill specifies — `AR_Assumptions`, `Customer_Detail`, `AR_Aging`, `AP_Assumptions`, `Vendor_Detail`, `AP_Aging`, `Payroll_Assumptions`, `Roster_Detail`, `Pay_Period_Ledger`, `Capex_Assumptions`, `Fixed_Asset_Schedule`, `Debt_Assumptions`, `Debt_Schedule` — is already uniquely named and needs no change.

**`Cover`, `Macro_Assumptions`, and `Checks` are never rebuilt per module.** Where a module skill's own instructions describe building these, skip that step here — they were already built once in Section 2, and every module's checks land on that same shared `Checks` tab (Section 6), not a tab of their own.

**Suggested tab order**, per Foundation's standard order with the model-specific tabs grouped by module, deliverable tabs first: `Cover`, `Macro_Assumptions`, `Checks`, `Consolidated_Cash_Flow`, `PL_Memo`, then each in-scope module's own tabs together (A/R's block, then A/P's, then payroll's, then capex's, then debt's) — only the blocks for modules actually in scope.

---

# 4. Build `Consolidated_Cash_Flow`

**This is the one genuinely new mechanic this skill adds.** Every module already computes its own cash-relevant figure correctly — the only new work is pulling each one in with the sign a cash flow statement actually needs, which is not the same sign every module uses internally.

## Why the sign isn't uniform across modules

Foundation's sign convention creates a real, documented "one-directional collision" between a balance roll-forward (signed by effect on the balance) and a cash flow statement (signed by effect on cash) — and different modules resolve that collision differently depending on how they present their own output:

- **A module built as a balance roll-forward, where the balance-effect and the cash-effect happen to point the *same* direction**, needs no flip. A/P's payment lines reduce the payables balance (negative) *and* are cash out (negative) — same sign both places. Debt's draws increase the balance (positive) *and* are cash in (positive); its scheduled principal and prepayments reduce the balance (negative) *and* are cash out (negative) — same sign both places, in both directions.
- **A module built as a balance roll-forward, where the balance-effect and the cash-effect point *opposite* directions**, needs exactly one flip — in either direction. A/R's receipts *reduce* the receivables balance (negative in A/R's own roll-forward) but are cash *in* (positive). Capex's additions run the other way: they *increase* the asset balance (positive, an addition) but are cash *out* (negative, money spent to acquire the asset). Same underlying collision, opposite direction — both need the flip, for the same reason.
- **A module that reports a plain positive magnitude rather than a balance roll-forward line at all**, needs a flip to negative wherever that magnitude represents cash leaving the company. Payroll's disbursement and debt's interest expense are both stated as positive amounts on their own tabs — neither is a balance roll-forward line to begin with (debt's interest is deliberately kept off the principal roll-forward, same as depreciation is kept off capex's), so neither carries a cash-flow sign at the source.

**Get this table wrong and the error is invisible** — a mis-signed line doesn't throw an error, it just quietly adds instead of subtracting (or vice versa), and the model still ties to itself perfectly while being wrong. This is exactly the kind of error `financial-modeling-auditor`'s "trace the reference, not just the result" priority exists to catch — verify every row in this table against its source module before trusting it.

## The pull, module by module

| Module | Line(s) | Source sign | Action | Cash flow sign |
|---|---|---|---|---|
| A/R | Receipts on Aged A/R + Receipts on New Sales + Receipts of Other | Negative (roll-forward reduction) | **Flip** | Positive (cash in) |
| A/P | Payments of Significantly Aged + Existing + New Purchases + Holdbacks + Other | Negative (roll-forward reduction, already cash-signed) | Pull through unchanged | Negative (cash out) |
| Payroll | Payroll Cash Disbursed | Positive (a stated amount, not a roll-forward line) | **Flip** | Negative (cash out) |
| Capex | Total capex additions, at cost, by period (the diagnostic memo row) | Positive (roll-forward addition — increases the asset balance) | **Flip** | Negative (cash out) |
| Debt | Draws | Positive (roll-forward addition, already cash-signed) | Pull through unchanged | Positive (cash in) |
| Debt | Scheduled Principal + Prepayments | Negative (roll-forward reduction, already cash-signed) | Pull through unchanged | Negative (cash out) |
| Debt | Interest Expense (total across tranches, the diagnostic memo row) | Positive (a stated amount, not a roll-forward line) | **Flip** | Negative (cash out) |

**Capex's line carries its own module's caveat forward — check this before pulling it.** `capex-depreciation-builder`'s own diagnostics state that its capex-additions figure is dated at the *in-service* period, which is not necessarily the period cash actually leaves — the two only coincide by default, when payment terms weren't distinguished from in-service timing during that module's own build. Where the modeler flagged different payment terms (a deposit, progress payments) while building capex, a separate cash-timing figure should already exist in that module's notes — pull *that* one instead, and say so here. Where capex was built at its default assumption, say that explicitly too, rather than let the pulled figure imply a precision the underlying module never claimed.

Only pull rows for modules actually in scope. **Depreciation is never pulled here** — it's the one figure this entire model computes that has no cash-flow line at all, by definition; it belongs only in `PL_Memo`.

## The roll-forward

```
  Opening Cash
+ A/R receipts (flipped positive)
− A/P payments
− Payroll cash disbursed (flipped negative)
− Capex additions at cost (flipped negative)
+ Debt draws
− Debt scheduled principal and prepayments
− Debt interest expense (flipped negative)
  Closing Cash   = SUM(Opening Cash : last line)
```

Anchor the opening cash balance in the opening column, per Foundation, as a blue input or a green link to wherever the modeler supplies a starting cash balance — this schedule does not derive one.

## Closing cash going negative is not an error

**This model has no revolver.** `debt-amortization-builder` was deliberately scoped to term loans only, with no draw-and-repay facility to plug a cash shortfall. If disbursements exceed available cash in a given month, `Closing Cash` goes negative, and that is the model working correctly — it is surfacing a real funding gap, not a broken formula. Carry a memo row flagging every month where this occurs; do not build a floor, a plug, or a warning that implies something is wrong with the schedule itself. Say this plainly in the notes block so a reader doesn't mistake a real finding for a bug.

---

# 5. Build `PL_Memo`

A light accrual view, for context alongside the cash roll-forward — not a full income statement, and not trying to be one.

```
  Sales (from A/R's Sales, gross invoiced)
− Purchases (from A/P's Purchases)
− Payroll Expense Accrued (from payroll — the accrued line, never the disbursed one; this is a P&L view)
  EBITDA   = Sales − Purchases − Payroll Expense Accrued

  Memo: Depreciation (from capex, where in scope)
  Memo: Interest Expense (from debt, where in scope)
```

**This EBITDA is only as complete as what A/R and A/P actually capture.** None of these five module skills forecasts general operating expense (rent, utilities, SG&A) as its own line — if the A/P purchase forecast the modeler supplied already rolls non-payroll opex into "Purchases," this EBITDA is genuinely complete; if it only captures COGS-type trade payables, this EBITDA overstates margin by whatever opex sits outside both A/P and payroll. State which case applies in the notes block — this is exactly the kind of scope caveat Foundation's provenance discipline requires, not an optional footnote.

Depreciation and interest are carried as memo lines, not summed into EBITDA — that's the definition of the metric, not an omission.

---

# 6. Extend the shared `Checks` tab

**Every in-scope module's own Checks-tab rows land on this same shared tab** — not a separate Checks tab per module. One master ALL CLEAR / ERROR flag governs the whole combined workbook, reading clear only when every module's own checks *and* the consolidation checks below all pass.

- **Every pulled line in `Consolidated_Cash_Flow`, summed over the full horizon, ties exactly to that same total on the source module's own tab** — a mismatch means a row was dropped, duplicated, or pulled with the wrong sign in Section 4's table.
- **The cash roll-forward ties out:** Opening Cash + every line = Closing Cash, every period, with no residual.
- **Every line in `Consolidated_Cash_Flow` carries the sign the table in Section 4 specifies**, checked at the total level — A/R's total should be positive, A/P's and payroll's and capex's and debt's principal/interest totals should be negative, debt's draws total should be non-negative. A sign flipped the wrong way usually still ties (see above), so this check exists specifically to catch what the tie-out check alone would miss.
- **`PL_Memo`'s EBITDA arithmetic ties:** Sales − Purchases − Payroll Expense Accrued = EBITDA exactly, no silent residual.

## Diagnostics that are not checks

- **Months where Closing Cash is negative** — the single most important output of this entire combined model, and, as stated above, not itself a failure.
- Cumulative funding gap over the horizon, where closing cash goes negative — the rough size of financing this company would need to raise.
- EBITDA margin (EBITDA ÷ Sales), by month, from `PL_Memo`.

## Reconciliation outcomes are documentation, not checks

Whether each module's own source reconciliations passed, failed, or were carried as disclosed reconciling items belongs in that module's own notes block, per that module's own skill. The scope decision from Section 1 — which modules were included and why — belongs in this hub's own notes block. `financial-modeling-auditor` reads both.
