---
name: 13-week-cash-flow-builder-ap
description: "Builds the accounts payable module of a board-room and PE-diligence grade 13-week direct-method cash flow forecast in Excel — a consolidated payables roll forward splitting the opening balance into significantly aged and existing pools, new purchase disbursements net of holdback, holdback accrual and release, and payment prioritization under a cash constraint. Interviews the modeler for every required input, classifies source files by the columns they carry rather than their names, and separates observed payment behaviour from stated terms so the current stretch is visible. Use whenever the user asks to build a weekly cash flow, a 13-week cash flow forecast, a rolling cash flow model, a short-term liquidity forecast, an A/P or vendor disbursement forecast, a holdback or retainage schedule, or a payment prioritization plan. REQUIRES financial-modeling-foundation — load and follow it first, and ask the user to enable it if unavailable."
---

# 13-Week Cash Flow Forecast — A/P Module

**Requires `financial-modeling-foundation`.** Load and follow it first; everything below is *additive* to it, never a replacement. Where this skill appears to contradict Foundation, Foundation wins — tell the modeler which passage conflicts rather than choosing silently. If Foundation is not available, say so and ask the modeler to enable it before building.

**Determine the build surface before anything else.** Foundation makes this non-optional, and it changes the technique for every period-driven row in this model:

- **Claude for Excel** (live Excel 365 workbook) — the full dynamic-array toolkit is available. The header spine, the Catchup and profile rows, and the cumulative holdback roll forward are built as spilling formulas.
- **Claude.ai chat or Cowork** (file delivered as a download, LibreOffice recalculation in the pipeline) — no spilling arrays. One formula copied identically across every period column.

Ask if it is not obvious. Building for the wrong surface can silently ship a broken file.

Builds the accounts payable half of a weekly direct-method cash flow forecast in Excel. Trade payables only — payroll, taxes, debt service and capex are separate disbursement lines and are not forecast here.

This module shares the header and footer spine with the A/R module. Where both are being built, build the spine once and let both read off it.

**The asymmetry that governs everything below:** A/R forecasts *behaviour* — you do not control when customers pay you. A/P forecasts *policy* — you largely control when you pay vendors. Every mechanic here therefore carries a decision the A/R module does not have, and the model must show what was decided rather than bury it in a rate.

**Build in this order:**

1. Interview the modeler — two phases
2. Classify and reconcile the sources
3. Choose the timing method and the payment policy
4. Build the spine, then the model
5. Build the Checks tab

---

# 1. Interview the modeler

## Phase 1 — Independent facts

Ask before looking at any file, as a single block. Where the A/R module already exists, inherit items 1–3 rather than asking twice.

> Before I look at any data, eight things about the model itself:
> 1. **Entity name**, as it should read on the roll forward — the beginning balance line is labelled with it.
> 2. **Week 1 start date.** The model runs Monday-to-Sunday — which Monday is Week 1?
> 3. **Reporting units** — whole dollars, thousands, or millions?
> 4. **Materiality for named vendors.** I default to the top ten by opening balance, or every vendor above 3% of the book, whichever is the longer list, pooling the rest. Different threshold?
> 5. **Payment run cadence.** How often does a run actually go out — weekly, twice monthly, monthly, continuous? Fixed day?
> 6. **Holdbacks.** What percentage is withheld, and on which vendors or contracts?
> 7. **Other disbursements** — any non-trade cash going out that belongs in this module rather than its own line? For each: the amount, and whether it recurs each period or lands in one specific period.
> 8. **Is cash constrained over the window?** If everything can be paid as it falls due, I build to behaviour. If not, I build a prioritization schedule and work backwards from available cash.

Item 5 matters more in A/P than anything equivalent in A/R. **Where payment runs are less frequent than weekly, some weeks carry no disbursement at all**, and a model that spreads payments evenly shows liquidity the company does not have on the weeks between runs.

If the modeler cannot answer 2, stop. Everything in the model is dated off the Week 1 start. Every other gap is recoverable with a named placeholder.

## Phase 2 — Sources and methodology

> Now the data. Six kinds of file matter to this model, and one upload often fills more than one of them:
> 1. **Open payables** — one row per unsettled item, with a vendor, a document date, an amount, and ideally a due date and terms.
> 2. **Payment history** — closed items carrying both a document date and a payment date. This is the one that matters most; it shows what the company actually does rather than what its terms say.
> 3. **Spend by vendor by period.**
> 4. **Purchase or COGS forecast by period** — the P&L or equivalent. This is the spine everything else reconciles to.
> 5. **Holdback or retainage schedule** — amounts withheld to date, by contract or vendor.
> 6. **Open purchase orders and goods received not invoiced** — committed spend that becomes a payable inside the window but is not in the ledger yet.
>
> Send whatever you have and I'll classify it by what's in it rather than what it's called. Where a role is missing I'll tell you what the model has to assume instead, and what to ask the client for.

**Where the purchase forecast is not weekly, ask how to convert it — do not choose silently.** A monthly figure spread evenly assumes purchasing is uniform, which it rarely is. Offer even spread, working-day weighting, or the historical weekly pattern from the payment history, and record the answer in the notes as an assumption.

**Where no payment history is offered, ask for one before building.** Without it the model cannot tell how far the company is already stretching its vendors — usually the largest single item in the first month of the forecast.

---

# 2. Classify and reconcile the sources

## Classify by shape, not by name

Foundation governs the discipline — classify by the columns a file holds, never by what it is called. **Six roles matter to this model:**

| Role | Recognise it by | Notes |
|---|---|---|
| **Open payables position** | One row per unsettled item — vendor, document date, amount outstanding. Recognise it by what it *lacks*: no payment date on any row. | Must carry a due date, or terms from which one can be computed. **Without a due date the aged/existing split cannot be made** — this is the one column the format depends on. Recompute from document date plus terms where absent, and check against any supplied aging columns. |
| **Payment history** | Closed items carrying **both** a document date and a payment date, or a document date and elapsed days-to-pay. | The only role carrying *observed* behaviour. It is what reveals the current stretch. Where absent, every timing figure is an assumption — say so in the notes. |
| **Spend by vendor by period** | Vendor, period, amount — long form or wide. | Distinguish from payment history by the absence of document-level rows. |
| **Purchase / COGS forecast by period** | The P&L or equivalent, at the model's periodicity or convertible to it. | The spine. Drives the purchases line; every other source reconciles against it. |
| **Holdback / retainage record** | Amounts withheld against invoices, often a column inside the open position rather than a file. Look for a retention, holdback or withheld column before concluding it is absent. | Carries the opening cumulative holdback. Without it the model cannot open the holdback balance at the right number. |
| **Open POs and GRNI** | Commitment or receipt records with no invoice number, or an invoice-status column reading unbilled. | **No A/R analog.** These become payables inside the window without appearing in the opening balance. Omitting them understates the back half of the forecast. |

## Derive each assumption from the best available source

Work down each row until a source exists, per Foundation.

| Assumption | Derive from | Fall back to | Last resort |
|---|---|---|---|
| Opening balance, aged/existing split | Open payables position, split on due date vs. Week 1 start | Summary aging, past-due column | Modeler input — the model cannot be built without this |
| Observed DPO | Payment history — weighted mean days to pay | Book-blended observed DPO | Stated terms, flagged as *not* observed |
| Stated terms (Payables Terms / DPO) | Open position or vendor master | Payment history implied terms | Modeler input |
| Current stretch | Observed DPO less stated terms | Book-blended stretch | Assume zero, and say so — this is a strong assumption |
| Catchup profile | Payment history — how past-due balances have actually cleared | Modeler's weekly percentages | Modeler input, named as placeholder |
| Existing pool run-off | Open position due dates | Dated forward at stated terms | Modeler's weekly percentages |
| Holdback % | Contract terms or the holdback record | Withheld value over invoiced value from the open position | Modeler input |
| Opening cumulative holdback | Holdback record | Open position retention column | Modeler input |
| Holdback release | Modeler input, by week, in every case — discretionary | — | — |
| Debit memo rate | Payment history or open position — memo value over invoiced value | Open position memos rated against the book | Modeler input |
| Spend share by vendor | Spend by vendor by period | Share of opening balance adjusted for payment speed | Share of opening balance, unadjusted |
| Weekly purchases | Purchase / COGS forecast by period | Modeler input | — |
| Committed spend not yet invoiced | Open PO and GRNI file | Modeler input | Assume nil, and disclose it |
| Other disbursements, payment run cadence | Modeler input in every case | — | — |

Foundation's three rules govern this table: a derived figure outranks a supplied one, every assumption carries its provenance into the notes block, and a fallback is not a substitute.

## The stretch is the headline number

*Compute observed DPO and stated terms separately, always, and show both.*

The difference is how far the company is already paying beyond its terms. It is not a modelling detail — it is usually the first question a lender or diligence team asks, and the number a turnaround plan is built around.

Two failures follow. **Building at terms when the company stretches** manufactures an enormous false outflow in the first fortnight, as the model pays down a book the company has no intention of clearing. **Building at observed behaviour without disclosing the stretch** presents an unsustainable position as steady state — a vendor stretched to 75 days on 30-day terms is about to move the company to COD. Carry both, with the stretch stated in days.

## Reconcile before building

Run these before a single forecast cell is written, per Foundation.

- **Open items raised in a period cannot exceed purchases invoiced in that period.** Above 100% is arithmetically impossible and means the files describe different businesses, scopes or periods.
- **Significantly aged plus existing equals the opening balance.** The split is exhaustive and non-overlapping — every open item sits in exactly one pool.
- **Open position plus payment history equals total purchases** over the window they share. A gap means one is filtered — a status, a subsidiary, a currency, an entity.
- **Spend by vendor sums to the purchase forecast**, period by period rather than only in total.
- **Open POs do not overlap the open payables position.** An item received *and* invoiced sits in both files; counting both double-counts the disbursement. Match on PO number where present, on vendor and amount where not, and disclose the basis.
- **Opening cumulative holdback ties to the open position.** Where holdback is a column inside the payables file, the sum of that column must equal the holdback record's opening balance.
- **Opening balance over trailing average daily purchases lands near the payment history's weighted days-to-pay.** Computed by different routes from different files; they should agree within a few days. When they do not, one file is stale.
- **Units and scale.** Ledger extracts are typically in whole currency units and management reporting in thousands or millions. Establish the scale of every source explicitly and convert once, on the way in.

**Where a test fails, follow Foundation's protocol** — stop, put the three options to the modeler using the script there, and record the outcome. Do not resolve a break silently.

---

# 3. Choose the timing method and the payment policy

## Fork 1 — Timing method for the Existing pool

*This fork applies only to the Existing pool. The Significantly Aged pool is always driven by Catchup, because paying a past-due balance is a decision rather than a date. New purchases always gather on the payment period row.*

> "For the part of your opening balance that isn't yet due: your aging gives me a due date on every open invoice, so I can pay it invoice by invoice, dated forward on the same convention the new-purchase gather uses. The alternative is a percentage profile — one row of thirteen weekly percentages applied to that pool. Invoice-level is more accurate and reconciles to the ledger line by line. The profile is coarser, but it is a single row a director can flex in front of a board. Which do you want? With no preference I'll build invoice-level and carry the implied percentages beneath it as an output."

| Answer | Build | Note |
|---|---|---|
| Invoice-level chosen, **or** no preference and the open position carries due dates | **Method A** | Profile is derived and shown as an output row, never an input |
| Percentage profile chosen, **or** the open position is a summary with no due dates | **Method B** | |

## Fork 2 — Payment policy

*No A/R equivalent. This determines whether disbursements drive the cash balance or the cash balance drives disbursements.*

> "Three ways to run the payments. **To behaviour** — I pay each vendor the way the company has actually been paying it, which shows what happens if nothing changes. **To terms** — I pay everything as it falls due, which shows what normalising would cost. **To constraint** — you give me a minimum cash balance to hold, and available cash each week sets what gets paid, with everything unpaid rolling into arrears. The first is the base case, the second is what vendors expect, the third is what you need if cash is tight. I can build the first two side by side — the gap between them is the funding requirement."

| Policy | Mechanic | Use when |
|---|---|---|
| **Pay to behaviour** | Existing pool and new purchases dated forward at *observed* DPO; Catchup as the modeler set it | Base case. Default when cash is not constrained |
| **Pay to terms** | Everything dated forward at *stated* terms; Catchup clears the aged pool immediately | Shows the cost of normalising. Run alongside behaviour to size the funding gap |
| **Pay to constraint** | Minimum cash floor; available cash sets disbursements; unpaid balances roll forward | Cash is constrained. Requires the prioritization layer below |

**Pay-to-constraint changes when this module runs.** Under the other two policies the A/P module feeds the cash build. Under this one it *inverts* — available cash sets disbursements, so the module runs **after** opening cash, A/R receipts and every non-A/P disbursement are already in place. Say so now, before building: discovering it later means re-sequencing the whole workbook.

Where the modeler is unsure, build **behaviour** and **terms** side by side. That gap is the funding requirement, and it is usually why the forecast was commissioned.

State both fork answers in the notes block. A reader must never have to infer from the formulas which forks the model took.

---

# 4. Build

Build in this order. **The order is not arbitrary — each step consumes something the step before it produces.**

1. **`Macro_Assumptions` globals** — entity name, units, forecast start date, period count. These come straight from Phase 1 and depend on no source data, so build them the moment Phase 1 is answered.
2. **The spine** — it reads the start date and period count from step 1. Nothing else in the model can be built correctly before it.
3. **`AP_Assumptions`** — the derived inputs, which need the source work from Phase 2.
4. **The roll forward and its components**, including the holdback mechanic.
5. **The method-specific mechanic** from Fork 1, and the prioritization layer where Fork 2 selected pay-to-constraint.

Steps 1 and 2 are unblocked by source data. **Build them while waiting for extracts** rather than holding the whole model until the files arrive.

## Format and cell conventions

Identical to the A/R module. Where both sit in one workbook the conventions are already set — do not restate them and do not diverge.

- Roboto throughout, size 11 in the forecast. **B2** entity name, Roboto 14 bold — **linked to `Macro_Assumptions`** (green font), not typed. **B3** the forecast title, built from the period count so it cannot drift (`="`&periods&`-week cash flow forecast"`). **B4** the reporting units, **linked to `Macro_Assumptions`**. Foundation defaults to actual dollars absent a stated preference.
- Row height 15.00 unless specified.
- Labels sit in column B and run across C–E as needed. **Column F is the opening column; forecast periods begin in column G** (see the spine section).

This B2:B4 block is the *sheet* header on `Forecast`, not a substitute for Foundation's Cover tab. Build both.

| Cell type | Font | Fill | Border |
|---|---|---|---|
| Input | Blue `#0000FF` | Light blue `#EEF5FC` | Hair-weight bottom |
| Formula | Black | None | None |
| Link to another tab in the same workbook | Green `#008000` | None | None |
| Link to an external file | Purple, per Foundation | None | None |

- **External links should not arise in this model.** Every source is brought into the workbook as its own tab, so the workbook is self-contained. If one does appear, Foundation's purple convention applies and it belongs in the notes.
- **Never use yellow fill to identify inputs.** Blue alone does that. Per Foundation, yellow fill means a *flagged* cell — something the modeler must look at or fill in — and carries only that meaning. No annotation text in column E or any column adjacent to the label or value columns.
- **Tab order and naming follow Foundation.** Underscores instead of spaces; no numbering. For this model:

```
Cover             — title, purpose, Change Log, version
Macro_Assumptions — entity name, currency, units, forecast start date, number of periods
Checks            — master ALL CLEAR flag and PASS/FAIL rows
Forecast          — the header spine, the A/P roll forward, memo rows
AP_Assumptions    — the schedule-level inputs listed below
Vendor_Detail     — per-vendor balances, pool split, DPO, terms, holdback applicability
AP_Aging          — the open payables position — Method A only, omit under Method B
Source_Data       — any remaining extracts
```

Zoom 80%, gridlines off, every sheet.
- Size column widths so no label is truncated — usable width is the label column plus every empty column before the first populated one.
- Freeze panes immediately below the header spine and to the left of the first forecast column.

## The header and footer spine

*Shared with the A/R module. Where it exists, read off it — do not rebuild it.*

**Build the opening column first**, per Foundation. Here it is **column F**; forecast periods begin in **column G**. It carries Week Number `0`, Week Beginning `= forecast start date − 7`, Projection Period - Date `= the Week Beginning cell above`, and two opening balances in the roll forward (below). Every other row in it is blank. Foundation's three rules apply: never summed into a total, never matched by a gather, visually distinct.

Header labels in **B6:B9**, values from column F:

| Row | Contents | Fill | Font |
|---|---|---|---|
| Week Number | `0` in the opening column, then prior cell + 1, running to the period count on `Macro_Assumptions` | `#275317` | White `#FFFFFF` |
| Actual/Forecast | `Fcst` in each column | `#3C7D22` | White `#FFFFFF` |
| Week Beginning | The weekly start date, formulaic, driven by the forecast start date input on `Macro_Assumptions` | `#3C7D22` | White `#FFFFFF` |
| Month | `=EOMONTH(week beginning,0)` | `#DAF2D0` | Black `#000000` |

Never white font on `#DAF2D0` — unreadable.

Footer block, headed **Model Mapping** in bold black, bounded top and bottom with a solid border:

| Row | Formula | Purpose |
|---|---|---|
| Projection Period - Date | `= the Week Beginning cell in the header` | The date every SUMIFS in the model matches against |
| Month Beginning | `=IF(prior period's Month <> this period's Month, this period's Month, "")` | Flags the first week of each month |
| Month Ending | `=IF(next period's Month <> this period's Month, this period's Month, "")` | Flags the last week of each month |
| Transaction Adjustment | `=prior week projection period - ROUND((prior week projection period - current week projection period)/2,0)` | Places the assumed average transaction date inside the week rather than on its boundary |
| A/R-to-Cash Collection Period | `=7-WEEKDAY(transaction adjustment + current week DSO on new sales,2)+(transaction adjustment + current week DSO on new sales)+1` | Snaps the expected collection date forward to a Monday, so it matches a Week Beginning date exactly |
| A/P-to-Cash Payment Period | `=7-WEEKDAY(transaction adjustment + current week DPO,2)+(transaction adjustment + current week DPO)+1` | Same convention, for the A/P module |

**This module drives the A/P-to-Cash Payment Period row.** The DPO it takes is the *policy* DPO — observed under pay-to-behaviour, stated terms under pay-to-terms.

**No first-column exception.** Transaction Adjustment references the prior column's Projection Period Date, which the opening column supplies, so the formula is identical in every forecast column including the first. Do not write a variant formula in the first forecast period.

**Why the snap matters.** `7-WEEKDAY(d,2)+d+1` advances any date to the following Monday, so payment dates land exactly on header dates and the `SUMIFS` gathers match.

**Where payment runs are less frequent than weekly**, add a run-week flag row beneath the spine — `1` on run weeks, `0` otherwise — and advance the payment date to the next flagged week.

**The Month Ending flag drives holdback accrual.** It exists precisely for monthly events inside a weekly model; do not build a separate calendar.

**Thirteen weeks is the default, not a constant.** Foundation requires the horizon to roll forward and extend from a `Macro_Assumptions` input, never by rebuilding. Every period-driven row — the Week Number spine, Catchup, the Existing profile, the holdback release row — sizes off that input. Read "Week 13" throughout this skill as "the final forecast period."

- **In Claude for Excel**, drive the spine with `SEQUENCE` off the period count, and combine periods and any total column in a single `HSTACK` so the range resizes as one unit. Cumulative Holdbacks is a stateful roll-forward and is a legitimate `SCAN`/`REDUCE` candidate — pair it with a plain-English note saying what accumulates and what releases it.
- **In Claude.ai chat or Cowork**, extend by adding columns. The formula in each column is identical; the period count input governs how many are populated.

**Which lines flip**, under Foundation's roll-forward convention: **none of the payment lines flip.** They reduce A/P and are cash out, so they carry through negative in both. This is the opposite of A/R, whose receipts must flip to positive — label both so the asymmetry is visible. Debit memos never reach the cash flow statement; they reduce the balance without moving cash.

## The roll forward

The consolidated block, on the forecast sheet. In column B, at least two rows below the header spine:

```
  Beginning Balance - [Entity Name]
+ Purchases
−  Payments of Significantly Aged
−  Payments of Existing
−  Payments on New Purchases
−  Payments on Holdbacks
−  Payments of Other
  Ending Balance   = SUM(Beginning Balance : Payments of Other)

  Memo: Cumulative Holdbacks
  Memo: Trade payables excluding holdback   = Ending Balance − Cumulative Holdbacks
  Memo: Implied DPO on ending balance
```

Link `[Entity Name]` to `Macro_Assumptions` rather than typing it, so the label cannot drift from the cover block. All payment lines are carried negative, so the ending balance is a single SUM down the block.

**Two opening balances anchor in the opening column**, per Foundation, both as green links to `AP_Assumptions`: the opening A/P balance in the **Ending Balance** cell, and the opening cumulative holdback in the **Cumulative Holdbacks** memo row. Both roll forwards then read `= the prior column's` value in every forecast column including the first.

**One block, not one per vendor.** Vendor detail lives on its own tab and feeds the derivation of DPO, the pool split and the prioritization queue — but the forecast sheet shows the consolidated position.

**Three pools, three mechanics.** The opening balance splits on due date against the Week 1 start:

| Pool | Definition | Paid by |
|---|---|---|
| **Significantly Aged** | Open items **already past due** at the Week 1 start | Catchup profile. Always modeler-driven |
| **Existing** | Open items **not yet due** at the Week 1 start | Method A or Method B per Fork 1 |
| **New Purchases** | Invoiced inside the window | Gather on the A/P-to-Cash Payment Period row, **net of holdback** |

The split is exhaustive and non-overlapping. Every open item sits in exactly one pool, and the two opening pools sum to the beginning balance.

## Holdbacks

*A portion of every purchase never enters the normal payment gather at all. This is the mechanic most easily got wrong, because nothing in the roll forward makes the withholding visible — it shows up only as an amount that fails to be paid.*

**Assumptions carried:**

```
Payables Terms (DPO)
Holdback %
Monthly Holdbacks
Catchup
Cumulative Holdbacks
```

**The mechanic:**

- **Holdback %** is withheld off gross purchases. Payments on New Purchases therefore gather on `purchases × (1 − holdback %)`, not on gross.
- **Monthly Holdbacks** is the amount withheld in each month, accrued off purchases and summarised on the **Month Ending** flag from the footer spine.
- **Cumulative Holdbacks** rolls forward: `prior week cumulative + holdback withheld this week − holdback released this week`. Open it at the balance from the holdback record.
- **Payments on Holdbacks** is the release, taken as **modeler input by week** — a horizontal row of release amounts, one column per forecast period.

**There is no withholding line in the roll forward, and there should not be.** Withheld amounts simply are not paid, so they stay in the ending balance on their own. Adding a line for them double counts. What the reader needs instead is the *memo*: cumulative holdback against the ending balance, and the trade portion as the difference.

**Release is discretionary** — a lever on the same footing as Catchup, flexed rather than justified; under a cash constraint it competes with every other payment for available cash. **Where holdback applies to some vendors and not others**, derive the effective book-level percentage as withheld over invoiced value for those vendors, weighted by their share of purchases — do not apply a contract-level percentage across a book where most vendors carry none.

## Catchup — the Significantly Aged pool

Past-due balances do not clear on a date. They clear on a decision, and Catchup is where that decision is stated.

- Lay Catchup out **horizontally** on `AP_Assumptions`: a formulaic row of week numbers sized to the period count, with input percentages beneath. Each is a share of the **opening significantly aged balance**.
- Pull it onto the forecast sheet with `INDEX` against the Catchup cell range, indexed on this column's Week Number. **Use a direct range reference or a `LET` label — never a Name Manager range**, per Foundation.
- **The percentages need not sum to 100%.** Where they do not, the shortfall is past-due balance still outstanding at Week 13 — show it as an explicit residual rather than leaving it implicit in the ending balance.
- Default starting point where the modeler has no view: **40 / 30 / 20 / 10** across Weeks 1–4. Name it as a placeholder.
- **Under pay-to-terms**, Catchup clears the pool in Week 1 by definition — everything past due is already due. Show what that costs; it is usually the largest single week in the forecast and the reason a facility is needed.
- **Under pay-to-constraint**, Catchup is an *output*, not an input. The prioritization queue determines what gets paid, and the implied percentages are derived and displayed beneath.

```
Payments of Significantly Aged
  =-(opening significantly aged balance)*(current week catchup %)
```

## The Existing pool

**Method A — invoice-level.** Put the open payables position on its own tab, one row per item, and never retype a figure from it. Where the source arrives grouped with subtotals or with the vendor carried once as a group header, flatten it to one clean row per item and reconcile the flattened total back to the source's own total.

Add a computed expected payment date:

```
=7-WEEKDAY(due date + current stretch,2)+(due date + current stretch)+1
```

The stretch is zero under pay-to-terms and the observed figure under pay-to-behaviour. **Using `due date + stretch` raw, without the snap, misaligns the pool against new purchases by up to a week.**

```
Payments of Existing
  =-SUMIFS(payables tab amount column, payables tab expected payment date column, Projection Period Date)
```

Carry an explicit total for items falling due **beyond Week 13**, and derive the implied weekly percentages as an output row beneath the payments line, black font, no fill.

**Method B — percentage profile.** Lay the profile out horizontally, week numbers sized to the period count with input percentages beneath, each a share of the **opening existing balance**. Pull it on with `INDEX` against that range on Week Number. Where the percentages fall short of 100%, show the residual explicitly.

```
Payments of Existing
  =-(opening existing balance)*(current week profile %)
```

## New purchases

```
Payments on New Purchases
  =-SUMIFS(purchases gross invoiced × (1 − holdback %), all periods to date,
           A/P-to-Cash Payment Period, all periods to date,
           Projection Period Date)
```

Where payment runs are less frequent than weekly, gather to the next flagged run week rather than the raw payment period.

**Committed spend from open POs and GRNI** enters Purchases in the week it is expected to be invoiced, then flows through this gather like any other purchase. Carry the expected invoice week as an input, and tie total conversions back to the PO file.

## Vendor detail

A single blended pool assumes the largest vendor behaves like the tail. It does not — one critical supplier moving to COD is the whole variance. **Segment on a supporting tab**, even though the forecast sheet shows one consolidated block.

- Name individually every vendor above the Phase 1 threshold. Pool the rest as **"All other vendors (n)"**, n being the count.
- Each named vendor carries its own opening balance, aged/existing split, observed DPO, stated terms, stretch and holdback applicability. The tail carries one balance-weighted set.
- **Flag any named vendor whose DPO falls to the book-blended fallback.** Naming a vendor asserts its behaviour differs from the pool; giving it the pool average silently withdraws that assertion.
- **Where no spend forecast by vendor exists**, derive each vendor's share of purchases from its share of opening A/P **adjusted for payment speed**. A vendor paid slowly holds more A/P per dollar of spend, so its share of purchases is *lower* than its share of the book. Divide each balance by that vendor's average age relative to the book average, then normalise.
- **Shrink each vendor's speed factor toward the book average — 40% is a reasonable default.** A vendor whose only open invoices are days old is not being paid fast; it is one that happened to invoice late in the period.
- The vendor spend forecast must tie to the P&L purchase line **at the P&L's own periodicity, not just in total**. Carry the P&L line and a variance row beneath, and require zero variance at that periodicity.

## The assumptions tabs

Foundation splits these across two homes. **Global controls live on `Macro_Assumptions`; everything scoped to the A/P schedule lives on `AP_Assumptions`.** No forecast formula contains a typed value in either case.

**On `Macro_Assumptions`** (shared with every other schedule — do not duplicate here):

```
Entity name
Currency and reporting units
Forecast start date (Week 1 beginning — Monday)
Number of forecast periods
Minimum cash floor, by period   — pay-to-constraint only; a cross-cutting control
```

**On `AP_Assumptions`:**

```
Opening A/P balance
  — Significantly aged (past due at Week 1 start)
  — Existing (not yet due at Week 1 start)
Payables Terms (DPO) — stated
Observed DPO, and the stretch in days
Holdback %
Cumulative Holdbacks — opening balance
Debit memos — aged balance, and new purchases (% of gross invoiced)
Payments on Holdbacks — horizontal, one column per forecast period
Catchup — horizontal, one column per forecast period
Vendor roster — opening balance, aged/existing split, observed DPO, terms, holdback applicability
Committed spend not yet invoiced, by expected invoice week
Other disbursements per week (non-trade)
Payment run cadence and run-week flags
```

Method- and policy-specific inputs — carry only those that apply:

| Input | Applies to |
|---|---|
| Existing pool payment profile (horizontal, one column per forecast period) | **Method B only.** Under Method A this is an output |
| Minimum cash floor, by week | **Pay-to-constraint only** |
| Vendor tier and tier payment rule | **Pay-to-constraint only** |

**Monthly Holdbacks is calculated, not input** — it is holdback % applied to each month's purchases, summarised on the Month Ending flag. Carry it as a formula row, not on the input list.

**Layout.** Labels in the label column, values in a single dedicated value column, right-justified so figures align on the decimal. Keep conditional check text short enough to sit inside the value column. Carry a legend naming the input, formula and link conventions, and a memo comparing the opening balance to trailing average daily purchases, expressed in days.

**Close with the "Notes and sources" block.** Every assumption gets a note naming the source role and derivation tier. Every placeholder gets named as a placeholder, in full. Where an assumption fell to a fallback tier, state what extract would have allowed a measured figure. Record every reconciliation test, passed or failed. State both fork answers.

---

# The prioritization layer — pay-to-constraint only

*Build only where the modeler answered "to constraint" at Fork 2. Under the other policies, disbursements drive the cash balance and none of this applies.*

The model inverts: **available cash sets what gets paid**. This module therefore runs after the cash build rather than feeding it, and needs opening cash, A/R receipts and every non-A/P disbursement already in place.

**Each period:** available cash = opening cash + receipts − non-A/P disbursements − minimum cash floor. Guard against a negative result — where the floor cannot be held before any vendor is paid, surface that rather than paying a negative amount. Rank the queue by tier, then by days past due within tier, and pay down until available cash is exhausted. Everything unpaid stays in the ending balance and re-enters next period's queue one period older.

**Holdback release enters the queue like anything else** — no priority of its own, ranked by the tier of the vendor it belongs to and deferred as cash requires. Where deferring release would itself trigger a stop-supply risk, that risk belongs in the vendor's tier, not in a separate rule for holdbacks.

**Vendor tiers:**

| Tier | Definition | Rule |
|---|---|---|
| **Critical** | Sole-source, contractual stop-supply rights, safety- or regulatory-essential, or a small balance with disruption out of all proportion to it | Paid in full, on time, before any other tier. Never deferred |
| **Essential** | Needed for operations but substitutable, or willing to hold a balance | Paid to terms where cash allows, deferred before critical |
| **Deferrable** | Everything else | Paid from residual cash only |

**Tier is not size.** The most common error is tiering by balance — a small vendor with a sole-source part stops the line just as effectively as a large one, and is far cheaper to keep current. Ask the modeler to name critical vendors explicitly rather than inferring tier from the ledger. **What the constraint costs.** The schedule alone is not the deliverable: carry arrears balance and aging by tier, flag the period each vendor crosses its stop-supply threshold, and carry the cumulative funding gap (pay-to-terms less pay-to-constraint disbursements) — the number that sizes a facility request.

---

# 5. Checks

**These live on the `Checks` tab**, per Foundation: a master ALL CLEAR / ERROR cell that reads clear only when every row below passes, and one clearly-labelled PASS/FAIL row per check with conditional formatting. Do not scatter check cells through the forecast sheet.

Foundation caps this at a handful of checks that catch real breakage. The set below is deliberately short; anything more exhaustive is `financial-modeling-auditor` territory.

**All methods and policies:**

- **Pool split is exhaustive:** significantly aged + existing = opening balance, with no item in both.
- **Cumulative Holdbacks rolls forward cleanly:** prior + withheld − released = current, every period, with no negative balance.
- **Ending Balance is never less than Cumulative Holdbacks.** A breach means holdback has been paid through the trade gather.
- **Payments on New Purchases gather on net-of-holdback purchases**, not gross — reconcile total withheld to `purchases × holdback %`.
- Named vendors plus the pooled tail sum to the opening balance.
- Vendor spend forecast ties to the P&L purchase line **at the P&L's own periodicity** — every period where it is weekly, every monthly subtotal where it is monthly. Never only in total.
- Open POs and GRNI convert to payables exactly once, with no overlap against the opening position.

**Method A only:**

- Every open item in the Existing pool is assigned to exactly one forecast period or the beyond-horizon residual, and the two sum to the opening existing balance.

**Method B only:**

- Existing pool profile sums to 100%, or the shortfall is shown as an explicit final-period residual.

**Pay-to-constraint only:**

- The minimum cash floor is never breached in any period.
- Total payments in each period are less than or equal to available cash in that period.
- No critical-tier vendor is deferred in any period.
- Arrears roll forward completely: closing arrears = opening arrears + amounts due − amounts paid, every period.

## Diagnostics that are not checks

These are analytical outputs, not pass/fail conditions. **Carry them as memo rows beneath the roll forward on `Forecast`**, not on the `Checks` tab:

- Cumulative Holdbacks, and trade payables excluding holdback.
- Implied DPO on the ending balance, period by period.
- Observed DPO and stated terms side by side, with the stretch in days.
- Under pay-to-constraint: arrears aging by tier, vendors crossing a stop-supply threshold, and the cumulative funding gap.

## Reconciliation outcomes are documentation, not checks

Whether each source reconciliation passed, failed, or was carried as a disclosed reconciling item belongs in the **"Notes and sources" block**, with the modeler's recorded decision. It is a statement about the inputs rather than a test of the model, and `financial-modeling-auditor` reads it there.
