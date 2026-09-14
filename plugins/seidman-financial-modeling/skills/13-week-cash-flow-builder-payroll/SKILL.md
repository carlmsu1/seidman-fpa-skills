---
name: 13-week-cash-flow-builder-payroll
description: "Builds the payroll module of a board-room and PE-diligence grade 13-week direct-method cash flow forecast in Excel — a bottom-up, roster-driven build covering gross wages, employer payroll taxes with wage-base caps tracked off cumulative YTD wages, benefits, PTO liability roll-forward, and bonus accrual and payout. Interviews the modeler for every required input, classifies source files by the columns they carry rather than their names, and separates the pay-period the cost is accrued in from the pay date the cash actually moves. Use whenever the user asks to build a weekly cash flow, a 13-week cash flow forecast, a rolling cash flow model, a short-term liquidity forecast, a payroll forecast, a headcount cost forecast, or wants to forecast payroll cash disbursements on a weekly basis. This skill REQUIRES financial-modeling-foundation — always load and follow it first, and ask the user to enable it if it is not available."
---

# 13-Week Cash Flow Forecast — Payroll Module

**Requires `financial-modeling-foundation`.** Load and follow it first; everything below is *additive* to it, never a replacement. Where this skill appears to contradict Foundation, Foundation wins — tell the modeler which passage conflicts rather than choosing silently. If Foundation is not available, say so and ask the modeler to enable it before building.

**Determine the build surface before anything else.** Foundation makes this non-optional, and it changes the technique for every period-driven row in this model:

- **Claude for Excel** (live Excel 365 workbook) — the full dynamic-array toolkit is available. The header spine, the cumulative wage-base tracker, and the PTO and bonus accrual roll-forwards are built as spilling formulas.
- **Claude.ai chat or Cowork** (file delivered as a download, LibreOffice recalculation in the pipeline) — no spilling arrays. One formula copied identically across every period column.

Ask if it is not obvious. Building for the wrong surface can silently ship a broken file.

Builds the payroll half of a weekly direct-method cash flow forecast in Excel. Payroll only — trade payables and non-payroll disbursements are the A/P module's job and are not forecast here.

This module shares the header and footer spine with the A/R and A/P modules. Where any of those is being built alongside this one, build the spine once and let all of them read off it.

**The asymmetry that governs everything below:** A/R forecasts behaviour and A/P forecasts policy, but payroll forecasts a **roster** — once the roster, rates, and elections are set, most of the arithmetic is deterministic rather than assumed. The two places judgment still enters are the **pay-date lag** (the wage period a cost is earned in is rarely the week the cash actually moves) and the **wage-base cap** (employer tax stops accruing once an employee's cumulative year-to-date wages cross a statutory threshold, so the same gross-wage dollar can cost more early in the year than late in it). Everything below exists to make those two facts visible rather than smoothed away.

**Build in this order.**

1. Interview the modeler — two phases
2. Classify and reconcile the sources
3. Choose the roster method and the pay-date convention
4. Build the spine, then the model
5. Build the Checks tab

---

# 1. Interview the modeler

## Phase 1 — Independent facts

Ask these before looking at any file, as a single block. Where the A/R or A/P module already exists for this company, inherit items 1–3 rather than asking twice.

> Before I look at any data, seven things about the model itself:
> 1. **Entity name**, as it should read on the roll forward.
> 2. **Week 1 start date.** The model runs Monday-to-Sunday — which Monday is Week 1?
> 3. **Reporting units** — whole dollars, thousands, or millions?
> 4. **Pay frequency and pay date lag.** Weekly, bi-weekly, semi-monthly, or monthly — and how many days after the period ends does the pay date actually fall? This is the single most consequential answer in this interview: it decides which week the cash leaves, not just which week the cost is earned.
> 5. **Materiality for named employees.** I default to naming anyone above a fixed dollar threshold you set, or anyone whose role makes their cost worth tracking individually (owners, executives, commissioned roles), and pooling the rest by role or department. Different threshold, or a different basis than dollars?
> 6. **Employer tax jurisdiction(s).** Which state(s) or country determines the FUTA/SUTA (or local equivalent) rates and wage-base thresholds that apply? A multi-state roster needs a rate and a cap per jurisdiction, not one blended figure.
> 7. **Benefits, PTO, and bonus in scope.** Does the forecast need to carry employer-paid benefits (health, 401(k) match), a PTO liability roll-forward, and bonus accrual/payout — or is base wages and statutory tax enough for this engagement?

If the modeler cannot answer 2, stop. Everything in the model is dated off the Week 1 start. Every other gap is recoverable with a named placeholder.

## Phase 2 — Sources and methodology

Ask only after Phase 1 is answered.

> Now the data. Five kinds of file matter to this model, and one upload often fills more than one of them:
> 1. **Roster / HRIS export** — one row per employee (or open requisition), with role, pay type (salary or hourly), rate, FTE or scheduled hours, and start/end dates.
> 2. **Prior payroll register, year-to-date.** This is the one that matters most — it is the only source that shows each employee's cumulative wages so far this year, which is what the employer tax wage-base cap actually runs off. Without it, every wage-base calculation restarts at zero and overstates employer tax for anyone who should already be capped.
> 3. **Benefit elections** — who is enrolled in what, and the employer-paid share of each.
> 4. **PTO policy** — accrual rate, whether it is per hour worked or a flat rate, any cap on the accrued balance, and the current opening liability by employee or in total.
> 5. **Bonus plan** — the pool or formula, the accrual cadence, and the payout date(s).
>
> Send whatever you have and I'll classify it by what's in it rather than what it's called. Where a role is missing I'll tell you what the model has to assume instead, and what to ask the client for.

**Where the roster gives an annual salary rather than a per-period rate, ask how to convert it — do not choose silently.** Dividing evenly by the pay-frequency count assumes every pay period is the same length, which is true for salaried employees on a fixed calendar but not for anyone paid hourly against actual scheduled hours. Confirm which employees are salaried (divide evenly) and which are hourly (rate × scheduled or forecast hours), and record the answer in the notes block.

**Where no year-to-date payroll register is offered, ask for one before building.** Without it, the wage-base cap for every employer tax cannot be positioned correctly, and a forecast that assumes every employee starts the year at zero will overstate employer tax cost for anyone already past the cap — often materially for higher-paid roles late in the year.

---

# 2. Classify and reconcile the sources

## Classify by shape, not by name

*Never route on a filename, a tab name, or the system a file came out of. Open every source, read its header row, and classify it by the columns it holds.*

Five roles matter. One upload may fill more than one; one role may be split across several uploads; a role may be absent entirely. Classify what is there, name what is missing, and never assume a role is filled because a file exists.

| Role | Recognise it by | Notes |
|---|---|---|
| **Roster** | One row per employee — name or ID, role, pay type, rate, FTE/scheduled hours, start/end date. | May also carry department, location, and manager — useful for pooling the tail by a meaningful grouping rather than a single blended bucket. |
| **Prior payroll register, YTD** | Closed pay periods carrying gross wages paid, by employee, cumulative through the most recent completed period. | The only role carrying *cumulative* wage data. Where present, the wage-base cap positions correctly for every employee. Where absent, every employer tax figure in the early forecast weeks is an assumption — say so in the notes. |
| **Benefit elections** | Employee, plan, employee/employer split, effective date. | Distinguish employer-paid share from employee-paid share; only the employer share is a cash disbursement in this module. |
| **PTO policy** | Accrual rate (often hours-per-hour-worked or a flat annual grant), any balance cap, opening liability. | Opening liability is often a single total rather than by employee — usable at that level of granularity if that is all that exists; say so. |
| **Bonus plan** | Pool size or formula, accrual cadence, payout date(s). | Distinguish an accrual (a liability building over the year) from a discretionary one-time payment with no prior accrual — the mechanic differs (see Section 4). |

## Derive each assumption from the best available source

Work down each row until a source exists. Take the first one that does, and record which one it was.

| Assumption | Derive from | Fall back to | Last resort |
|---|---|---|---|
| Gross wages per employee per period | Roster (rate × FTE/hours) | Prior payroll register, most recent period annualised | Modeler input — the model cannot be built without this |
| Cumulative YTD wages (for wage-base positioning) | Prior payroll register | Roster rate × pay periods elapsed since fiscal year start | Assume zero, flagged explicitly as understating employer tax for anyone likely already capped |
| Employer tax rates and wage-base thresholds | Current statutory tables for the named jurisdiction(s) | Prior payroll register's employer tax line, back-solved | Modeler input, named as placeholder |
| Benefit employer cost per employee | Benefit elections | Prior payroll register's benefit deduction lines | Modeler input |
| PTO accrual rate and opening liability | PTO policy document | Prior payroll register's PTO liability line, if carried | Modeler input, named as placeholder |
| Bonus pool and cadence | Bonus plan document | Prior year's actual payout, grown by a stated assumption | Modeler input |
| Pay date lag | Modeler input in every case (Phase 1, item 4) | — | — |

Foundation's three rules govern this table: a derived figure outranks a supplied one, every assumption carries its provenance into the notes block, and a fallback is not a substitute.

## Reconcile before building

Run these before a single forecast cell is written, per Foundation.

- **Roster headcount (or FTE total) ties to the prior payroll register's most recent period headcount.** A gap means the roster includes open requisitions not yet on payroll, terminated employees not yet removed, or a scope mismatch — resolve which before building.
- **Cumulative YTD wages by employee, summed, ties to the prior payroll register's total YTD payroll expense.** These are the same number computed two ways; a gap means one file is filtered by department, entity, or pay type that the other is not.
- **Benefit elections carry no employee absent from the roster**, and no active roster employee is missing an expected election where enrollment is meant to be universal.
- **Units and scale.** Payroll systems are typically whole currency units; management reporting may be thousands. Establish the scale of every source explicitly and convert once, on the way in.

**Where a test fails, follow Foundation's protocol** — stop, put the three options to the modeler using the script there, and record the outcome. Do not resolve a break silently.

---

# 3. Choose the roster method and the pay-date convention

*Both are forks in the model, not styling preferences. Put both to the modeler and wait. Do not choose on their behalf.*

## Fork 1 — named employees vs. pooled roles

A full named-employee build carries every individual's rate, YTD wages, and elections — the most accurate wage-base positioning, at the cost of a roster tab that grows with headcount. A role-pooled build carries one blended rate and one blended wage-base position per role or department — faster to build and easier to present, at the cost of understating the wage-base cap's effect for anyone materially above or below the pooled average.

> "I can build this two ways. Named-employee carries every person's own rate and year-to-date wages, so the wage-base cap lands correctly for each individual — most accurate, especially if pay varies a lot within a role. Pooled-by-role blends everyone in a role into one rate and one cumulative-wage position — faster to build and read, but it smooths out anyone whose wage-base cap timing differs from the pool average. Which do you want? Above your materiality threshold I'd name individuals either way; this choice is really about how the pool below that threshold gets built."

| Answer | Build | Note |
|---|---|---|
| Named employees, **or** no preference and roster is small enough to name in full | **Method A** | Every roster row carries its own rate and cumulative wages |
| Pooled by role, **or** roster is large and only a few roles are individually material | **Method B** | Named individuals above the materiality threshold still get their own row; everyone else pools by role |

## Fork 2 — pay-date convention

> "Payroll cost is earned in the week worked but the cash moves on the pay date, which usually lags the period end by a fixed number of days. Do the pay dates for this roster fall inside the same week they're earned, or do they lag into the following week (or further)? If they lag, I'll build two rows — cost accrued in the week earned, and cash disbursed in the week actually paid — so the two are never confused."

| Answer | Build |
|---|---|
| Pay date falls within the week earned | Single row: cost and cash disbursement in the same week |
| Pay date lags into a later week | Two rows: **Payroll Expense Accrued** (the week earned) and **Payroll Cash Disbursed** (the pay date, offset by the stated lag) — only the disbursement row feeds the combined cash roll-forward |

State both fork answers in the notes block. A reader must never have to infer from the formulas which fork the model took.

---

# 4. Build

Build in this order. **The order is not arbitrary — each step consumes something the step before it produces.**

1. **`Macro_Assumptions` globals** — entity name, units, forecast start date, period count. Shared with the A/R and A/P modules where they coexist; build once.
2. **The spine** — reads the start date and period count from step 1. Nothing else in the model can be built correctly before it.
3. **`Payroll_Assumptions`** — rates, wage-base thresholds, benefit and PTO and bonus parameters, which need the source work from Phase 2.
4. **The Roster tab** — named employees and, under Method B, the pooled tail.
5. **The wage-base tracker, then gross wages, then employer taxes.**
6. **The PTO and bonus accrual roll-forwards.**
7. **The consolidated Payroll Disbursement line** that feeds the combined cash roll-forward.

Steps 1–2 are unblocked by source data. **Build them while waiting for extracts** rather than holding the whole model until the files arrive.

## Format and cell conventions

Identical to the A/R and A/P modules, per Foundation — repeated here for completeness rather than as a variant.

- Format the full model in **Roboto**. Forecast labels, dates, values and cell contents are all Roboto 11.
- **B2** — company name, Roboto 14 bold, linked to `Macro_Assumptions` (green font). **B3** — forecast title built from the period count. **B4** — reporting units, linked to `Macro_Assumptions`.
- Row height 15.00 throughout unless specified otherwise. Labels in column B, running across C–E as needed. **Column F is the opening column; forecast periods begin in column G.**

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
Cover              — title, purpose, Change Log, version
Macro_Assumptions  — shared with A/R and A/P where they coexist
Checks             — master ALL CLEAR / ERROR flag and PASS/FAIL rows
Forecast           — the header spine, the payroll disbursement lines, memo rows
Payroll_Assumptions — rates, wage bases, benefit/PTO/bonus parameters
Roster_Detail      — per-employee rate, YTD wages, elections — Method A, or named employees + pooled tail under Method B
Source_Data        — any remaining extracts
```

## The wage-base tracker

This is the payroll module's equivalent of the A/R and A/P roll-forward — a balance carried forward period to period, per Foundation's roll-forward sign convention (additions positive, so the running total is a single `SUM` down the block).

```
Cumulative YTD Wages (per employee or pool)
  = prior period's Cumulative YTD Wages + current period Gross Wages
```

**Anchor the opening figure in the opening column**, per Foundation: the YTD wages carried in from the prior payroll register go in column F, not Week 1's addition. `Beginning-of-period Cumulative Wages = prior column's Cumulative YTD Wages` then holds in every forecast column including the first.

Employer tax for a wage-base-capped tax (Social Security / OASDI equivalent, FUTA, SUTA) in a given period:

```
Employer Tax, Capped
  = MIN(current period Gross Wages, MAX(0, Wage Base Threshold − prior period's Cumulative YTD Wages)) × tax rate
```

The `MAX(0, ...)` guards against a negative taxable amount once an employee is already past the cap; the outer `MIN` ensures no more than the current period's actual gross wages are taxed even where the remaining headroom under the cap exceeds it. **Employer Medicare-equivalent tax (uncapped in most US jurisdictions) is simply gross wages × rate, with no wage-base term at all** — do not apply the capped formula to an uncapped tax.

## Gross wages, benefits, and the consolidated disbursement line

```
Gross Wages (per employee or pool)
  Salaried: annual rate / pay periods per year
  Hourly: rate × scheduled or forecast hours for the period

Employer Benefit Cost
  = SUMIFS across enrolled employees of each plan's employer-paid share,
    from Payroll_Assumptions benefit elections

401(k) Employer Match
  = MIN(employee deferral × match rate, match cap) per employee, summed

Payroll Expense Accrued (the week earned)
  = Gross Wages + Employer Taxes (capped and uncapped) + Employer Benefit Cost + 401(k) Match

Payroll Cash Disbursed (the pay date — only this row feeds the combined cash roll-forward)
  = Payroll Expense Accrued, offset by the pay-date lag from Fork 2
```

Where Fork 2 resolved to same-week pay dates, these two rows collapse into one and the lag offset is zero.

## PTO liability roll-forward

A genuine roll-forward, in the same shape as the A/R and A/P balance conventions — additions positive, reductions negative, ending balance a single `SUM` down the block.

```
  PTO Liability, Beg. Bal.
+ PTO Accrued (accrual rate × hours worked or a flat per-period grant, per PTO policy)
− PTO Used / Paid Out (from the roster's forecast time-off, or an assumed usage rate where none is supplied)
  PTO Liability, End. Bal.
```

**Anchor the opening liability in the opening column**, per Foundation, exactly as the wage-base tracker does. Where the PTO policy caps the accrued balance, apply the cap at the point of accrual (`MIN(prior balance + current accrual, cap)`), not as a separate adjusting entry — an uncapped accrual followed by a cap adjustment obscures how much accrual was actually forfeited.

## Bonus accrual and payout

Where the bonus plan is a genuine accrual (a liability building toward a known payout date), build it as a roll-forward identical in shape to PTO:

```
  Bonus Liability, Beg. Bal.
+ Bonus Accrued (pool or formula, per period, per the bonus plan)
− Bonus Paid Out (the payout week(s) specified in the bonus plan)
  Bonus Liability, End. Bal.
```

Where the bonus plan is a one-time discretionary payment with no prior accrual, skip the roll-forward and carry it as a single dated disbursement line instead — building an accrual mechanic for a payment that was never accrued manufactures a liability that does not exist.

---

# 5. Checks

**These live on the `Checks` tab**, per Foundation: a master ALL CLEAR / ERROR cell that reads clear only when every row below it passes, and one clearly-labelled PASS/FAIL row per check with conditional formatting. Do not scatter check cells through the forecast sheet.

Foundation caps this at a handful of checks that catch real breakage. The set below is deliberately short; anything more exhaustive is `financial-modeling-auditor` territory.

- **PTO Liability rolls forward cleanly:** opening + accrued − used = closing, every period, with no negative balance.
- **Bonus Liability rolls forward cleanly** (where an accrual mechanic is used), on the same basis.
- **The wage-base cap never taxes beyond the threshold:** cumulative capped-tax base for any employee or pool never exceeds the stated wage-base threshold for that tax.
- **Payroll Expense Accrued and Payroll Cash Disbursed reconcile in total over the full forecast window** — the lag shifts *when* the cash moves, never *how much* moves in total.
- Named employees plus the pooled tail (Method B) sum to total roster headcount cost.
- Gross wages by roster tie to rate × hours/FTE, recomputed independently of the roll-forward for a sample of rows.

## Diagnostics that are not checks

These are analytical outputs, not pass/fail conditions. **Carry them as memo rows beneath the disbursement lines on `Forecast`**, not on the `Checks` tab:

- Effective blended employer tax rate (total employer tax ÷ total gross wages), period by period — should visibly decline over the year as more of the roster crosses the wage-base cap.
- Average fully-loaded cost per FTE (gross wages + employer taxes + benefits + match, divided by headcount).
- PTO liability expressed in weeks of average payroll, as a quick sense of exposure size.

## Reconciliation outcomes are documentation, not checks

Whether each source reconciliation passed, failed, or was carried as a disclosed reconciling item belongs in the **"Notes and sources" block**, with the modeler's recorded decision. It is a statement about the inputs rather than a test of the model, and `financial-modeling-auditor` reads it there.
