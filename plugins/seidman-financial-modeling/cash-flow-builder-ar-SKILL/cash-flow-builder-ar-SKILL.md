---
name: cash-flow-builder-ar
description: "Builds the accounts receivable module of a board-room and PE-diligence grade monthly direct-method cash flow forecast in Excel — aged and new-sales collections, aging waterfall, credit memos, uncollectibles by bucket, and customer segmentation. Interviews the modeler for every required input, classifies source files by the columns they carry rather than their names, reconciles the sources before building, and routes between invoice-level and percentage-profile collection methods. Use whenever the user asks to build the A/R piece of a cash flow forecast, an A/R collections forecast, a rolling A/R cash flow model, an A/R liquidity forecast, or wants to forecast cash receipts on a monthly basis. For a combined model spanning A/R, A/P, payroll, capex and/or debt, use `master-cash-flow-builder` instead, which orchestrates this skill as one of its modules. This skill REQUIRES financial-modeling-foundation — always load and follow it first, and ask the user to enable it if it is not available."
---

# Cash Flow Forecast — A/R Module

**Requires `financial-modeling-foundation`.** Load and follow it first; everything below is *additive* to it, never a replacement. Where this skill appears to contradict Foundation, Foundation wins — tell the modeler which passage conflicts rather than choosing silently. If Foundation is not available, say so and ask the modeler to enable it before building.

**Determine the build surface before anything else.** Foundation makes this non-optional, and it changes the technique for every period-driven row in this model:

- **Claude for Excel** (live Excel 365 workbook) — the full dynamic-array toolkit is available. The header spine, the profile rows and the roll forward are built as spilling formulas.
- **Claude.ai chat or Cowork** (file delivered as a download, LibreOffice recalculation in the pipeline) — no spilling arrays. One formula copied identically across every period column.

Ask if it is not obvious. Building for the wrong surface can silently ship a broken file.

Builds the accounts receivable half of a monthly direct-method cash flow forecast in Excel. The output is a workbook a director can take into a board meeting or a diligence room: every hardcoded number on an assumptions tab, every assumption carrying its provenance, every reconciliation test recorded whether it passed or failed.

**Build in this order.** Each phase depends on the one before it, and skipping ahead is the most common way this model goes wrong.

1. Interview the modeler — two phases
2. Classify and reconcile the sources
3. Choose the collection method
4. Build the spine, then the model
5. Build the Checks tab

---

# 1. Interview the modeler

## Phase 1 — Independent facts

Ask these before looking at any file. They are things the modeler knows without opening a ledger, and the answers determine what the sources even need to contain. Ask them as a single block, not one at a time.

> Before I look at any data, six things about the model itself:
> 1. **Company name**, as it should read on the cover of the forecast.
> 2. **Month 1 start date.** Which calendar month opens the forecast — the first of that month becomes Month 1's start.
> 3. **Reporting units** — whole dollars, thousands, or millions?
> 4. **Materiality for named customers.** I default to naming the top ten by opening balance, or every customer above 3% of the book, whichever is the longer list, and pooling the rest. Do you want a different threshold?
> 5. **Other receipts** — any non-trade cash coming in (grants, rebates, interest, insurance, settlements)? For each: the amount, and whether it recurs monthly or lands in one specific month. Where the month is uncertain, tell me the range and I will carry it as a dated input you can flex.
> 6. **DPO** — days payable outstanding, for the payment-period row in the footer spine. This module doesn't forecast A/P, but the spine carries the row.

If the modeler cannot answer 2, stop. Everything in the model is dated off the Month 1 start. Every other gap is recoverable with a named placeholder.

## Phase 2 — Sources and methodology

Ask only after Phase 1 is answered.

> Now the data. Six kinds of file matter to this model, and one upload often fills more than one of them:
> 1. **Open receivables** — one row per unsettled item, with a customer, a document date and an amount.
> 2. **Settlement history** — closed items carrying both a document date and a settlement date. This is the one that matters most; it is the only file that shows how customers actually pay rather than how they are supposed to.
> 3. **Revenue by customer by period.**
> 4. **Revenue forecast by period** — the P&L or equivalent. This is the spine everything else reconciles to.
> 5. **Write-off or bad-debt record** — often just a status column inside the settlement history.
> 6. **Credit memos** — usually negative lines inside one of the above rather than a file of their own.
>
> Send whatever you have and I'll classify it by what's in it rather than what it's called. Where a role is missing I'll tell you what the model has to assume instead, and what to ask the client for.

**Where the revenue forecast's periodicity doesn't already match the model's monthly periods, ask how to convert it — do not choose silently.** A quarterly or annual forecast spread evenly across months assumes revenue is uniform within the coarser period, which it rarely is; seasonal weighting can shift a month's receipts materially. A weekly or daily source needs the reverse treatment — summed up into months, watching for weeks that straddle a month boundary. Put it to the modeler:

> "Your revenue forecast doesn't already run monthly. Do you want it spread evenly, weighted by working days, or weighted by a historical monthly pattern in the settlement history — or, if it's finer than monthly, summed up into months? Even is simplest; historical weighting is the most accurate where the data supports it. The choice can move receipts by a meaningful amount, so it is worth a moment."

Record the answer in the notes block as an assumption, not a mechanic.

**Where no settlement history is offered, ask for one explicitly before building.** DSO, the collection profile, the loss curve and the memo rate are all measured with it and all assumed without it — and they are the four assumptions that move the forecast most. It is a standard extract from any ledger and usually a five-minute request.

---

# 2. Classify and reconcile the sources

## Classify by shape, not by name

*Never route on a filename, a tab name, or the system a file came out of. A file called "aging" may be a summary with no dates in it, and the file that actually carries the collection behaviour may be called anything at all. Open every source, read its header row, and classify it by the columns it holds.*

Six roles matter. One upload may fill more than one; one role may be split across several uploads; a role may be absent entirely. Classify what is there, name what is missing, and never assume a role is filled because a file exists.

| Role | Recognise it by | Notes |
|---|---|---|
| **Open receivables position** | One row per unsettled item — customer, document date, amount outstanding. Recognise it by what it *lacks*: no settlement date on any row, because every row is still open. | May also carry due date, payment terms and pre-computed aging columns. Use those where present; recompute from the document date where not, and check the recomputation against the supplied columns before trusting either. |
| **Settlement history** | Closed items carrying **both** a document date and a settlement date, or a document date and elapsed-days-to-settle. | The only role carrying *observed* payment behaviour. Where present, nothing about collection timing needs to be assumed. Where absent, everything about collection timing is an assumption — say so in the notes rather than let derived-looking numbers imply otherwise. |
| **Revenue by customer by period** | Customer, period, amount — long form or wide. | Distinguish from settlement history by the absence of document-level rows. |
| **Revenue forecast by period** | The P&L or equivalent, at the model's periodicity or convertible to it. | This is the spine. It drives the sales line, and every other source reconciles against it rather than the other way round. |
| **Loss and write-off record** | Sometimes standalone, more often a status value inside the settlement history. | Look for the status column before concluding the role is unfilled. |
| **Credit memo record** | Almost never a separate file. Negative amounts inside the open position, the settlement history, or both. | |

**The settlement history is the pivot.** With it, DSO, the collection profile, the loss curve and the memo rate are all measured. Without it, all four are assumed.

## Derive each assumption from the best available source

Work down each row until a source exists. Take the first one that does, and record which one it was.

| Assumption | Derive from | Fall back to | Last resort |
|---|---|---|---|
| Opening balance, aging split | Open receivables position | Trial balance plus a summary aging | Modeler input — the model cannot be built without this |
| DSO by customer | Settlement history — weighted mean days to settle | Payment terms on the open position | Book-blended DSO applied to every customer |
| Aged collection profile | Settlement history — days-to-settle distribution applied to the open position | Open position dated forward at terms | Modeler's monthly percentages |
| Loss rate by aging bucket | Settlement history — written-off value over value reaching each age | Standalone write-off record | Indicative curve, named as placeholder |
| Uncollectibles, new sales | Settlement history — written-off value over value invoiced | The Current bucket loss rate, since new sales are current by definition | Modeler input, named as placeholder |
| Credit memo rate | Settlement history or open position — memo value over invoiced value | Open position memos rated against the book | Modeler input |
| Sales share by customer | Revenue by customer by period | Share of opening balance adjusted for payment speed | Share of opening balance, unadjusted |
| Monthly sales | Revenue forecast by period | Modeler input | — |
| Other receipts, DPO | Modeler input in every case | — | — |

Foundation's three rules govern this table: a derived figure outranks a supplied one, every assumption carries its provenance into the notes block, and a fallback is not a substitute.

## Reconcile before building

Run these before a single forecast cell is written, per Foundation.

- **Open items raised in a period cannot exceed revenue invoiced in that period.** Group the open position by document date into the model's periods and compare each against the revenue forecast's actual periods. Anything above 100% is arithmetically impossible and means the two files describe different businesses, scopes or periods.
- **Open position plus settlement history equals total revenue** over the window they share, once credit memos are excluded. A gap means one of the two is filtered — a status, a subsidiary, a currency, an entity.
- **Revenue by customer sums to the revenue forecast**, period by period rather than only in total.
- **The customer list agrees across sources.** A name in one and absent from another is usually a merge, a rename or a parent-child rollup, not a new account.
- **Opening balance over trailing average daily sales lands near the settlement history's weighted days-to-settle.** These are computed from different files by different routes; they should agree within a few days. When they do not, one of the two files is stale.
- **Units and scale.** Ledger extracts are typically in whole currency units and management reporting in thousands or millions. Establish the scale of every source explicitly and convert once, on the way in.

**Where a test fails, follow Foundation's protocol** — stop, put the three options to the modeler using the script there, and record the outcome. Do not resolve a break silently.

---

# 3. Choose the collection method

*This is a fork in the model, not a styling preference. Put the question to the modeler and wait. Do not choose on their behalf.*

An invoice-level aging file already carries the date and amount of every open item. Where one exists, a percentage profile is a lossy summary of data the model is already holding in full — it collapses many dated invoices into a handful of numbers, and everything the collapse discards (which customer, which bucket, which memo) has to be reintroduced by assumption further down. Where no such file exists, the profile is the only way to express run-off at all.

Both are legitimate. They are not interchangeable: the choice changes the assumptions tab, the receipts formula, and the checks.

> "Your aging file gives me the date and the amount of every open invoice, so I can collect the opening balance invoice by invoice — each one dated forward by its customer's collection DSO, using the same date convention the new-sales gather uses. The alternative is the percentage profile: one row of monthly percentages applied to the opening balance. The invoice-level gather is more accurate and reconciles to the ledger line by line. The profile is coarser, but it is a single row a director can flex in front of a board. Which do you want? If you have no preference I will build the invoice-level gather and carry the implied monthly percentages beneath it as an output, so you have both."

| Answer | Build | Note |
|---|---|---|
| Invoice-level chosen, **or** no preference and an open position with document dates exists | **Method A** | Profile is derived and shown as an output row, never carried as an input |
| Percentage profile chosen, **or** the only open position is a bucket summary with no document dates | **Method B** | |
| No open receivables position of any kind | **Method B** | Say so in the notes. Every monthly percentage is then the modeler's assumption, not the client's data |

State the method and the modeler's answer in the notes block. A reader must never have to infer from the formulas which fork the model took.

---

# 4. Build

Build in this order. **The order is not arbitrary — each step consumes something the step before it produces.**

1. **`Macro_Assumptions` globals** — entity name, units, forecast start date, period count. These come straight from Phase 1 and depend on no source data, so build them the moment Phase 1 is answered.
2. **The spine** — it reads the start date and period count from step 1. Nothing else in the model can be built correctly before it.
3. **`AR_Assumptions`** — the derived inputs, which need the source work from Phase 2.
4. **The roll forward and its components.**
5. **The method-specific mechanic** from Fork 1.

Steps 1 and 2 are unblocked by source data. **Build them while waiting for extracts** rather than holding the whole model until the files arrive.

## Format and cell conventions

- Format the full model in **Roboto**. Forecast labels, dates, values and cell contents are all Roboto 11.
- **B2** — company name. Roboto 14, bold. **Link it to `Macro_Assumptions`** (green font) rather than typing it.
- **B3** — the forecast title, built from the period count so it cannot drift: `="`&periods&`-month cash flow forecast"`. Roboto 11, plain.
- **B4** — the reporting units, written out. **Link to `Macro_Assumptions`** (green font). Foundation defaults to actual dollars where the modeler expresses no preference.

- Row height 15.00 throughout unless specified otherwise.
- Labels sit in column B and run across C–E as needed. **Column F is the opening column; forecast periods begin in column G** (see the spine section).

This B2:B4 block is the *sheet* header on `Forecast`, not a substitute for Foundation's Cover tab. Build both — the Cover carries the Change Log and version; this block identifies the sheet when it is printed or exported on its own.

| Cell type | Font | Fill | Border |
|---|---|---|---|
| Input | Blue `#0000FF` | Light blue `#EEF5FC` | Hair-weight bottom |
| Formula | Black | None | None |
| Link to another tab in the same workbook | Green `#008000` | None | None |
| Link to an external file | Purple, per Foundation | None | None |

- **External links should not arise in this model.** Every source is brought into the workbook as its own tab, so the workbook is self-contained. If one does appear, Foundation's purple convention applies and it belongs in the notes.
- **Never use yellow fill to identify inputs.** The blue input convention alone identifies what the reader must replace. Per Foundation, yellow fill is reserved for *flagged* cells — something the modeler must look at or fill in — and carries that meaning only.
- **No annotation or comment text in column E**, or any column adjacent to the label or value columns. All notes belong in the "Notes and sources" block on `AR_Assumptions`.
- **Tab order and naming follow Foundation.** Underscores instead of spaces; no numbering. For this model:

```
Cover             — title, purpose, Change Log, version
Macro_Assumptions — company name, currency, units, forecast start date, number of periods
Checks            — master ALL CLEAR flag and PASS/FAIL rows
Forecast          — the header spine, the A/R roll forward, memo rows
AR_Assumptions    — the schedule-level inputs listed below
Customer_Detail   — per-customer balances, DSO, profiles, sales share
AR_Aging          — the open receivables position — Method A only, omit under Method B
Source_Data       — any remaining extracts
```

Foundation permits a schedule's own assumptions to live with that schedule, so `AR_Assumptions` is correct — but **global items belong on `Macro_Assumptions`, not here**. Where a compact model makes a separate `AR_Assumptions` tab unnecessary, stack it at the top of `Forecast` behind a labelled section header row.

- Zoom 80% and gridlines off on every sheet.
- Size column widths so no label is truncated. A label is cut off when the cell to its right holds a value, so the usable width is the label column plus every empty column before the first populated one — widen until the longest label on the sheet fits inside that span.
- Freeze panes immediately below the header spine and to the left of the first forecast column.

## The header and footer spine

*Every timing mechanic in this model reads off these rows. Nothing else can be built correctly without them.*

**Shared with the A/P module.** Where the spine already exists because A/P was built first, read off it — do not rebuild it.

### Header

**Build the opening column first**, per Foundation. Here it is **column F**; forecast periods begin in **column G**. It carries Month Number `0`, Month Beginning `= EDATE(forecast start date, -1)`, Projection Period - Date `= the Month Beginning cell above`, and the opening A/R balance in the roll forward's Ending Balance row (below). Every other row in it is blank. Foundation's three rules apply: never summed into a total, never matched by a gather, visually distinct.

Labels in **B6:B8**. Across from each, from column F:

| Row | Contents | Fill | Font |
|---|---|---|---|
| Month Number | `0` in the opening column, then prior cell + 1, running to the period count on `Macro_Assumptions` | `#275317` | White `#FFFFFF` |
| Actual/Forecast | `Fcst` in each column | `#3C7D22` | White `#FFFFFF` |
| Month Beginning | The monthly start date, formulaic, driven by the forecast start date input on `Macro_Assumptions` via `EDATE` — never by adding a fixed day count, since months vary in length | `#DAF2D0` | Black `#000000` |

The band runs dark to pale down the spine. Font is white on the two darker greens and black on the pale green — **never white on `#DAF2D0`**, which is unreadable.

**No separate "Month" row.** A finer-than-monthly spine would need a row deriving which calendar month each period falls into, purely to flag month boundaries. That has no purpose here — each column already *is* a month — so no such row exists.

### Footer

Leave two blank rows below the last assumption row. On the next row, in column B, insert the title **Model Mapping** in bold black. The block begins on the row immediately below, with formulas running as far right as the header spine displays.

Bound the block top and bottom with a solid border: the top border on the *Projection Period - Date* row, the bottom on the *A/P-to-Cash Payment Period* row. Both run the full width of the block.

| Row | Formula | Purpose |
|---|---|---|
| Projection Period - Date | `= the Month Beginning cell in the header` | The date every SUMIFS in the model matches against |
| Transaction Adjustment | `=prior period projection period - ROUND((prior period projection period - current period projection period)/2,0)` | Places the assumed average transaction date at the midpoint of the period rather than on its boundary |
| A/R-to-Cash Collection Period | `=EOMONTH(transaction adjustment + current period DSO on new sales,0)+1` | Snaps the expected collection date forward to the first of the following month, so it matches a Month Beginning date exactly |
| A/P-to-Cash Payment Period | `=EOMONTH(transaction adjustment + current period DPO,0)+1` | Same convention, for the A/P module |

**No first-column exception.** Transaction Adjustment references the prior column's Projection Period Date, which the opening column supplies, so the formula above is identical in every forecast column including the first. Do not write a variant formula in the first forecast period.

**Why the snap matters.** `EOMONTH(d,0)+1` advances any date forward to the first day of the *following* month — every date inside a given month, including the 1st of that month itself, resolves to the same target: the next month's start. This mirrors the logic a weekly spine would use to snap every date inside a week forward to the *following* Monday, never the current one — the same "always strictly forward" principle, just applied at month grain. Because every Month Beginning is the first of a month, this makes the collection date land exactly on a header date, so the `SUMIFS` gathers match. Using `date + DSO` raw, without the snap, silently misaligns receipts against the header spine — and because months vary from 28 to 31 days, that drift is larger and less predictable than a week's ever would be.

**Twelve months is a common default, not a constant.** Foundation requires the horizon to roll forward and extend from a `Macro_Assumptions` input, never by rebuilding. Every period-driven row — the Month Number spine, the collection profile, the past-due spread — sizes off that input. Read "the final month" throughout this skill as "the final forecast period."

- **In Claude for Excel**, drive the spine with `SEQUENCE` off the period count, and combine periods and any total column in a single `HSTACK` so the range resizes as one unit. Generate each period's Month Beginning date with `EDATE`, never by adding a fixed day count — month lengths vary, and a fixed offset will drift off calendar-month boundaries within a year.
- **In Claude.ai chat or Cowork**, extend by adding columns. The formula in each column is identical; the period count input governs how many are populated.

**Which lines flip**, under Foundation's roll-forward convention: **Receipts on Aged A/R, Receipts on New Sales and Receipts of Other all flip to positive** on the way into the cash flow statement — they reduce A/R but are cash in. Credit memos and write-offs never reach the cash flow statement at all; they reduce the balance without moving cash.

**The A/P row and the DPO input are scaffolding for the A/P module — nothing in this skill consumes either.** Build them where A/P is being built too, so both modules share one timing basis. **Where only A/R is being built, skip both**: leave the row out of the footer and drop question 6 from the Phase 1 interview. An unused input on an assumptions tab is an audit finding, not harmless.

## The assumptions tabs

Foundation splits these across two homes. **Global controls live on `Macro_Assumptions`; everything scoped to the A/R schedule lives on `AR_Assumptions`.** No forecast formula contains a typed value in either case.

**On `Macro_Assumptions`** (shared with every other schedule — do not duplicate here):

```
Company name
Currency and reporting units
Forecast start date (Month 1 beginning — the first of the month)
Number of forecast periods
```

**On `AR_Assumptions`:**

```
Opening A/R balance — aged (start of Month 1)
Opening A/R split by aging bucket (Current / 30-59 / 60-89 / 90-119 / 120+)
Loss rate by aging bucket
Customer roster — opening balance, average age, speed vs. book, collection DSO, sales share
DSO on new sales (days)
Credit memos — aged balance, and new sales (% of gross invoiced)
Uncollectibles — new sales (% of gross invoiced)
Other receipts — recurring per period, and any one-off items with their expected period
DPO (days)
```

Two inputs are method-specific — carry only the one that applies:

| Input | Method |
|---|---|
| Aged A/R collection profile (horizontal, one column per forecast period) | **Method B only.** Under Method A this is an output, not an input |
| Past-due catch-up spread (horizontal, first two months) | **Method A only** |

**Layout.** Labels in the label column, every value in a **single dedicated value column**. Size that column to fit its longest entry, and right-justify everything in it so figures align on the decimal. Keep conditional check text short enough to sit inside the value column — a long failure message forces the column wider than the numbers need. Carry a **legend** naming the input, formula and link conventions, and a **memo comparing the opening A/R balance to trailing average daily sales, expressed in days**, so the modeler can sanity-check the aging total against the P&L.

**Close with the "Notes and sources" block.** Every assumption gets a note naming the source role it was derived from and the tier of the derivation table it landed on. Every placeholder the modeler supplied rather than the client gets named as a placeholder, in full, so nothing unsourced reaches a board pack. Where an assumption fell to a fallback tier, state what extract would have allowed a measured figure instead. Record the outcome of every reconciliation test, passed or failed, and carry any accepted break as a named reconciling item rather than folding it into an assumption.

## The A/R roll forward

In column B, at least two rows below the header spine:

```
  A/R Beg. Bal.
+ Sales (gross invoiced)
− Credit Memos
− Receipts on Aged A/R
− Receipts on New Sales
− Receipts of Other
− Write-offs — Aged A/R
− Write-offs — New Sales
  A/R End. Bal.   = SUM(Beg. Bal. : last reduction line)
```

Receipts and write-offs are carried negative, so the ending balance is a single SUM down the block rather than a chain of signs.

**The opening balance anchors in the opening column**, per Foundation: put it in the **A/R End. Bal.** cell of column F as a green link to `AR_Assumptions`, not in Month 1's Beginning Balance. `A/R Beg. Bal. = the prior column's A/R End. Bal.` then holds in every forecast column including the first.

## Customer segmentation

A single blended pool assumes the largest account behaves like the tail. On a monthly horizon it does not — one large customer slipping a month is the whole variance. **Segment the book before applying any of the mechanics below.**

- Name individually every customer above the materiality threshold from Phase 1. Pool the remainder on a single row labelled **"All other customers (n)"**, where n is the count.
- Each named customer carries its own opening aged balance, its own collection DSO and its own aged collection profile. The tail carries one balance-weighted set of the same three.
- **Flag any named customer whose DSO falls to the book-blended fallback.** Naming a customer asserts its behaviour differs from the pool; assigning it the pool average silently withdraws that assertion. Name these in the notes and say what would measure them — usually a longer settlement history.
- Receipts on Aged A/R and Receipts on New Sales become **the sum across the customer pools**, not a single calculation. Build the customer detail on its own tab and sum onto the forecast sheet.
- **Where no sales forecast by customer exists**, derive each customer's share of sales from its share of opening A/R **adjusted for payment speed**. A slow payer holds more A/R per dollar of sales, so its share of sales is *lower* than its share of the book. Divide each balance by that customer's average age relative to the book average, then normalise.
- **Shrink each customer's speed factor toward the book average — 40% is a reasonable default — before using it.** A customer whose only open invoices are days old is not a fast payer; it is a customer that happened to invoice late in the period.
- The customer sales forecast must tie to P&L sales **at the finer of the two periodicities, never just in total** — every month where the P&L is monthly; where the P&L is coarser than monthly (quarterly or annual), tie the model's monthly figures rolled up to that coarser periodicity. Carry the P&L line and a variance row beneath the customer block, and require zero variance at that periodicity.

## Credit memos

Credit memos reduce a future collection. They are **not** a collection, and must never appear in a receipts line.

- Give them their own row directly beneath Sales (gross invoiced), carried negative, in the period the memo is issued.
- Apply each memo against the balance of the customer it belongs to, and against the pool it belongs to — aged or new.
- **Reduce the collectible base before the collection mechanic runs.** For the aged pool, the profile applies to the opening aged balance *less* aged credit memos. For new sales, the gather collects on gross invoiced *less* new-sales credit memos.
- Because the memo row and the reduced collectible base together account for the full opening balance, the pool still resolves to zero. Test this — it is the fastest way to catch a memo being double counted.
- Source the memo rate from the aging file: negative lines are credit memos already issued. Rate them against the book and carry the result as an input, per pool.

> **Under Method A:** negative lines in the aging file already carry their own date, customer and bucket. Let them flow through the gather against the item they belong to, rather than rating them against the book and carrying a blended input.

## Uncollectibles by aging bucket

A single loss rate across the whole book understates the tail and overstates the current bucket. **Loss experience is a curve.**

Carry the opening balance split by aging bucket on `AR_Assumptions`, taken from the aging file, with a check that the buckets sum to the opening A/R balance. Carry a loss rate against each bucket as a separate input.

Indicative starting points where no loss history exists. **These are placeholders and must be named as such in the notes block:**

| Bucket | Indicative loss rate |
|---|---|
| Current | 0.25% |
| 30–59 | 1% |
| 60–89 | 5% |
| 90–119 | 25% |
| 120+ | 60% |

Aged write-offs in total = `SUMPRODUCT(bucket balances, bucket loss rates)`. **New sales keep a single rate** — they have no aging yet, so there is nothing to bucket, and being current by definition the Current bucket's rate is the natural proxy where no separate measurement exists.

> **Under Method B:** recognise the total pro-rata as the pool runs off, so the timing treatment is unchanged and only the amount is bucket-derived.
>
> **Under Method A:** each invoice carries its own bucket, so apply that bucket's rate to that invoice and skip the pro-rata allocation entirely.

## New sales gather

New sales collect through the date-driven gather in every case — the method fork affects only the *aged* pool.

```
Receipts on New Sales
  =-SUMIFS(sales gross invoiced, all periods to date,
           A/R-to-Cash Collection Period, all periods to date,
           Projection Period Date)
    *(1 - write-off rate for current period)

Write-offs — New Sales
  =-SUMIFS(sales gross invoiced, all periods to date,
           A/R-to-Cash Collection Period, all periods to date,
           Projection Period Date)
    *(write-off rate for current period)
```

Both gather against the **A/R-to-Cash Collection Period** row in the footer spine, which is why that row must be built first.

---

# Method A — the invoice-level gather

Under this method the percentage profile is **not an input at all**. It is derived from the gather and displayed as an output row.

**Set up the source tab.** Put the open receivables position on its own tab, one row per open item, and **never retype a figure from it anywhere else in the workbook**. Where the source arrives grouped with subtotal rows, or with the customer carried once as a group header rather than on every row, flatten it to one clean row per item on the way in — then reconcile the flattened total back to the source's own total before using it.

**Date each item forward.** Add a computed column for the expected collection date:

```
=EOMONTH(invoice date + that customer's collection DSO,0)+1
```

This is the same convention the A/R-to-Cash Collection Period row uses, so the aged run-off and the new-sales gather sit on one timing basis. **Using `invoice date + DSO` raw, without the snap, silently advances the aged pool by up to a month against new sales and manufactures a liquidity hole in the handover between them.**

The collection DSO each item is dated forward by comes from the settlement history where one exists, from payment terms where one does not. Say which in the notes: dating the pool forward at terms assumes every customer pays to terms, which is the assumption the model exists to test.

**Past-due items need their own treatment.** Every invoice whose expected collection date already sits before the Month 1 start returns a date outside the window and would otherwise vanish. **Do not clamp them into Month 1** — no collections function clears its entire past-due book in the first month. Carry the past-due balance as a separate sub-pool on `AR_Assumptions` with a short catch-up spread as input percentages — a default of **60 / 40** across Months 1–2 is reasonable — and total it against the aging so the modeler can see how much of the book is being cleared on assumption rather than on date. Where the past-due slug is large relative to the book, say so in the notes: it is usually the single largest swing factor in the forecast.

**Two outputs the profile method cannot produce.** Carry an explicit total for every open item whose expected collection date falls **beyond the final forecast period** — the profile method hides this inside a residual; the gather can state it, and a reader will ask for it. And derive the implied monthly percentages from the gather, shown as an output row beneath the receipts line in black font, no fill.

```
Receipts on Aged A/R
  =-SUMIFS(aging tab amount column, aging tab expected collection date column, Projection Period Date)
    *(1 - that item's bucket loss rate)
  summed across customer pools, plus the past-due sub-pool at its catch-up percentage for the period

Write-offs — Aged A/R
  =-SUMIFS(aging tab amount column, aging tab expected collection date column, Projection Period Date)
    *(that item's bucket loss rate)
  Amount and timing are both invoice-derived; no pro-rata allocation needed.
```

---

# Method B — the percentage profile

The opening A/R balance is every invoice raised before the Month 1 start date. It is collected on **its own run-off schedule**, never through the new-sales gather — mixing the two double counts the collections.

- Lay the profile out **horizontally** on `AR_Assumptions`: a formulaic row of month numbers sized to the period count, with a row of input percentages directly beneath it. Each percentage is a share of the **opening aged balance**, not of that month's sales.
- Total the percentages, add the aged uncollectible rate, and confirm the two sum to 100%. **Surface this on the `Checks` tab, not as an inline cell** — Foundation reserves error signalling to that tab.
- Pull the profile onto the forecast sheet with `INDEX` against the profile's cell range, indexed on this column's Month Number, so the row tracks the header spine rather than a hardcoded offset. **Use a direct range reference or a `LET` label — never a Name Manager range**, per Foundation.

**Sourcing the profile.** Ask the modeler for the aging file. Absent one, the run-off must be **consistent with the collection mechanic**: at a given DSO the new-sales receipts do not begin until the collection period formula lands inside the forecast window, so the aged pool has to carry every month before that point — or the model will show a false liquidity hole.

```
Receipts on Aged A/R
  =-(opening aged balance less aged credit memos)*(current period aged collection profile %)
  summed across customer pools

Write-offs — Aged A/R
  The aged uncollectible amount, recognised pro-rata as the pool runs off:
  =-SUMPRODUCT(bucket balances, bucket loss rates)*(current period profile % / total profile %)
  Guard the denominator against zero.
```

The pro-rata step is a Method B compromise. It exists because the model has no invoice-level detail to attach a bucket rate to.

---

# 5. Checks

**These live on the `Checks` tab**, per Foundation: a master ALL CLEAR / ERROR cell that reads clear only when every row below passes, and one clearly-labelled PASS/FAIL row per check with conditional formatting. Do not scatter check cells through the forecast sheet.

Foundation caps this at a handful of checks that catch real breakage. The set below is deliberately short; anything more exhaustive is `financial-modeling-auditor` territory.

**Both methods:**

- **Aged pool fully resolved:** opening A/R + total credit memos + total aged receipts + total aged write-offs = 0.
- Aging buckets sum to the opening A/R balance.
- Named customers plus the pooled tail sum to the opening A/R balance.
- Customer sales forecast ties to P&L sales **at the finer of the two periodicities** — every month where the P&L is monthly, every quarterly or annual subtotal where the P&L is coarser than monthly. Never only in total.

**Method A only:**

- Every open item on the aging tab is assigned to exactly one forecast month, a past-due catch-up month, or the beyond-horizon residual — and the three sum to the opening A/R balance.
- Past-due catch-up percentages sum to 100% of the past-due sub-pool.

**Method B only:**

- Collection profile + aged uncollectible rate = 100%. Where uncollectibles are bucket-derived, use the blended rate: total bucket write-offs divided by the opening balance.

## Diagnostics that are not checks

These are analytical outputs, not pass/fail conditions. **Carry them as memo rows beneath the roll forward on `Forecast`**, not on the `Checks` tab:

- Implied DSO on the A/R ending balance, period by period, to confirm the roll forward behaves.
- Implied monthly collection percentages, where Method A derives them as an output.

## Reconciliation outcomes are documentation, not checks

Whether each source reconciliation passed, failed, or was carried as a disclosed reconciling item belongs in the **"Notes and sources" block**, with the modeler's recorded decision. It is a statement about the inputs rather than a test of the model, and `financial-modeling-auditor` reads it there.
