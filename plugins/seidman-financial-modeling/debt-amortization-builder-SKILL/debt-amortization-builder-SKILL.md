---
name: debt-amortization-builder
description: "Builds a board-room and PE-diligence grade debt schedule in Excel — tranche-by-tranche roll-forward of term loans (fully amortizing, interest-only, or bullet/balloon), fixed or floating (reference-rate-plus-spread) interest, on a monthly periodicity. Interviews the modeler for day count convention, amortization style, and floating-rate curve source before building. Classifies source files by the columns they carry rather than their names; reconciles the loan agreements against any existing debt schedule before building. Use whenever the user asks to build a debt schedule, a loan amortization schedule, a term loan schedule, a debt roll-forward, or an interest expense forecast for term debt. For a combined model where this feeds a company-wide cash flow, use `master-cash-flow-builder` instead, which orchestrates this skill as one of its modules. This skill REQUIRES financial-modeling-foundation — always load and follow it first, and ask the user to enable it if it is not available."
---

# Debt Amortization Schedule Builder

**Requires `financial-modeling-foundation`.** Load and follow it first; everything below is *additive* to it, never a replacement. Where this skill appears to contradict Foundation, Foundation wins — tell the modeler which passage conflicts rather than choosing silently. If Foundation is not available, say so and ask the modeler to enable it before building.

**Determine the build surface before anything else.** Foundation makes this non-optional, and it changes the technique for the balance roll-forward on every tranche:

- **Claude for Excel** (live Excel 365 workbook) — the full dynamic-array toolkit is available. Use `SCAN`/`REDUCE` with `LAMBDA` for each tranche's stateful balance roll-forward.
- **Claude.ai chat or Cowork** (file delivered as a download, LibreOffice recalculation in the pipeline) — no spilling arrays. One formula copied identically across every period column.

Ask if it is not obvious. Building for the wrong surface can silently ship a broken file.

Builds a monthly, tranche-by-tranche debt schedule: every term loan's scheduled amortization and interest, rolling to a closing balance that ties to the balance sheet. Every hardcoded number lives on an assumptions tab, every policy choice is recorded as a decision rather than assumed, every reconciliation test is recorded whether it passed or failed.

**This schedule covers term loans only** — fully amortizing, interest-only, and bullet/balloon structures. It does not cover revolvers or other draw-and-repay facilities, whose balance is sized off a cash flow forecast rather than a fixed contractual schedule; that's a different mechanical problem and belongs in a separate skill.

**This schedule runs monthly.** That is not a Phase 1 question in this skill the way periodicity is in others — it is fixed, because monthly is the periodicity almost every credit agreement's payment and rate-reset provisions are actually written against.

**Build in this order.** Each phase depends on the one before it, and skipping ahead is the most common way this schedule goes wrong.

1. Interview the modeler — two phases
2. Classify and reconcile the sources
3. Build the term loan tranches
4. Build the Checks tab

---

# 1. Interview the modeler

## Phase 1 — Independent facts

Ask these before looking at any file. They are things the modeler knows without opening a credit agreement, and several of them are policy choices this skill will not guess at. Ask them as a single block, not one at a time.

> Before I look at any data, five things about the schedule itself:
> 1. **Company name**, as it should read on the cover of the schedule.
> 2. **Model start date and horizon** — the first forecast month, and how many months.
> 3. **Reporting units** — whole dollars, thousands, or millions?
> 4. **Day count convention.** How should monthly interest be calculated?
>    - **30/360** — annual rate ÷ 12, flat every month regardless of actual days. Simplest, and what most monthly-pay term loans are actually documented against.
>    - **Actual/360 or Actual/365** — annual rate × (actual days in the month ÷ 360 or 365). More precise, more common on syndicated or bank facilities with actual-day interest provisions.
> 5. **Amortization style**, for any fully-amortizing tranche whose credit agreement doesn't already specify a fixed schedule:
>    - **Level payment** — the total payment (principal + interest) is constant, like a mortgage; principal grows and interest shrinks each month.
>    - **Level principal** — the principal payment is constant each month; the total payment shrinks as the balance runs off.

If the modeler cannot answer 2, stop. Every rate reset and every payment date is dated off the model start. Every other gap is recoverable with a named default, stated as an assumption.

**Where day count convention or amortization style is not answered, do not default silently.** Both are genuine policy choices — the same loan produces a different interest or principal number under each option. Put the tradeoff to the modeler as written above and record the choice in the notes block.

## Phase 2 — Sources and methodology

Ask only after Phase 1 is answered.

> Now the data. Three kinds of file matter to this schedule, and one upload often fills more than one of them:
> 1. **Loan agreements / existing debt schedule** — for each outstanding tranche: lender, original principal, current balance, origination and maturity dates, rate (a fixed rate, or a spread over a named reference rate), payment structure (amortizing, interest-only, or bullet), and its contractual amortization schedule if one is fixed rather than calculated.
> 2. **Planned or committed new financing** — tranches not yet drawn: amount, expected draw date, rate terms, structure.
> 3. **Reference rate assumption** — for any floating-rate tranche: a forward curve, or an instruction to hold the current rate flat, or a modeler-specified path.
>
> Send whatever you have and I'll classify it by what's in it rather than what it's called. Where a role is missing I'll tell you what the schedule has to assume instead, and what to ask for.

**Where a tranche's rate is floating, ask for its reset frequency explicitly if the agreement doesn't state one.** Default to resetting monthly, matching the schedule's own periodicity, and flag that as an assumption — some facilities reset quarterly even though they pay monthly, and that distinction changes the interest number.

---

# 2. Classify and reconcile the sources

## Classify by shape, not by name

*Never route on a filename, a tab name, or the system a file came out of. Open every source, read its header row, and classify it by the columns it holds.*

Three roles matter. One upload may fill more than one; one role may be split across several uploads; a role may be absent entirely.

| Role | Recognise it by | Notes |
|---|---|---|
| **Existing debt schedule / loan agreements** | One row (or one document) per outstanding tranche — an origination date already in the past, a current balance. | Their remaining schedule is what's left of an existing contract, not a new one — the contractual terms govern, not this skill's defaults. |
| **Planned new financing** | Tranches with a draw date **inside or after** the forecast window, typically no current-balance column because the tranche doesn't exist yet. | Rate and structure are sometimes indicative rather than final — flag where that's the case. |
| **Reference rate curve** | A rate by period, or a single current rate with no forward path. | The fallback for a bare current rate is to hold it flat — say so in the notes rather than let a flat line imply a forecast. |

## Derive each assumption from the best available source

Work down each row until a source exists. Take the first one that does, and record which one it was.

| Assumption | Derive from | Fall back to | Last resort |
|---|---|---|---|
| Opening balance, each existing tranche | Existing debt schedule / loan agreement | Trial balance debt total, undifferentiated by tranche | Modeler input — the schedule cannot start without this |
| Scheduled amortization, existing tranches | The agreement's own fixed schedule, where one exists | Recalculated from rate, remaining term, and the amortization style chosen in Phase 1 | Modeler input |
| Rate (fixed value, or spread + index), each tranche | Loan agreement | Modeler input | — |
| Reference rate path, floating tranches | Supplied forward curve | Current rate held flat, flagged as an assumption | Modeler input |

**Reconcile before building.** Sum the existing debt schedule's opening balances and tie them to the balance sheet's total debt line if one is supplied. Record whether the tie passed, failed, or was carried as a disclosed reconciling item in the notes block — this is documentation, not a Checks-tab test (see Section 4).

---

# 3. Build the term loan tranches

## Tab structure

Following Foundation's standard order — Cover/README, Macro_Assumptions, Checks, then model-specific tabs — model start date, horizon, and reporting units from the Phase 1 interview land on **Macro_Assumptions**. This schedule adds two tabs after Checks:

- **Debt_Assumptions** — day count convention, amortization style default, and the reference rate curve. This is the fallback source every gap in `Debt_Schedule` pulls from.
- **Debt_Schedule** — one block of rows per tranche, stacked on a single tab per Foundation's tab-structure default for a compact model. Promote it to a dedicated tab of its own if the tranche count is large enough to be "an abundance of related data" in its own right.

## Format and cell conventions

- Format the full model in **Roboto**. Schedule labels, dates, values and cell contents are all Roboto 11.
- **B2** — company name. Roboto 14, bold. **Link it to `Macro_Assumptions`** (green font) rather than typing it.
- **B3** — the schedule title, built from the period count so it cannot drift: `="`&periods&`-month debt schedule"`. Roboto 11, plain.
- **B4** — the reporting units, written out. **Link to `Macro_Assumptions`** (green font). Foundation defaults to actual dollars where the modeler expresses no preference.
- Row height 15.00 throughout unless specified otherwise. Labels sit in column B and run across C–E as needed. **Column F is the opening column; forecast periods begin in column G.**

This B2:B4 block is the *sheet* header on `Debt_Schedule`, not a substitute for Foundation's Cover tab. Build both — the Cover carries the Change Log and version; this block identifies the sheet when it is printed or exported on its own.

| Cell type | Font | Fill | Border |
|---|---|---|---|
| Input | Blue `#0000FF` | Light blue `#EEF5FC` | Hair-weight bottom |
| Formula | Black | None | None |
| Link to another tab in the same workbook | Green `#008000` | None | None |
| Link to an external file | Purple, per Foundation | None | None |

- **Never use yellow fill to identify inputs.** The blue input convention alone identifies what the reader must replace. Per Foundation, yellow fill is reserved for *flagged* cells only.
- **No annotation or comment text in column E**, or any column adjacent to the label or value columns. All notes belong in the "Notes and sources" block on `Debt_Assumptions`.
- Zoom 80% and gridlines off on every sheet. Size column widths so no label is truncated. Freeze panes immediately below the header spine and to the left of the first forecast column.

## Row structure — one block per tranche

Every term loan tranche, existing or newly financed, gets the same row shape:

`Tranche ID | Lender/Description | Rate Type (Fixed/Floating) | Fixed Rate or Spread | Reference Rate Index | Payment Structure (Amortizing/Interest-Only/Bullet) | Amortization Style | Maturity Date | Opening Balance | Draws | Scheduled Principal | Prepayments | Closing Balance` — then, as its own row directly beneath: **Interest Expense**, one column per period.

Interest is deliberately **not** a column inside the balance roll-forward. It doesn't reduce the principal balance the way a payment does (this skill assumes cash-pay interest throughout — flag it explicitly if any tranche is PIK, since that does add to principal and changes the roll-forward), so it lives as its own row beneath the roll-forward, the same way depreciation sits beneath a capex roll-forward.

**Existing tranches** carry their opening balance from the loan agreement in the model's opening column (per Foundation's opening-column convention — this is a roll-forward, so it gets one). **New financing** carries a blank opening balance; its schedule doesn't start until its draw date falls inside the forecast window, at which point the draw amount becomes its opening balance for the following period.

**`Prepayments`, unlike `Scheduled Principal`, is never a calculated row.** Whether voluntary or a mandatory excess-cash-flow sweep, this schedule doesn't determine the amount or timing — the modeler enters it directly on the tranche's row, one hardcoded (blue) figure per period it occurs, sourced from the financing plan or the sweep calculation wherever that lives outside this schedule. Its only job here is to roll into the balance correctly, which the `MIN(Opening Balance, Scheduled Principal)` guard and the zero-after-payoff rule below already cover.

## Principal mechanics, by payment structure

**Bullet / balloon:** Scheduled Principal is zero in every period except the maturity month, where it equals the full remaining balance. Add a check (Section 4) that confirms this — a bullet tranche that doesn't zero out at maturity is a broken formula, not a valid outcome.

**Interest-only:** Scheduled Principal is zero for the interest-only period, then converts to one of the two styles below for any remaining term after the interest-only period ends. Where the agreement doesn't state a post-IO structure, ask — don't assume it becomes a bullet.

**Fully amortizing — level payment:**

```
Payment
  = PMT(rate at amortization start, total amortizing periods, −Balance at amortization start)

Scheduled Principal (this period)
  = Payment − (Opening Balance × period rate)
```

**Calculate `Payment` once, from the tranche's original terms, and hold it fixed across every period** — never recompute it from the current, declining opening balance. A voluntary prepayment reduces the balance and therefore shortens how many periods it takes to reach zero; it does not re-cast the dollar payment, because that isn't how a fixed contractual payment behaves. Recomputing `PMT` period-to-period off the current balance would silently re-amortize the loan every time a prepayment occurred — a real bug, not a simplification, since it quietly lowers every future payment instead of shortening the tail.

**Fully amortizing — level principal:**

```
Scheduled Principal (this period)
  = Original Principal ÷ Number of amortizing periods
```

`Number of amortizing periods` is the original count, fixed at the tranche's terms — guard the denominator (`MAX(1, …)`) in case an interest-only stub consumes the entire stated term, leaving no amortizing periods at all; treat that as a structure worth asking the modeler about rather than a formula to silently divide by zero.

Both styles are guarded the same way: `MIN(Opening Balance, Scheduled Principal)` in every period, so a rounding difference — or a prepayment that pays the tranche off early — never tries to pay down more than remains. **Every period after a tranche reaches zero, whether at its stated maturity or early because of a prepayment, shows zero scheduled principal, zero interest, and an unchanged balance — never a blank cell**, the same rule the roll-forward's `SUM` depends on in every other schedule.

**Where a tranche is both floating-rate and fully amortizing, ask whether the payment recasts at each rate reset or the term flexes instead.** Some agreements hold the term fixed and let the dollar payment change with the rate (recompute `PMT` off the current balance and current rate, but only *on reset dates*, not every period); others hold the payment fixed and let the amortization period run long or short. This is a real policy difference between agreements — don't assume either one.

## Interest mechanics

**Period rate:**

```
30/360:            Period Rate = Annual Rate / 12
Actual/360 or 365:  Period Rate = Annual Rate × (Actual days in period / 360 or 365)
```

**Floating-rate tranches:**

```
Annual Rate (this period) = Reference Rate (as of this period's reset date) + Spread
```

Look the reference rate up against the curve on `Debt_Assumptions` by period, the same `INDEX`-against-a-labeled-range pattern Foundation prefers over a Name Manager range. Where the reset frequency (Phase 2) is quarterly rather than monthly, the rate only updates on reset periods — carry it forward unchanged in between, don't silently recalculate it every month.

**Interest expense, every tranche:**

```
Interest Expense (this period) = Opening Balance × Period Rate
```

Interest is calculated on the **opening** balance for every tranche — the standard convention for a period-pay loan, and it keeps each tranche's interest independent of any other cash activity in the model.

## Roll-forward, per Foundation's sign convention

This is a balance roll-forward: every line signed by its effect on the balance, additions positive, reductions negative, so the closing balance is a single `SUM` down the block.

```
Closing Balance = Opening Balance + Draws − Scheduled Principal − Prepayments
```

**Build surface note.** On the live-Excel surface, each tranche's period-by-period balance is a natural `SCAN` candidate — the state that carries forward is the balance itself, and the accumulator function is `Balance − Scheduled Principal − Prepayments + Draws`. Pair it with a brief note describing what accumulates and what resets it (nothing, within a tranche's life). On the chat/Cowork surface, build the same logic as one formula copied across every period column.

---

# 4. Checks

**These live on the `Checks` tab**, per Foundation: a master ALL CLEAR / ERROR cell that reads clear only when every row below it passes, and one clearly-labelled PASS/FAIL row per check with conditional formatting.

Foundation caps this at a handful of checks that catch real breakage. The set below is deliberately short; anything more exhaustive — covenant ratio testing in particular, which varies by credit agreement and is not a universal mechanical check — is `financial-modeling-auditor` territory, or a separate covenant-analysis overlay on top of this schedule.

- **Roll-forward ties out:** for every tranche, Opening Balance + Draws − Scheduled Principal − Prepayments = Closing Balance.
- **No tranche's balance goes negative** in any period.
- **Total opening balance ties to the existing debt schedule's total** (or the balance sheet debt line, where supplied).
- **Every bullet/balloon tranche's balance is exactly zero the period after maturity**, and exactly the pre-maturity balance in the maturity period itself.
- **Every fully-amortizing tranche's balance is zero at or before its stated maturity date** — a schedule that leaves a residual balance past maturity has a rate, term, or payment-style mismatch somewhere upstream.

## Diagnostics that are not checks

Carry these as memo rows beneath the roll-forward on `Debt_Schedule`, not on the `Checks` tab:

- Total debt outstanding by period, across all tranches, for tying to the balance sheet.
- Total interest expense by period, across all tranches, for tying to the P&L.
- Weighted average interest rate across tranches, by period.

## Reconciliation outcomes are documentation, not checks

Whether the existing debt schedule's total reconciled to a supplied balance sheet figure belongs in the **"Notes and sources" block**, with the modeler's recorded decision on any variance. `financial-modeling-auditor` reads it there.
