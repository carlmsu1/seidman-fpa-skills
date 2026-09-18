---
name: payroll-schedule-builder
description: "Builds a board-room and PE-diligence grade payroll forecast in Excel that reports on a monthly spine while computing every dollar at the payroll system's real frequency — weekly, bi-weekly, or semi-monthly — before rolling up. A bottom-up, roster-driven schedule covering gross wages, employer payroll taxes with wage-base caps tracked off cumulative YTD wages at true pay-period grain, benefits, PTO liability roll-forward, and bonus accrual and payout, aggregated to monthly output columns without smoothing away the wage-base cap's real crossing point or the fact that months contain a variable number of pay periods. Interviews the modeler for every required input, classifies source files by the columns they carry rather than their names, and separates the pay period a cost is accrued in from the pay date the cash actually moves. Use whenever the user asks to build a payroll forecast, a headcount cost forecast, a payroll cash disbursement forecast, or wants to forecast payroll expense or cash on a monthly basis — standalone, or alongside a monthly A/R or A/P build. For a combined model spanning A/R, A/P, payroll, capex and/or debt, use `master-cash-flow-builder` instead, which orchestrates this skill as one of its modules. This skill REQUIRES financial-modeling-foundation — always load and follow it first, and ask the user to enable it if it is not available."
---

# Payroll Schedule Builder

**Requires `financial-modeling-foundation`.** Load and follow it first; everything below is *additive* to it, never a replacement. Where this skill appears to contradict Foundation, Foundation wins — tell the modeler which passage conflicts rather than choosing silently. If Foundation is not available, say so and ask the modeler to enable it before building.

**Determine the build surface before anything else.** Foundation makes this non-optional, and it changes the technique for every period-driven row in this model — including a row-dynamic structure Foundation's own guidance doesn't directly cover (see the Pay_Period_Ledger note below):

- **Claude for Excel** (live Excel 365 workbook) — the full dynamic-array toolkit is available. The monthly header spine, the pay-period ledger's row generation, the cumulative wage-base tracker, and the PTO and bonus accrual roll-forwards are all built as spilling formulas.
- **Claude.ai chat or Cowork** (file delivered as a download, LibreOffice recalculation in the pipeline) — no spilling arrays anywhere, rows or columns. One formula copied identically down every ledger row and across every period column.

Ask if it is not obvious. Building for the wrong surface can silently ship a broken file.

Builds a monthly, cash-basis payroll forecast in Excel — payroll only; it does not forecast trade payables or other non-payroll disbursements. It stands alone: it does not assume a companion A/R or A/P build exists, though its monthly output tabs and conventions match the monthly A/R and A/P modules closely enough to combine by hand.

**The asymmetry that governs everything below:** most cash forecasting problems are about behaviour or policy — when a customer actually pays, when a vendor actually gets paid. Payroll forecasts a **roster** instead — once the roster, rates, and elections are set, most of the arithmetic is deterministic rather than assumed. Two places judgment still enters at any periodicity: the **pay-date lag** (the pay period a cost is earned in is rarely the exact date the cash moves) and the **wage-base cap** (employer tax stops accruing once an employee's cumulative year-to-date wages cross a statutory threshold, so the same gross-wage dollar can cost more early in the year than late in it).

**A third fact matters specifically because this schedule's output is monthly.** Payroll doesn't run on a monthly clock, and calendar months don't divide evenly into pay periods — a weekly payroll has either four or five paydays in a given month, semi-monthly always has exactly two, bi-weekly usually has two but occasionally three. Treating every month as a uniform 1/12 of the year (annual salary ÷ 12) discards this entirely — understating cost in a five-payday month, overstating it in a four-payday one, and positioning the wage-base cap's crossing point wrong for every employee near it. **Everything below computes at the pay period's real frequency and aggregates to months only at the final step, specifically so this effect shows up correctly instead of being averaged away.**

**Build in this order.**

1. Interview the modeler — two phases
2. Classify and reconcile the sources
3. Choose the roster method and set the pay-date lag
4. Build the monthly spine, the pay-period ledger, then the rollup
5. Build the Checks tab

---

# 1. Interview the modeler

## Phase 1 — Independent facts

Ask these before looking at any file, as a single block.

> Before I look at any data, eight things about the model itself:
> 1. **Entity name**, as it should read on the roll forward.
> 2. **Month 1 start date.** Which calendar month opens the forecast — the first of that month becomes Month 1's start. This is the monthly *output* spine's start; it does not have to land on a pay date.
> 3. **Reporting units** — whole dollars, thousands, or millions?
> 4. **Pay frequency.** Weekly, bi-weekly, semi-monthly, or monthly.
> 5. **Anchor pay date** — one actual pay date, past or upcoming, so I can project every other pay date forward (or back) at the stated frequency without guessing which week of a bi-weekly or which half of a semi-monthly cycle you're on. Any real pay date works; it doesn't need to be near Month 1's start.
> 6. **Pay date lag** — how many days after a pay period ends does the pay date actually fall? This decides which *month* the cash leaves, which is no longer automatically the same month the cost was earned once the lag is large enough to cross a month boundary.
> 7. **Materiality for named employees.** I default to naming anyone above a fixed dollar threshold you set, or anyone whose role makes their cost worth tracking individually (owners, executives, commissioned roles), and pooling the rest by role or department. Different threshold, or a different basis than dollars?
> 8. **Employer tax jurisdiction(s), and benefits/PTO/bonus scope.** Which state(s) or country sets the FUTA/SUTA (or local equivalent) rates and wage-base thresholds? And does the forecast need employer-paid benefits, a PTO liability roll-forward, and bonus accrual/payout, or is base wages and statutory tax enough?

If the modeler cannot answer 2 or 5, stop. The monthly spine is dated off item 2; every pay date in the ledger is dated off item 5. Every other gap is recoverable with a named placeholder.

## Phase 2 — Sources and methodology

Ask only after Phase 1 is answered. The pay-period ledger consumes source data at true pay-period grain, regardless of the model's monthly output.

> Now the data. Five kinds of file matter to this model, and one upload often fills more than one of them:
> 1. **Roster / HRIS export** — one row per employee (or open requisition), with role, pay type (salary or hourly), rate, FTE or scheduled hours, and start/end dates.
> 2. **Prior payroll register, year-to-date.** This is the one that matters most — it is the only source that shows each employee's cumulative wages so far this year, which is what the employer tax wage-base cap actually runs off. Without it, every wage-base calculation restarts at zero and overstates employer tax for anyone who should already be capped.
> 3. **Benefit elections** — who is enrolled in what, and the employer-paid share of each.
> 4. **PTO policy** — accrual rate, whether it is per hour worked or a flat rate, any cap on the accrued balance, and the current opening liability by employee or in total.
> 5. **Bonus plan** — the pool or formula, the accrual cadence, and the payout date(s).
>
> Send whatever you have and I'll classify it by what's in it rather than what it's called. Where a role is missing I'll tell you what the model has to assume instead, and what to ask the client for.

**Where the roster gives an annual salary rather than a per-period rate, ask how to convert it — do not choose silently.** Dividing evenly by the pay-frequency count assumes every pay period is the same length, which is true for salaried employees on a fixed calendar but not for anyone paid hourly against actual scheduled hours. Confirm which employees are salaried (divide evenly across pay periods, never across months) and which are hourly (rate × scheduled or forecast hours), and record the answer in the notes block.

**Where no year-to-date payroll register is offered, ask for one before building.** Without it, the wage-base cap for every employer tax cannot be positioned correctly, and a forecast that assumes every employee starts the year at zero will overstate employer tax cost for anyone already past the cap — often materially for higher-paid roles late in the year.

---

# 2. Classify and reconcile the sources

## Classify by shape, not by name

*Never route on a filename, a tab name, or the system a file came out of. Open every source, read its header row, and classify it by the columns it holds.*

Five roles matter. One upload may fill more than one; one role may be split across several uploads; a role may be absent entirely. Classify what is there, name what is missing, and never assume a role is filled because a file exists.

| Role | Recognise it by | Notes |
|---|---|---|
| **Roster** | One row per employee — name or ID, role, pay type, rate, FTE/scheduled hours, start/end date. | May also carry department, location, and manager — useful for pooling the tail by a meaningful grouping rather than a single blended bucket. |
| **Prior payroll register, YTD** | Closed pay periods carrying gross wages paid, by employee, cumulative through the most recent completed period. | The only role carrying *cumulative* wage data, at true pay-period grain. Where present, the wage-base cap positions correctly for every employee. Where absent, every employer tax figure in the early forecast periods is an assumption — say so in the notes. |
| **Benefit elections** | Employee, plan, employee/employer split, effective date. | Distinguish employer-paid share from employee-paid share; only the employer share is a cash disbursement in this module. |
| **PTO policy** | Accrual rate (often hours-per-hour-worked or a flat annual grant), any balance cap, opening liability. | Opening liability is often a single total rather than by employee — usable at that level of granularity if that is all that exists; say so. |
| **Bonus plan** | Pool size or formula, accrual cadence, payout date(s). | Distinguish an accrual (a liability building over the year) from a discretionary one-time payment with no prior accrual — the mechanic differs (see Section 4). |

## Derive each assumption from the best available source

Work down each row until a source exists. Take the first one that does, and record which one it was.

| Assumption | Derive from | Fall back to | Last resort |
|---|---|---|---|
| Gross wages per employee per pay period | Roster (rate × FTE/hours) | Prior payroll register, most recent period annualised | Modeler input — the model cannot be built without this |
| Cumulative YTD wages (for wage-base positioning) | Prior payroll register | Roster rate × pay periods elapsed since fiscal year start | Assume zero, flagged explicitly as understating employer tax for anyone likely already capped |
| Employer tax rates and wage-base thresholds | Current statutory tables for the named jurisdiction(s) | Prior payroll register's employer tax line, back-solved | Modeler input, named as placeholder |
| Benefit employer cost per employee | Benefit elections | Prior payroll register's benefit deduction lines | Modeler input |
| PTO accrual rate and opening liability | PTO policy document | Prior payroll register's PTO liability line, if carried | Modeler input, named as placeholder |
| Bonus pool and cadence | Bonus plan document | Prior year's actual payout, grown by a stated assumption | Modeler input |
| Pay frequency, anchor pay date, pay date lag | Modeler input in every case (Phase 1, items 4–6) | — | — |

Foundation's three rules govern this table: a derived figure outranks a supplied one, every assumption carries its provenance into the notes block, and a fallback is not a substitute.

## Reconcile before building

Run these before a single forecast cell is written, per Foundation.

- **Roster headcount (or FTE total) ties to the prior payroll register's most recent period headcount.** A gap means the roster includes open requisitions not yet on payroll, terminated employees not yet removed, or a scope mismatch — resolve which before building.
- **Cumulative YTD wages by employee, summed, ties to the prior payroll register's total YTD payroll expense.** These are the same number computed two ways; a gap means one file is filtered by department, entity, or pay type that the other is not.
- **Benefit elections carry no employee absent from the roster**, and no active roster employee is missing an expected election where enrollment is meant to be universal.
- **Units and scale.** Payroll systems are typically whole currency units; management reporting may be thousands. Establish the scale of every source explicitly and convert once, on the way in.

**Where a test fails, follow Foundation's protocol** — stop, put the three options to the modeler using the script there, and record the outcome. Do not resolve a break silently.

---

# 3. Choose the roster method and set the pay-date lag

*The roster method is a fork; the pay-date lag is an input, not a branch — the monthly output always carries both an Accrued and a Disbursed line (see Section 4), regardless of the stated lag. Put the roster-method fork to the modeler and wait. Do not choose on their behalf.*

## Fork — named employees vs. pooled roles

A full named-employee build carries every individual's rate, YTD wages, and elections — the most accurate wage-base positioning, at the cost of a roster tab that grows with headcount. A role-pooled build carries one blended rate and one blended wage-base position per role or department — faster to build and easier to present, at the cost of understating the wage-base cap's effect for anyone materially above or below the pooled average.

> "I can build this two ways. Named-employee carries every person's own rate and year-to-date wages, so the wage-base cap lands correctly for each individual — most accurate, especially if pay varies a lot within a role. Pooled-by-role blends everyone in a role into one rate and one cumulative-wage position — faster to build and read, but it smooths out anyone whose wage-base cap timing differs from the pool average. Which do you want? Above your materiality threshold I'd name individuals either way; this choice is really about how the pool below that threshold gets built."

| Answer | Build | Note |
|---|---|---|
| Named employees, **or** no preference and roster is small enough to name in full | **Method A** | Every roster row carries its own rate and cumulative wages |
| Pooled by role, **or** roster is large and only a few roles are individually material | **Method B** | Named individuals above the materiality threshold still get their own row; everyone else pools by role |

**Why the pay-date lag never collapses the two rows.** If this schedule reported at the pay period's own frequency, a zero lag would make the Accrued and Disbursed lines identical in every single period, and collapsing them into one row would avoid pure redundancy. At monthly grain that reasoning doesn't hold even with a modest lag: most months will still show Accrued = Disbursed, but any month where a pay period's lag pushes its pay date across a month boundary will show a real difference — and that's exactly the information a collapsed row would hide. **Always build both rows**, regardless of the stated lag.

---

# 4. Build

Build in this order. **The order is not arbitrary — each step consumes something the step before it produces, and the pay-period ledger has to exist before anything can be rolled up to it.**

1. **`Macro_Assumptions` globals** — entity name, units, monthly forecast start date, monthly period count.
2. **The monthly header spine** — reads start date and period count from step 1.
3. **`Payroll_Assumptions`** — rates, wage-base thresholds, benefit and PTO and bonus parameters, pay frequency, anchor pay date, and lag — needs the source work from Phase 2.
4. **The Roster tab** — named employees and, under Method B, the pooled tail.
5. **`Pay_Period_Ledger`** — one row per actual pay date across the horizon, per employee or pool: the wage-base tracker, gross wages, employer taxes, benefits, and the PTO and bonus roll-forwards, all at true pay-period grain.
6. **The monthly rollup on `Forecast`** — aggregates the ledger into the Accrued and Disbursed lines, and pulls month-end snapshots of the PTO and bonus liabilities.

Steps 1–3 are unblocked by source data other than the pay-frequency interview answers. **Build them while waiting for extracts** rather than holding the whole model until the files arrive.

## Format and cell conventions

Per Foundation, and consistent with the other schedule-builder skills in this plugin — repeated here for completeness rather than as a variant.

- Format the full model in **Roboto**. Forecast labels, dates, values and cell contents are all Roboto 11.
- **B2** — company name, Roboto 14 bold, linked to `Macro_Assumptions` (green font). **B3** — forecast title built from the period count (`="`&periods&`-month payroll forecast"`). **B4** — reporting units, linked to `Macro_Assumptions`.
- Row height 15.00 throughout unless specified otherwise. Labels in column B, running across C–E as needed. **On `Forecast`, column F is the opening column; forecast periods begin in column G. `Pay_Period_Ledger` has no opening column — it is a row-based table, not a period spine (see below).**

| Cell type | Font | Fill | Border |
|---|---|---|---|
| Input | Blue `#0000FF` | Light blue `#EEF5FC` | Hair-weight bottom |
| Formula | Black | None | None |
| Link to another tab in the same workbook | Green `#008000` | None | None |
| Link to an external file | Purple, per Foundation | None | None |

- **Never use yellow fill to identify inputs** — reserved for flagged cells per Foundation.
- **No annotation or comment text in column E.** All notes belong in the "Notes and sources" block on `Payroll_Assumptions`.
- **Tab order and naming follow Foundation.** Underscores instead of spaces; no numbering. For this model:

```
Cover               — title, purpose, Change Log, version
Macro_Assumptions   — this schedule's own copy
Checks              — master ALL CLEAR / ERROR flag and PASS/FAIL rows
Forecast            — the monthly header spine, the rolled-up disbursement lines, memo rows
Payroll_Assumptions — rates, wage bases, benefit/PTO/bonus parameters, pay frequency, anchor date, lag
Roster_Detail       — per-employee rate, YTD wages, elections — Method A, or named employees + pooled tail under Method B
Pay_Period_Ledger   — one row per pay date, per employee or pool — the fine-grained calculation engine
Source_Data         — any remaining extracts
```

## The monthly header spine, on `Forecast`

Identical in construction to the monthly A/R and A/P modules' spine — build the opening column first, per Foundation. Column F carries Month Number `0`, Month Beginning `= EDATE(forecast start date, -1)`, Projection Period - Date `= the Month Beginning cell above`. Forecast periods begin in column G.

| Row | Contents | Fill | Font |
|---|---|---|---|
| Month Number | `0` in the opening column, then prior cell + 1, running to the period count on `Macro_Assumptions` | `#275317` | White `#FFFFFF` |
| Actual/Forecast | `Fcst` in each column | `#3C7D22` | White `#FFFFFF` |
| Month Beginning | The monthly start date, via `EDATE` off the forecast start date — never a fixed day-count offset | `#DAF2D0` | Black `#000000` |

**Twelve months is a common default, not a constant.** Size off the `Macro_Assumptions` period count, per Foundation, exactly as the monthly A/R and A/P modules do.

## `Pay_Period_Ledger` — the fine-grained calculation engine

**This is not a period spine in Foundation's sense, and doesn't take an opening column.** It's a row-based table: one row per actual pay date, per employee or pool, running from the anchor pay date across however many pay periods the forecast horizon requires. Foundation's opening-column convention exists to give a roll-forward a clean first-period reference; this ledger gets that instead from the prior payroll register's cumulative YTD wages, entered directly as each employee's or pool's starting cumulative-wages figure on the first ledger row that falls after the register's as-of date.

**Row structure**, one row per (employee-or-pool, pay date):

`Employee/Pool ID | Pay Period Number | Period Start | Period End | Pay Date | Accrual Month | Disbursement Month | Gross Wages | Cumulative YTD Wages | Employer Tax (capped) | Employer Tax (uncapped) | Employer Benefit Cost | 401(k) Match | PTO Accrued | PTO Used | PTO Liability End. Bal. | Bonus Accrued | Bonus Paid | Bonus Liability End. Bal. | Last Period of Month? (Accrual) | Last Period of Month? (Disbursement)`

**Sort order is not incidental — it's what makes "prior row" and "next row" mean anything.** Every roll-forward below (Cumulative YTD Wages, PTO Liability, Bonus Liability) and both "Last Period of Month?" flags reference the prior or next row for **the same employee or pool**. That reference is only correct if the table is grouped by Employee/Pool ID first, with each employee's or pool's own rows sorted chronologically by Pay Period Number within that group — never interleaved by date across employees. Build the ledger in that grouped order from the start; do not sort it by date first and rely on a lookup to find each employee's neighbouring period, since every stateful formula below is written as a simple prior-row / next-row reference, not a conditional lookup.

**Generating the pay dates.** Every pay date is the anchor pay date (Phase 1, item 5) plus a whole number of pay-frequency intervals — 7 days for weekly, 14 for bi-weekly; semi-monthly and monthly follow the calendar (the 1st and 15th, or one fixed day) rather than a fixed day count. Generate forward and backward from the anchor as needed so the ledger's first row is at or before `Macro_Assumptions`' forecast start date and the last row covers the full horizon.

- **In Claude for Excel**, drive the row count with `SEQUENCE`, sized off a formula computing periods-needed from the monthly horizon and the stated frequency (e.g., horizon months × ~4.34 for weekly, rounded up with headroom) — the exact count matters less than covering the full horizon, since any ledger rows beyond it are simply unused. Generate this per employee/pool (each gets its own contiguous block of dated rows, per the sort-order requirement above), not as one date sequence broadcast across every employee. Cumulative YTD Wages, and both liability roll-forwards, are stateful and are legitimate `SCAN`/`REDUCE` candidates down the row axis, **reset at each employee/pool's first row** — pair each with a plain-English note describing what accumulates and what resets it. For Cumulative YTD Wages specifically, there are two reset conditions: a new employee/pool's first row, and a calendar-year boundary within one employee's own rows (see below) — both are always present in the formula, not special cases for horizons that happen to trigger them.
- **In Claude.ai chat or Cowork**, build a fixed number of rows per employee/pool, sized generously for the maximum expected horizon, with the same formula copied down every row within each employee/pool's block. Add a check (Section 5) confirming every pay date needed by the horizon actually has a row — an under-sized ledger fails silently otherwise.

**Accrual Month and Disbursement Month, and the whole-period convention.** `Accrual Month = the calendar month containing Period End`; `Disbursement Month = the calendar month containing Pay Date`. **A pay period is never split across two months** — even where its Period Start and Period End fall in different calendar months, the whole period's cost accrues to the single month containing its end date. This is a deliberate simplification, not an oversight: prorating a pay period's cost across a month boundary adds real complexity for a correction that's almost always immaterial at the pay-period level, and it keeps every dollar in the ledger traceable to exactly one accrual month and one disbursement month. State this convention in the notes block.

**The wage-base tracker**, per pay period rather than per calendar month — this is the mechanism that keeps the cap's real crossing point intact:

```
Cumulative YTD Wages (per employee or pool, per pay period)
  = IF(this is the employee/pool's first ledger row,
       seeded opening YTD wages from the payroll register,
       IF(YEAR(this pay date) <> YEAR(prior pay period's pay date), 0, prior period's Cumulative YTD Wages))
    + this period's Gross Wages

Employer Tax, Capped
  = MIN(this period's Gross Wages, MAX(0, Wage Base Threshold − prior period's Cumulative YTD Wages)) × tax rate
```

**Two resets guard this formula, not one.** The year-boundary reset (`IF(YEAR(...)<>YEAR(...), 0, ...)`) is not optional: wage-base caps reset every January 1st, and a forecast horizon of 12 months starting anywhere other than January 1st spans a year boundary — without it the ledger would keep accumulating a single employee's wages across the boundary indefinitely, understating employer tax for the entire second year. **The first-row reset is equally load-bearing, for a different reason**: because the ledger is grouped by employee/pool (see Row structure, above), the row immediately above a new employee's first row belongs to the *previous* employee — referencing it as "prior period's Cumulative YTD Wages" would silently carry one employee's cumulative wages into another's. This is the pay-period-grain equivalent of the opening-column convention twice over: it removes both a first-period-of-the-year exception and a first-row-of-a-new-employee exception the same way Foundation's opening column removes a first-forecast-period exception.

**Where the horizon spans more than one calendar year, the wage-base threshold and tax rate themselves may also change year to year** — most statutory wage bases are indexed annually. Look each pay period's threshold and rate up against the calendar year it falls in, from a `Payroll_Assumptions` table carrying one row per tax per year, rather than a single static figure assumed to hold for the whole horizon.

This is the same wage-base-cap formula that applies at any periodicity — "period" here means the true pay period, tracked down `Pay_Period_Ledger` rows rather than across `Forecast` columns. **Employer Medicare-equivalent tax (uncapped in most US jurisdictions) is simply gross wages × rate, with no wage-base term** — do not apply the capped formula to an uncapped tax.

**Gross wages:**

```
Gross Wages (per employee or pool, per pay period)
  Salaried: annual rate / pay periods per year
  Hourly: rate × scheduled or forecast hours for the period
```

**PTO and bonus roll-forwards** run down the ledger's rows, per pay period — additions positive, reductions negative, ending balance a single `SUM` down the block, capped at accrual where the PTO policy caps the balance:

```
  PTO Liability, Beg. Bal. (prior pay period's End. Bal.)
+ PTO Accrued (accrual rate × hours worked, or a flat per-period grant, per PTO policy — capped at accrual: MIN(prior + accrual, cap))
− PTO Used / Paid Out
  PTO Liability, End. Bal.
```

Bonus follows the identical shape where the bonus plan is a genuine accrual; where it's a one-time discretionary payment with no prior accrual, skip the roll-forward and place it as a single dated line on the pay period nearest its payout date instead.

**The "Last Period of Month?" flags.** For each of Accrual Month and Disbursement Month, flag the row `TRUE` when this row's month differs from the next row's month for that same employee/pool, or when it's that employee/pool's final ledger row. These flags are what let the monthly rollup pull a clean month-end snapshot of a cumulative balance without needing a lookup any more complex than a `SUMIFS`.

## The rollup, on `Forecast`

```
Payroll Expense Accrued (this month)
  = SUMIFS(Pay_Period_Ledger's [Gross Wages + Employer Tax, capped + uncapped + Employer Benefit Cost + 401(k) Match],
           Pay_Period_Ledger's Accrual Month, this month)

Payroll Cash Disbursed (this month)
  = SUMIFS(the same components, Pay_Period_Ledger's Disbursement Month, this month)

PTO Liability, End. Bal. (this month)
  = SUMIFS(Pay_Period_Ledger's PTO Liability End. Bal., Accrual Month, this month, Last Period of Month? (Accrual), TRUE)

Bonus Liability, End. Bal. (this month)
  = SUMIFS(Pay_Period_Ledger's Bonus Liability End. Bal., Accrual Month, this month, Last Period of Month? (Accrual), TRUE)
```

Because `SUMIFS` naturally picks up however many pay periods actually fall in a given month, a five-payday month sums five periods' worth and a four-payday month sums four — the variation shows up correctly with no separate mechanic required.

**Anchor the opening liability balances in `Forecast`'s opening column**, per Foundation, as green links to the ledger's own opening figures (the prior payroll register's opening PTO and bonus liabilities) — the same convention every other roll-forward in this plugin uses, even though the ledger itself, as a row-based table, doesn't take one.

---

# 5. Checks

**These live on the `Checks` tab**, per Foundation: a master ALL CLEAR / ERROR cell that reads clear only when every row below it passes, and one clearly-labelled PASS/FAIL row per check with conditional formatting. Do not scatter check cells through the forecast sheet.

Foundation caps this at a handful of checks that catch real breakage. The set below is deliberately short; anything more exhaustive is `financial-modeling-auditor` territory.

- **`Pay_Period_Ledger` covers the full horizon:** the ledger's last row's Pay Date is on or after `Macro_Assumptions`' final forecast month's end. A ledger that runs out before the horizon does understates every later month silently.
- **The rollup ties to the ledger, exactly:** total Payroll Expense Accrued across every `Forecast` month equals the sum of every ledger row's accrual components; same test for Payroll Cash Disbursed against Disbursement Month. A mismatch means a pay period is being dropped or double-counted in the `SUMIFS`.
- **The wage-base cap never taxes beyond the threshold**, checked at pay-period grain on `Pay_Period_Ledger`: cumulative capped-tax base for any employee or pool never exceeds the stated wage-base threshold for that tax.
- **Cumulative YTD Wages resets to zero at the first pay period of each new calendar year, for every employee or pool** — test this explicitly wherever the horizon spans a year boundary; a non-zero carry-forward past December 31st means the reset formula was altered or miscopied.
- **Cumulative YTD Wages never carries across an employee/pool boundary:** the first ledger row for every employee or pool equals its seeded opening figure plus that row's own gross wages — never a value that could only have come from the previous employee's rows. Spot-check the row immediately following any employee/pool change.
- **`Pay_Period_Ledger` is grouped by Employee/Pool ID with each employee's or pool's rows sorted chronologically** — not interleaved by date across employees. Every stateful roll-forward on the ledger depends on this; a date-sorted or otherwise re-ordered ledger silently breaks every prior-row and next-row reference in the model.
- **PTO Liability and Bonus Liability roll forward cleanly on `Pay_Period_Ledger`:** opening + accrued − used = closing, every pay period, with no negative balance — and the monthly snapshots on `Forecast` tie exactly to the ledger's own "Last Period of Month" rows.
- **Every ledger row has exactly one Accrual Month and one Disbursement Month, both inside the model horizon or explicitly flagged beyond it** — none blank.
- Named employees plus the pooled tail (Method B) sum to total roster headcount cost, on the ledger.

## Diagnostics that are not checks

These are analytical outputs, not pass/fail conditions. **Carry them as memo rows beneath the disbursement lines on `Forecast`**, not on the `Checks` tab:

- **Pay periods per month** — a simple count from the ledger, shown explicitly so the four-vs-five-payday effect (or bi-weekly's occasional third period) is visible rather than assumed away.
- Effective blended employer tax rate (total employer tax ÷ total gross wages), period by period on the ledger — should visibly decline over the year as more of the roster crosses the wage-base cap.
- Average fully-loaded cost per FTE (gross wages + employer taxes + benefits + match, divided by headcount).
- PTO liability expressed in weeks of average payroll, as a quick sense of exposure size.

## Reconciliation outcomes are documentation, not checks

Whether each source reconciliation passed, failed, or was carried as a disclosed reconciling item belongs in the **"Notes and sources" block**, with the modeler's recorded decision. It is a statement about the inputs rather than a test of the model, and `financial-modeling-auditor` reads it there.
