---
name: capex-depreciation-builder
description: "Builds a board-room and PE-diligence grade monthly fixed asset / capex and depreciation schedule in Excel — asset-by-asset roll-forward from opening NBV and new capex additions through depreciation to closing NBV, with disposal and gain/loss handling. Supports straight-line, declining balance (with the standard switch-to-straight-line), and units-of-production depreciation, each chosen per asset. Interviews the modeler for in-service timing convention, capitalization threshold, and depreciation policy before building; classifies source files by the columns they carry rather than their names; reconciles the fixed asset register and the capex budget before building. Use whenever the user asks to build a capex schedule, a depreciation schedule, a fixed asset roll-forward, a PP&E (property, plant & equipment) schedule, an asset register forecast, or a capex budget with depreciation. For a combined model where this feeds a company-wide cash flow, use `master-cash-flow-builder` instead, which orchestrates this skill as one of its modules. This skill REQUIRES financial-modeling-foundation — always load and follow it first, and ask the user to enable it if it is not available."
---

# Capex & Depreciation Schedule Builder

**Requires `financial-modeling-foundation`.** Load and follow it first; everything below is *additive* to it, never a replacement. Where this skill appears to contradict Foundation, Foundation wins — tell the modeler which passage conflicts rather than choosing silently. If Foundation is not available, say so and ask the modeler to enable it before building.

**Determine the build surface before anything else.** Foundation makes this non-optional, and it changes the technique for the NBV roll-forward on every asset row:

- **Claude for Excel** (live Excel 365 workbook) — the full dynamic-array toolkit is available. Use `SCAN`/`REDUCE` with `LAMBDA` for each asset's stateful NBV roll-forward.
- **Claude.ai chat or Cowork** (file delivered as a download, LibreOffice recalculation in the pipeline) — no spilling arrays. One formula copied identically across every period column.

Ask if it is not obvious. Building for the wrong surface can silently ship a broken file.

Builds an asset-by-asset fixed asset schedule: every existing asset's remaining depreciation, every planned capex addition's depreciation from its in-service date, every disposal's gain or loss, rolling to a closing NBV that ties to the balance sheet. Every hardcoded number lives on an assumptions tab, every policy choice is recorded as a decision rather than assumed, every reconciliation test is recorded whether it passed or failed.

**Build in this order.** Each phase depends on the one before it, and skipping ahead is the most common way this schedule goes wrong.

1. Interview the modeler — two phases
2. Classify and reconcile the sources
3. Build the spine, then the asset-by-asset schedule
4. Handle disposals
5. Build the Checks tab

---

# 1. Interview the modeler

## Phase 1 — Independent facts

Ask these before looking at any file. They are things the modeler knows without opening the asset register, and several of them are policy choices this skill will not guess at. Ask them as a single block, not one at a time.

**This schedule runs monthly.** That's fixed, not a Phase 1 question — it matches the periodicity every other schedule in this plugin now shares, so a capex build can sit on the same `Macro_Assumptions` tab as a monthly A/R, A/P, debt, or payroll build without a periodicity mismatch.

> Before I look at any data, five things about the schedule itself:
> 1. **Company name**, as it should read on the cover of the schedule.
> 2. **Model start date and horizon** — the first forecast month, and how many months.
> 3. **Reporting units** — whole dollars, thousands, or millions?
> 4. **In-service timing convention.** When an asset is placed in service partway through a month, how should its first month of depreciation be handled?
>    - **Full-month convention** — placed in service any time during a month earns a full month of depreciation. Simplest, and the most common approach for monthly book schedules.
>    - **Half-month convention** — a half-month of depreciation in the month of acquisition and the month of disposal, regardless of the actual date. Standard for MACRS-style tax schedules; less common for book.
>    - **Exact date proration** — depreciation prorated by the actual days remaining in the month from the in-service date. Most precise, most useful where a single month's additions are material enough that a full-month assumption would distort that month's depreciation.
> 5. **Capitalization threshold.** Any planned purchase below this dollar amount is expensed as incurred rather than added to the schedule as a depreciable asset. What's the threshold — or is everything on the capex budget capitalized?

If the modeler cannot answer 2, stop. Every period boundary and every asset's depreciation start point is dated off it. Every other gap is recoverable with a named default, stated as an assumption.

**Where the in-service convention is not answered, do not default silently.** It's a genuine policy choice, not a modeling detail — the same asset produces a different first-period depreciation number under each of the three. Put the tradeoff to the modeler as written above and record the choice in the notes block.

## Phase 2 — Sources and methodology

Ask only after Phase 1 is answered.

> Now the data. Four kinds of file matter to this schedule, and one upload often fills more than one of them:
> 1. **Fixed asset register** — the existing, already-in-service asset list: description, in-service date, original cost, accumulated depreciation (or net book value), useful life, method, salvage value.
> 2. **Capex budget or forecast** — planned future purchases: description, expected in-service date, budgeted cost. Method and useful life are often missing here and filled from depreciation policy instead.
> 3. **Disposal schedule** — planned or already-executed disposals: which asset, disposal date, expected or actual proceeds.
> 4. **Depreciation and capitalization policy** — by asset category: default method, default useful life, default salvage value or percentage. This is what fills in every gap in the capex budget.
>
> Send whatever you have and I'll classify it by what's in it rather than what it's called. Where a role is missing I'll tell you what the schedule has to assume instead, and what to ask for.

**Where the fixed asset register carries a net book value but not accumulated depreciation (or vice versa), derive the missing one rather than asking twice** — `NBV = Cost − Accumulated Depreciation` holds by definition; use whichever two of the three the source provides to solve for the third, and flag any row where a supplied third figure doesn't reconcile to the other two.

**Where the capex budget has no method or useful life for a line item, do not leave it blank or guess a number.** Apply the category's default from the depreciation policy, color the applied figure blue as an input, and note in the row that it was policy-derived rather than modeler-specified. Where the item's category itself is ambiguous, ask.

**Where units-of-production is selected for any asset, its total estimated units and its per-period usage driver are both required inputs** — this method cannot be built without them. Ask for the usage forecast (or historical usage pattern to project from) at the point a units-of-production asset is identified, not as a blanket upfront question if no asset uses the method.

---

# 2. Classify and reconcile the sources

## Classify by shape, not by name

*Never route on a filename, a tab name, or the system a file came out of. Open every source, read its header row, and classify it by the columns it holds.*

Four roles matter. One upload may fill more than one; one role may be split across several uploads; a role may be absent entirely.

| Role | Recognise it by | Notes |
|---|---|---|
| **Fixed asset register** | One row per existing asset — in-service date already in the past, and either accumulated depreciation or NBV populated. | Carries assets that are *already* depreciating. Their remaining schedule is what's left of an existing plan, not a new one. |
| **Capex budget / forecast** | One row per planned purchase — an in-service date that falls **inside or after** the forecast window, and typically no accumulated depreciation column at all, because the asset doesn't exist yet. | Method and useful life are frequently absent and filled from policy. |
| **Disposal schedule** | Asset identifier, disposal date, proceeds (or expected proceeds). | Often just a status column and a date inside the fixed asset register rather than a separate file — check there before concluding the role is unfilled. |
| **Depreciation / capitalization policy** | Not a transaction list at all — a short table of category, default method, default useful life, default salvage. | This is the fallback source for every gap in the other three. |

## Derive each assumption from the best available source

Work down each row until a source exists. Take the first one that does, and record which one it was.

| Assumption | Derive from | Fall back to | Last resort |
|---|---|---|---|
| Opening NBV, per existing asset | Fixed asset register (Cost less Accumulated Depreciation) | Trial balance PP&E balance, undifferentiated by asset | Modeler input — the schedule cannot start without this |
| Useful life (original), existing assets | Fixed asset register's own useful-life column | Where the register gives only *remaining* life: back-calculate an effective original life as remaining life + periods elapsed since in-service, so every asset — existing or new — runs off the same per-period remaining-life formula (Section 3) | Depreciation policy's category default, applied from the in-service date |
| Method, existing assets | Fixed asset register | Depreciation policy's category default | Straight-line — state this was assumed, not observed |
| Cost, useful life, method — new capex | Capex budget, line by line | Depreciation policy's category default for whatever it omits | Modeler input |
| Salvage value | Fixed asset register or capex budget, where present | Depreciation policy's category default percentage of cost | 0 — state this was assumed |
| Units-of-production driver | A usage forecast or production schedule supplied for that asset | A historical usage pattern, projected forward at the modeler's direction | The method cannot be built without one — fall back to straight-line and say so |

**Reconcile before building.** Sum the fixed asset register's opening NBV and tie it to the balance sheet's PP&E line if one is supplied. Sum the capex budget's total planned spend and tie it to any capex line in a P&L or cash flow forecast if one exists. Record whether each tie passed, failed, or was carried as a disclosed reconciling item in the notes block — this is documentation, not a Checks-tab test (see Section 5).

---

# 3. Build the spine, then the asset-by-asset schedule

## Tab structure

Following Foundation's standard order — Cover/README, Macro_Assumptions, Checks, then model-specific tabs — **model start date and reporting units from the Phase 1 interview land on `Macro_Assumptions`**, not on a schedule-specific tab. This schedule adds two tabs after Checks:

- **Capex_Assumptions** — the depreciation and capitalization policy table (category, default method, default useful life, default salvage %), the in-service convention chosen in the interview, and the capitalization threshold. This is the fallback source every gap in Fixed_Asset_Schedule pulls from.
- **Fixed_Asset_Schedule** — one row per asset, existing and new alike, on a single continuous list. Do not split existing assets and new capex onto separate tabs; the roll-forward logic is nearly identical for both and a reader auditing total NBV needs one place to sum it.

## Format and cell conventions

- Format the full model in **Roboto**. Schedule labels, dates, values and cell contents are all Roboto 11.
- **B2** — company name. Roboto 14, bold. **Link it to `Macro_Assumptions`** (green font) rather than typing it.
- **B3** — the schedule title, built from the period count so it cannot drift: `="`&periods&`-month capex & depreciation schedule"`. Roboto 11, plain.
- **B4** — the reporting units, written out. **Link to `Macro_Assumptions`** (green font). Foundation defaults to actual dollars where the modeler expresses no preference.
- Row height 15.00 throughout unless specified otherwise. Labels sit in column B and run across C–E as needed. **Column F is the opening column; forecast periods begin in column G.**

This B2:B4 block is the *sheet* header on `Fixed_Asset_Schedule`, not a substitute for Foundation's Cover tab. Build both — the Cover carries the Change Log and version; this block identifies the sheet when it is printed or exported on its own.

| Cell type | Font | Fill | Border |
|---|---|---|---|
| Input | Blue `#0000FF` | Light blue `#EEF5FC` | Hair-weight bottom |
| Formula | Black | None | None |
| Link to another tab in the same workbook | Green `#008000` | None | None |
| Link to an external file | Purple, per Foundation | None | None |

- **Never use yellow fill to identify inputs.** The blue input convention alone identifies what the reader must replace. Per Foundation, yellow fill is reserved for *flagged* cells only.
- **No annotation or comment text in column E**, or any column adjacent to the label or value columns. All notes belong in the "Notes and sources" block on `Capex_Assumptions`.
- Zoom 80% and gridlines off on every sheet. Size column widths so no label is truncated. Freeze panes immediately below the header spine and to the left of the first forecast column.

## Row structure — one row per asset

Every asset, whether already in service or a planned capex addition, gets the same row shape:

`Asset ID | Description | Category | In-Service Date | Cost | Salvage Value | Useful Life (original, periods) | Method | [Units-of-production only: Total Estimated Units, usage-driver reference] | Opening NBV | Period 1 Depreciation | Period 2 Depreciation | ... | Accumulated Depreciation | Closing NBV`

Useful Life is always the asset's **original** life, sourced or back-calculated as described in Section 2 — never overwritten as time passes. Remaining life, where a formula needs it, is derived fresh each period (see the declining balance mechanics below), not stored as a second column that would drift out of sync with the first.

**Existing assets** carry their opening NBV from the fixed asset register in the model's opening column (per Foundation's opening-column convention — this is a roll-forward, so it gets one). **New capex additions** carry a blank opening NBV; their schedule doesn't start until their in-service date falls inside the forecast window, at which point their Cost becomes their effective opening balance for depreciation purposes.

Depreciation only begins in the month containing (or, in half-month convention, the month of) the in-service date, and only if that date is not later than the current period. A row's periods before its in-service date, and every period after the asset is fully depreciated or disposed, show zero depreciation and an unchanged NBV — never a blank cell, so the roll-forward's `SUM` down the block never breaks on a missing value.

## Depreciation mechanics, by method

Each asset's Method column determines which formula its depreciation row uses. Never hardcode a method into a formula — reference the Method column and branch on it, so changing one asset's method changes one input cell.

**Straight-line**

```
Period depreciation
  = IF(period date < in-service date, 0,
      MIN(NBV_open − Salvage, (Cost − Salvage) / Useful_Life_Periods))
```

The `MIN` guards against depreciating past salvage value in the asset's final period, where the straight-line amount may exceed what's left to depreciate.

**Declining balance**

`Remaining_Useful_Life_Periods` is derived fresh every period — never a stored column, so it can't drift out of sync with the original life it's computed from:

```
Remaining_Useful_Life_Periods (this period)
  = MAX(1, Useful_Life_Periods − periods elapsed since in-service date, counted through this period)
```

The `MAX(1, …)` guard prevents a divide-by-zero in the asset's final period, when elapsed periods equals its useful life.

```
Declining balance amount
  = NBV_open × (multiplier / Useful_Life_Periods)

Straight-line-on-remainder amount
  = (NBV_open − Salvage) / Remaining_Useful_Life_Periods

Period depreciation
  = IF(period date < in-service date, 0,
      MIN(NBV_open − Salvage, MAX(Declining balance amount, Straight-line-on-remainder amount)))
```

Taking the greater of the two each period is what produces the standard switch to straight-line partway through the asset's life — once the straight-line-on-remainder amount overtakes the declining balance amount, every subsequent period uses it automatically. No separate switch-detection logic is needed; the `MAX` does it. Ask the modeler for the multiplier (200% for double-declining, 150%, etc.) as part of the category policy — never assume double-declining.

**Units-of-production**

`Total_Estimated_Units` is an input on the asset's own row (see Row structure, above), never buried inside a formula.

```
Period depreciation
  = IF(OR(period date < in-service date, Total_Estimated_Units = 0), 0,
      MIN(NBV_open − Salvage, (Cost − Salvage) / Total_Estimated_Units × Units_this_period))
```

The `Total_Estimated_Units = 0` guard prevents a divide-by-zero where the input hasn't been filled in yet — treat that as an unfilled input to flag (Foundation's yellow-fill convention), not a silent zero. `Units_this_period` references that asset's usage driver row — either its own, or a shared category-level driver where several assets run off the same production schedule. Where a shared driver is used, say so in the notes; it means those assets' depreciation is correlated, which matters to anyone stress-testing the schedule.

## Roll-forward, per Foundation's sign convention

This is a balance roll-forward: every line signed by its effect on NBV, additions positive, reductions negative, so the closing balance is a single `SUM` down the block.

```
Closing NBV = Opening NBV + Capex Additions (at cost, in the in-service period) − Depreciation − Disposals (at NBV, see Section 4)
```

**Build surface note.** On the live-Excel surface, each asset's period-by-period NBV is a natural `SCAN` candidate — the state that carries forward is NBV itself, and the accumulator function is `NBV − this period's depreciation`, gated by the `IF` conditions above. Pair it with a brief note describing what accumulates (NBV) and what resets it (nothing, within an asset's life — a new asset is a new row, not a reset within one). On the chat/Cowork surface, build the same logic as one formula copied across every period column.

---

# 4. Handle disposals

A disposal removes an asset from the schedule and, separately, produces a gain or loss that is a P&L event rather than a roll-forward line.

**In the roll-forward:** in the period of disposal, the asset's NBV drops to zero — recorded as a negative line equal to its NBV immediately before disposal (i.e., after that period's depreciation, if the convention takes a partial period in the disposal period; before it, if not — apply the same in-service convention chosen in Phase 1 symmetrically to disposals, and say so). Every period after disposal shows zero for that asset, same as a fully depreciated one.

**As a memo output, not a roll-forward line:**

```
Gain / (Loss) on Disposal = Proceeds − NBV at disposal date
```

Carry this as its own row beneath the roll-forward on `Fixed_Asset_Schedule`, one column per period, summed across every asset disposing that period. It is not part of depreciation expense and should never be netted into it — a reader reconciling depreciation expense to the P&L will look for this line separately.

---

# 5. Checks

**These live on the `Checks` tab**, per Foundation: a master ALL CLEAR / ERROR cell that reads clear only when every row below it passes, and one clearly-labelled PASS/FAIL row per check with conditional formatting.

Foundation caps this at a handful of checks that catch real breakage. The set below is deliberately short; anything more exhaustive is `financial-modeling-auditor` territory.

- **Roll-forward ties out:** for every asset, Cost − Accumulated Depreciation − (0 if not yet disposed, else full remaining NBV written off) = Closing NBV.
- **No asset depreciates below salvage value** in any period — test the minimum of each asset's NBV row across all periods against its salvage value.
- **Total opening NBV ties to the fixed asset register's total** (or the balance sheet PP&E line, where supplied).
- **Total capex additions tie to the capex budget's total planned spend.**
- **No asset on the schedule has a cost below the capitalization threshold** — a line item that small should have been expensed at the source, not depreciated; a hit here usually means a capex budget line was capitalized by mistake.
- **Every disposed asset's post-disposal periods show zero NBV and zero depreciation.**
- **Depreciation policy percentages/multipliers are populated for every category referenced** on `Fixed_Asset_Schedule` — an asset whose category has no matching row on `Capex_Assumptions` is a silent gap, not a zero.

## Diagnostics that are not checks

Carry these as memo rows beneath the roll forward on `Fixed_Asset_Schedule`, not on the `Checks` tab:

- Total depreciation expense by period, for tying to the P&L.
- Total capex additions (at cost) by period, for tying to the cash flow forecast's capex line — note that this is the *in-service* period, not necessarily the *cash payment* period if the modeler has indicated payment terms differ from in-service timing (ask, don't assume, if this matters to the build).
- Gain/(loss) on disposal by period, for tying to the P&L's non-operating section.

## Reconciliation outcomes are documentation, not checks

Whether the fixed asset register's total reconciled to a supplied balance sheet figure, and whether the capex budget's total reconciled to a supplied P&L or cash flow capex line, belongs in the **"Notes and sources" block**, with the modeler's recorded decision on any variance. `financial-modeling-auditor` reads it there.
