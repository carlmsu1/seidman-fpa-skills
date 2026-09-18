---
name: variance-analysis
description: "Builds a variance bridge comparing actual results to budget or a prior forecast in Excel — flexed budget mechanics that separate volume-driven variance from rate and spending variance, an optional price/volume/mix decomposition, and an EBITDA bridge waterfall. Interviews the modeler for scope and decomposition depth, classifies actual and budget/forecast source files by the columns they carry, and reconciles them before building so every named driver sums exactly to the actual-less-budget gap with no silent residual. Use whenever the user asks to build a budget-vs-actual analysis, a variance bridge, a flexed budget, a price/volume/mix decomposition, or an EBITDA bridge, or wants to explain why actual results differ from plan. This skill REQUIRES financial-modeling-foundation — always load and follow it first, and ask the user to enable it if it is not available."
---

# Variance Analysis — Flexed Budget and EBITDA Bridge

**Requires `financial-modeling-foundation`.** Load and follow it first; everything below is *additive* to it, never a replacement. Where this skill appears to contradict Foundation, Foundation wins — tell the modeler which passage conflicts rather than choosing silently. If Foundation is not available, say so and ask the modeler to enable it before building.

**Determine the build surface before anything else.** Foundation makes this non-optional, and it changes the technique for every period column in the bridge:

- **Claude for Excel** (live Excel 365 workbook) — the full dynamic-array toolkit is available. The period header row and the bridge waterfall are built as spilling formulas.
- **Claude.ai chat or Cowork** (file delivered as a download, LibreOffice recalculation in the pipeline) — no spilling arrays. One formula copied identically across every period column.

Ask if it is not obvious. Building for the wrong surface can silently ship a broken file.

**This model looks backward, not forward.** The A/R, A/P, and payroll builder skills forecast periods that have not happened yet; this one explains periods that already have. Where those three share a rolling 13-week spine, this one's spine is whatever completed periods are in scope — a month, a quarter, or a full year — and the "opening column" convention from Foundation applies only where a cumulative year-to-date roll-up is in scope, not to every build.

**The single idea that governs everything below:** a variance is not one number, it is several, and collapsing them into one line hides which of them is real performance and which is just the plan being wrong about volume. **Actual minus Budget** mixes "we sold a different amount than planned" with "we spent or charged a different amount per unit than planned." A **flexed budget** — what the budget would have said at the actual volume — separates the two. Everything in this skill exists to make that separation explicit rather than leave it buried in one unexplained gap.

**Build in this order.**

1. Interview the modeler — two phases
2. Classify and reconcile the sources
3. Choose the decomposition depth
4. Build the assumptions, then the bridge
5. Build the Checks tab

---

# 1. Interview the modeler

## Phase 1 — Independent facts

Ask these before looking at any file, as a single block.

> Before I look at any data, six things about the analysis itself:
> 1. **Entity name**, as it should read on the bridge.
> 2. **Comparison periodicity and periods in scope** — monthly, quarterly, or annual, and which specific period(s)?
> 3. **Reporting units** — whole dollars, thousands, or millions?
> 4. **What's being compared to actual** — the original approved budget, the latest reforecast, or the prior year's same period? Only one of these is "the budget" in the flex mechanic below; if more than one comparison is wanted, I'll build one bridge per comparison rather than blend them.
> 5. **Metric(s) in scope** — revenue, gross margin/COGS, opex by category, EBITDA, or all of it rolled into one EBITDA bridge?
> 6. **Materiality threshold** for calling out a line item individually versus rolling it into an "other" bucket.

If the modeler cannot answer 2, stop — every other question depends on knowing which periods are being explained.

## Phase 2 — Sources and methodology

Ask only after Phase 1 is answered.

> Now the data. Five kinds of file matter to this analysis, and one upload often fills more than one of them:
> 1. **Actual results** — the closed-period P&L or GL/trial balance extract, at the periodicity in scope.
> 2. **Budget or plan file** — the approved static budget, set once and unchanged regardless of when it's pulled.
> 3. **Latest forecast file** — only if the comparison in Phase 1 item 4 is to a reforecast rather than the original budget; distinguish this from the budget by its revision date or version tag.
> 4. **Volume or unit data**, actual and budgeted — only needed for a price/volume/mix decomposition; skip if a simpler total-dollar variance is sufficient.
> 5. **Standard rate or cost file** — the price or cost per unit the budget was built on. This is what makes flexing possible; without it, the budget can only be compared at its original total, not re-calculated at actual volume.
>
> Send whatever you have and I'll classify it by what's in it rather than what it's called. Where a role is missing I'll tell you what the analysis has to assume instead, and what to ask for.

**Where volume data exists but standard rates do not, say so before building rather than defaulting to a total-dollar variance silently** — a price/volume/mix decomposition without a standard rate has nothing to flex against, and the modeler may prefer to derive an implied rate (budgeted revenue ÷ budgeted volume) as a named assumption rather than skip the decomposition entirely.

---

# 2. Classify and reconcile the sources

## Classify by shape, not by name

*Never route on a filename, a tab name, or the system a file came out of. Open every source, read its header row, and classify it by the columns it holds.*

| Role | Recognise it by | Notes |
|---|---|---|
| **Actual results** | A closed period — every row has a single settled value, no forward-looking or placeholder figures. | Recognise a management P&L from a raw GL export by aggregation level; either works, but note which was used, since a GL export may need account-to-line mapping the P&L has already done. |
| **Budget / plan** | A single, unchanging set of values regardless of when the file is pulled — the number set at the start of the planning cycle. | Distinguish from a forecast by the absence of a revision date; a true static budget doesn't have one because it was never meant to move. |
| **Latest forecast** | Carries a revision date or version tag distinguishing it from the original budget. | Only relevant where Phase 1 named the forecast, not the budget, as the comparison basis. |
| **Volume / unit data** | Units, not dollars — counts, hours, or another physical measure, actual and budgeted. | Required only for price/volume/mix; a total-dollar variance does not need it. |
| **Standard rate / cost** | Price or cost per unit, as budgeted. | The flex mechanic's pivot — without it, the budget can be compared at its original total but not recalculated at actual volume. |

## Derive each assumption from the best available source

Work down each row until a source exists. Take the first one that does, and record which one it was.

| Assumption | Derive from | Fall back to | Last resort |
|---|---|---|---|
| Standard price/rate per unit | Standard rate file | Budgeted revenue ÷ budgeted volume, named as an implied rate | Modeler input, named as placeholder |
| Budgeted volume | Volume/unit data | Budgeted revenue ÷ standard rate, back-solved | Modeler input |
| Actual volume | Volume/unit data | Actual revenue ÷ standard price, named as an approximation | Skip price/volume/mix; build total-dollar variance only |
| Budget-line comparison basis | Budget or latest forecast, per Phase 1 item 4 | — | Modeler input |
| Mix weights (multi-product only) | Volume/unit data by product | Revenue share by product | Modeler input |

Foundation's three rules govern this table: a derived figure outranks a supplied one, every assumption carries its provenance into the notes block, and a fallback is not a substitute.

## Reconcile before building

Run these before a single bridge cell is written, per Foundation.

- **Actual results tie to the GL or trial balance** for the period(s) in scope, not a management-adjusted figure that hasn't been reconciled back.
- **Budget total ties to the approved budget document**, not a re-run or an informally updated copy.
- **Volume × standard price approximately reconciles to budgeted revenue.** This is the sanity check on the flex mechanism itself — if it does not hold within a small tolerance, the standard rate or the volume figure is stale relative to the budget it's meant to explain.
- **Units of measure are consistent** across the volume file and the rate file — a volume in cases and a rate per unit will silently misstate the flex by whatever the pack size is.

**Where a test fails, follow Foundation's protocol** — stop, put the three options to the modeler using the script there, and record the outcome. Do not resolve a break silently.

---

# 3. Choose the decomposition depth

*This is a fork in the model, not a styling preference. Put the question to the modeler and wait. Do not choose on their behalf.*

> "I can build this bridge at three depths. The simplest is a **total variance only** — actual minus budget, one number per line, no explanation of why. Better is a **flexed budget**, which recalculates what the budget would have said at your actual volume, so you can see how much of the gap is 'we sold a different amount' versus 'our costs or pricing moved.' The fullest is a **price/volume/mix decomposition**, which further splits that into a price effect, a volume effect, and — if you sell more than one product — a mix effect from selling more of one thing than another. The fuller versions need unit volume and a standard rate; the simple version doesn't. Which do you want?"

| Answer | Build | Note |
|---|---|---|
| Total variance only, **or** no volume/rate data available | **Depth 1** | Actual − Budget per line, no flex |
| Flexed budget | **Depth 2** | Adds a Volume Variance and a combined Price/Spending Variance |
| Price/volume/mix | **Depth 3** | Splits Depth 2's price/spending variance further into price and mix, where more than one product exists |

State the chosen depth and the modeler's answer in the notes block. A reader must never have to infer from the formulas which depth the model was built at.

---

# 4. Build

Build in this order. **The order is not arbitrary — each step consumes something the step before it produces.**

1. **`Macro_Assumptions` globals** — entity name, units, periods in scope, comparison basis. Build the moment Phase 1 is answered.
2. **`Bridge_Assumptions`** — standard rates, volumes, mix weights, which need the source work from Phase 2.
3. **The bridge itself** — one row per P&L line, columns for each named driver.
4. **The EBITDA bridge summary**, rolling the line-level bridges into one waterfall.

## Format and cell conventions

Identical to the cash-flow builder modules, per Foundation.

- Format the full model in **Roboto**. Labels, dates, values and cell contents are all Roboto 11.
- **B2** — company name, Roboto 14 bold, linked to `Macro_Assumptions` (green font). **B3** — analysis title, built from the periods in scope. **B4** — reporting units, linked to `Macro_Assumptions`.
- Row height 15.00 throughout unless specified otherwise. Labels in column B, running across C–E as needed.

| Cell type | Font | Fill | Border |
|---|---|---|---|
| Input | Blue `#0000FF` | Light blue `#EEF5FC` | Hair-weight bottom |
| Formula | Black | None | None |
| Link to another tab in the same workbook | Green `#008000` | None | None |
| Link to an external file | Purple, per Foundation | None | None |

- **Never use yellow fill to identify inputs** — reserved for flagged cells per Foundation.
- **No annotation or comment text in a column adjacent to the bridge.** All notes belong in the "Notes and sources" block on `Bridge_Assumptions`.
- **Tab order and naming follow Foundation.** Underscores instead of spaces; no numbering. For this model:

```
Cover              — title, purpose, Change Log, version
Macro_Assumptions  — entity name, units, periods in scope, comparison basis
Checks             — master ALL CLEAR / ERROR flag and PASS/FAIL rows
Bridge             — the line-by-line variance bridge
Bridge_Assumptions — standard rates, volumes, mix weights, the "Notes and sources" block
EBITDA_Bridge      — the rolled-up waterfall summary
Source_Data        — any remaining extracts
```

## The bridge — Depth 1 (total variance)

```
  Budget
+ Total Variance   = Actual − Budget
  Actual
```

One row per P&L line. No flex, no decomposition — this is the floor every deeper build starts from, and it never disappears even at Depth 3: it is the total that every named driver at a deeper depth must still sum to.

## The bridge — Depth 2 (flexed budget)

```
  Budget (static, as approved)
+ Volume Variance          = (Actual Volume − Budget Volume) × Standard Price
  Flexed Budget            = Budget + Volume Variance
+ Price / Spending Variance = Actual − Flexed Budget
  Actual
```

**The Flexed Budget row is the pivot of the whole model.** It answers "what would the budget have said if it had known the actual volume in advance?" Volume Variance isolates the effect of selling a different amount at the standard price; everything left over — Price/Spending Variance — is not explained by volume at all, and is therefore about rate (for a revenue line) or cost control (for an expense line).

**Guard the interpretation by line type.** On a revenue line, a positive Price/Spending Variance means realized price ran above standard. On a cost line, the same positive sign means cost ran *above* budget at the actual volume — worse, not better. Label each row's Price/Spending Variance as "Price Variance" for revenue lines and "Spending Variance" for cost lines so the sign never has to be mentally flipped by the reader.

## The bridge — Depth 3 (price/volume/mix)

Only where more than one product or service line exists — with a single product, mix is undefined and Depth 3 collapses back to Depth 2.

```
  Budget (static, as approved)
+ Volume Variance    = (Total Actual Volume − Total Budget Volume) × Blended Budget Price
+ Mix Variance       = Total Actual Volume × (Actual Mix % − Budget Mix %) × (Product's Standard Price − Blended Budget Price)
  Flexed Budget       = Budget + Volume Variance + Mix Variance
+ Price Variance      = (Actual Price − Standard Price) × Actual Volume, summed across products
  Actual
```

**Mix Variance is the effect of selling a different proportion of products than planned, holding total volume and each product's own price fixed.** A shift toward higher-priced products shows as a positive mix variance even if total units sold exactly hit budget — this is usually the row a reader finds most informative and most often confuses with price.

**Where mix cannot be measured** — a single blended volume figure with no per-product breakdown — do not force a mix split; fall back to Depth 2 for that line and say so in the notes rather than manufacture a mix number the data cannot support.

## The EBITDA bridge summary

A waterfall rolling every line's variance drivers up into one walk from Budget EBITDA to Actual EBITDA:

```
Budget EBITDA
+ Revenue: Volume Variance
+ Revenue: Price Variance (or Mix, at Depth 3)
+ COGS: Volume Variance
+ COGS: Spending Variance
+ Opex: Spending Variance (by category, or rolled to one line under the materiality threshold)
+ Other / Unexplained (see Checks — this row should be zero or near-zero; a nonzero balance here is a signal, not a plug)
= Actual EBITDA
```

Each named driver on this summary is a link to its corresponding row on `Bridge`, never re-derived independently — a hub-level EBITDA bridge that recomputes its own numbers instead of summing the line-level bridge is how a summary silently drifts from its detail.

---

# 5. Checks

**These live on the `Checks` tab**, per Foundation: a master ALL CLEAR / ERROR cell that reads clear only when every row below it passes, and one clearly-labelled PASS/FAIL row per check with conditional formatting. Do not scatter check cells through the bridge itself.

Foundation caps this at a handful of checks that catch real breakage. The set below is deliberately short; anything more exhaustive is `financial-modeling-auditor` territory.

**All depths:**

- **Every named driver on a line sums exactly to that line's Total Variance (Actual − Budget).** This is the single most important check in this model — a decomposition that doesn't foot to the total it's decomposing is worse than no decomposition, because it looks authoritative while being wrong.

**Depth 2 and 3 only:**

- **Flexed Budget arithmetic ties:** Budget + Volume Variance (+ Mix Variance, at Depth 3) = Flexed Budget, exactly.
- **Volume × Standard Price reconciles to the Flexed Budget revenue line** within the tolerance established during source reconciliation.

**Depth 3 only:**

- **Actual mix percentages sum to 100%** across products, every period.
- **Price Variance uses each product's own standard price**, not the blended rate — spot-check a sample row.

**EBITDA bridge:**

- **The waterfall ties end to end:** Budget EBITDA + every named driver = Actual EBITDA, with no gap.
- **The Other/Unexplained row is small relative to Total Variance** (a reasonable default: under 5%). Where it is not, a driver is missing from the decomposition rather than genuinely unexplained — say so rather than let the row absorb it silently.

## Diagnostics that are not checks

These are analytical outputs, not pass/fail conditions. **Carry them as memo rows beneath the bridge**, not on the `Checks` tab:

- Variance as a percentage of budget, by line, to flag which lines moved most in relative rather than absolute terms.
- The three largest variance drivers by absolute dollar value, called out explicitly rather than left for the reader to find.
- Trend versus the same variance in the prior period, where multiple periods are in scope — is a variance widening or narrowing.

## Reconciliation outcomes are documentation, not checks

Whether each source reconciliation passed, failed, or was carried as a disclosed reconciling item belongs in the **"Notes and sources" block**, with the modeler's recorded decision. It is a statement about the inputs rather than a test of the model, and `financial-modeling-auditor` reads it there.
