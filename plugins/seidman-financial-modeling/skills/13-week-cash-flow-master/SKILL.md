---
name: 13-week-cash-flow-master
description: "Orchestrates a complete, board-room and PE-diligence grade 13-week direct-method cash flow forecast by calling the A/R, A/P, and payroll builder skills in sequence against one shared spine, consolidating their outputs into a single combined cash roll-forward, and routing to variance-analysis once actuals exist. This is the hub in a hub-and-spoke skill architecture — it never invents a collection curve, a payment policy, or a wage calculation itself; every mechanic belongs to its spoke skill, and this hub's job is sequencing, consolidation, and the one combined roll-forward no spoke builds alone. Use whenever the user asks for a full, complete, or consolidated 13-week cash flow forecast, wants to combine existing A/R, A/P, and payroll modules into one model, or asks how their cash flow skills fit together. REQUIRES financial-modeling-foundation, and calls 13-week-cash-flow-builder-ar, 13-week-cash-flow-builder-ap, 13-week-cash-flow-builder-payroll, and variance-analysis as sub-skills — confirm which are available before beginning, and tell the modeler which are missing rather than substituting your own judgment for a missing module."
---

# 13-Week Cash Flow Forecast — Master / Hub

**Requires `financial-modeling-foundation`.** Load and follow it first; everything below is *additive* to it, never a replacement. If Foundation is not available, say so and ask the modeler to enable it before building.

**This skill does not model anything itself.** It never invents a collection curve, a payment policy, a wage calculation, or a variance mechanic — every one of those belongs to its own spoke skill (`13-week-cash-flow-builder-ar`, `13-week-cash-flow-builder-ap`, `13-week-cash-flow-builder-payroll`, and downstream, `variance-analysis`). Confirm each spoke you need is available before beginning. Where a spoke is missing, say so plainly and ask whether to proceed without it, rather than building that module's mechanics yourself from general knowledge — Foundation's scoped-confidence rule applies here as much as anywhere else.

## Why a hub, not one undifferentiated model

Building A/R, A/P, and payroll as one combined cash flow model from scratch becomes unmanageable quickly — too long to audit in one sitting, too long for a reviewer to hold in their head, and too many independent sub-processes competing for space inside a single file. **Build and validate each spoke on its own first.** This hub exists to combine already-validated modules into one consolidated forecast — it is not a shortcut around validating them individually, and it should never be the first time any one spoke's mechanics are exercised.

**Build in this order.**

1. Confirm scope — which spokes, and are they already built
2. Interview for shared globals only
3. Build (or re-platform) each requested spoke, in sequence
4. Consolidate into the combined cash roll-forward
5. Build the combined Checks tab
6. Route to `variance-analysis` once actuals exist

---

# 1. Confirm scope

Not every engagement needs every spoke. Ask directly:

> "Which modules does this forecast need — accounts receivable, accounts payable, payroll, or some combination? And for each one you want included: has it already been built as its own model, or does it need to be built from scratch as part of this run?"

A company with no direct employees needs only A/R and A/P. A company forecasting payroll in isolation from its trade cycle needs only payroll. Build exactly the set requested — do not add a spoke the modeler didn't ask for, and do not silently omit one they did.

**Where a spoke already exists as an independently-built standalone model** (for example, built earlier as a point-deployment demonstration), the hub's first job for that spoke is **re-platforming**, not rebuilding: carrying its assumptions and mechanics onto tabs inside the combined workbook, reading off the combined workbook's one shared spine, rather than leaving it as a separate file linked from the hub. Say explicitly when a spoke is being re-platformed versus built fresh, and confirm before doing either.

---

# 2. Interview — shared globals only

**Do not re-ask what a spoke will ask itself.** Each spoke's own interview covers its module-specific inputs; this hub asks only for what every spoke needs and would otherwise ask for redundantly:

> Before any spoke starts building, five things that apply to the whole forecast:
> 1. **Entity name**, as it should read across every module.
> 2. **Week 1 start date.** Every spoke shares this exact date — a spoke built on its own dates cannot be consolidated without rebuilding its spine.
> 3. **Reporting units** — whole dollars, thousands, or millions, consistent across every module.
> 4. **Opening cash balance**, for the combined roll-forward's starting point.
> 5. **Minimum cash threshold and revolver terms**, if a facility exists — needed for the combined Ending Cash check.

Once these five are answered, each requested spoke runs its own Phase 1 and Phase 2 interview for its module-specific inputs (customer materiality for A/R, vendor materiality and payment cadence for A/P, roster and pay-date lag for payroll) — this hub does not duplicate any of that.

---

# 3. Sequencing and the shared spine

**Foundation once, then `Macro_Assumptions` and the header/footer spine once — never rebuilt per spoke.** Every spoke reads off this single shared spine rather than each constructing its own copy. This is the single most important consolidation rule: a spoke built with its own independent spine cannot simply be dropped into the combined workbook, because its period dates, its opening column, and its row structure will not line up with the others.

**Build order across spokes**, where more than one is requested:

1. **A/R first.** It establishes the revenue-driven detail (customer-level sales forecast) that A/P's purchase forecast, where COGS-driven, often derives from.
2. **A/P second.** It consumes the revenue forecast A/R just established, where the modeler's purchase forecast is expressed as a function of sales rather than supplied independently.
3. **Payroll third**, or in parallel with A/P once the shared spine exists — it is independent of the trade cycle and has no sequencing dependency on either A/R or A/P.

This order is a default, not a rule to enforce blindly — where the modeler's purchase forecast is supplied independently rather than derived from revenue, A/P has no actual dependency on A/R having run first, and the two can build in either order. State the actual dependency, don't assume the default order always applies.

---

# 4. Consolidate — the combined cash roll-forward

Add one new tab, `Cash_Flow_Summary`, positioned immediately after `Checks` and before the individual spoke tabs — the first thing a reader sees after the check flag.

```
  Beginning Cash
+ Receipts on Aged A/R
+ Receipts on New Sales
+ Receipts of Other                    (A/R module)
− Payments on Existing A/P
− Payments on New Purchases
− Payments on Holdbacks                (A/P module)
− Payroll Cash Disbursed                (Payroll module)
− Other Disbursements                  (any module carrying one)
  Net Cash Flow
+ Beginning Cash
= Ending Cash
[Revolver Draw / (Paydown), if Ending Cash < Minimum Cash Threshold]
= Ending Cash After Revolver
```

**Every line is a green link to its spoke's own memo row on that spoke's `Forecast` tab — never retyped, and never independently re-derived.** A consolidation figure that recomputes its own version of a spoke's number, rather than summing the spoke's own line, is exactly how a hub-level total silently drifts from the detail beneath it.

**The revolver draw, if one exists, sizes off beginning-of-period balances only** — opening cash, opening revolver balance, and the period's own forecast net cash flow — per Foundation's circularity rule. Never let the draw formula reference the current period's own Ending Cash; that is the circular reference Foundation's rule exists to prevent.

---

# 5. Checks

**These live on the `Checks` tab**, per Foundation — positioned immediately after `Macro_Assumptions`, before `Cash_Flow_Summary`. A master ALL CLEAR / ERROR cell that reads clear only when every row below it passes.

**Before anything else, each spoke's own Checks tab must independently read ALL CLEAR.** A hub built on top of a spoke with an unresolved break inherits that break silently — surface each spoke's own check status on this combined tab rather than assume it, and stop if any spoke is not clear.

**Hub-level checks, beyond what each spoke already verifies on its own:**

- **Combined disbursements equal the sum of each spoke's own total disbursement memo row.** Never an independent recomputation — if this doesn't tie, a `Cash_Flow_Summary` line has drifted from its source link.
- **Ending Cash never falls below the minimum threshold without a revolver draw covering the full gap.**
- **Each spoke's `Macro_Assumptions` values — entity name, Week 1 start date, units, period count — match the shared globals exactly**, not approximately. A spoke that was re-platformed but still carries its own original dates is not actually on the shared spine, and every consolidation check above it is unreliable until that's fixed.
- **The revolver balance rolls forward cleanly:** opening + draws − paydowns = closing, every period, with no negative balance below zero paydown.

## Diagnostics that are not checks

Carry these as memo rows beneath `Cash_Flow_Summary`, not on the `Checks` tab:

- 13-week Ending Cash trend, to spot the tightest week in the forecast at a glance.
- One headline metric pulled from each active spoke's own diagnostics — DSO from A/R, DPO from A/P, cost per FTE from payroll — so a reader gets a one-line pulse on each module without opening its tab.

---

# 6. Handoff to variance-analysis

Once actuals exist for periods this forecast covers, route the comparison to `variance-analysis` rather than building a comparison mechanic here — this hub's job at that point is pointing to the right lines, not re-implementing the bridge.

- The **budget side** of the comparison is this hub's own forecast: `Cash_Flow_Summary` for the consolidated view, or an individual spoke's `Forecast` tab where the comparison is scoped to one module (for example, A/R collections performance in isolation).
- Confirm with the modeler which scope they want compared — the full consolidated cash position, or a single spoke — before handing off, since `variance-analysis`'s interview will ask for a budget file and this hub's own output can serve that role directly rather than being re-exported and re-uploaded.
- Do not duplicate `variance-analysis`'s flex-budget or price/volume/mix mechanics inside this hub. If the modeler wants that depth of explanation for a gap between this forecast and actuals, that is exactly what `variance-analysis` is for.

---

# Tab order for the combined workbook

Extends Foundation's standard order. Where a spoke's own tab list would collide with another spoke's tab names once combined (each spoke's own `Forecast`, `Cover`, and assumptions tabs), suffix each with its module — flag this rename explicitly when re-platforming a spoke that was built standalone under the unsuffixed names.

```
Cover                                — one shared cover for the combined workbook
Macro_Assumptions                    — shared globals, built once
Checks                               — combined, plus each spoke's own status surfaced
Cash_Flow_Summary                    — the consolidation
Forecast_AR / AR_Assumptions / Customer_Detail / AR_Aging        — A/R spoke, if included
Forecast_AP / AP_Assumptions / Vendor_Detail / AP_Aging          — A/P spoke, if included
Forecast_Payroll / Payroll_Assumptions / Roster_Detail           — Payroll spoke, if included
Source_Data                          — any remaining extracts, consolidated across spokes
```

---

# Scope note

Each spoke's own source reconciliation outcomes stay recorded in that spoke's own "Notes and sources" block, per that spoke's skill — this hub does not re-litigate a spoke's reconciliation, only confirms that spoke's Checks tab reads clear before consolidating it. `financial-modeling-auditor` reads each spoke's notes block independently where a full audit of the combined workbook is requested.
