---
name: financial-modeling-foundation
description: "Universal standards for building, editing, or auditing ANY financial model in Excel — cell color conventions, tab structure and naming, sign conventions, units, hardcoding discipline, formula construction rules, the no-named-ranges rule, circularity handling, dynamic array usage (including the strict Excel-vs-LibreOffice surface rule), check-tab design, the opening-column convention, source classification and provenance discipline, and documentation standards. ALWAYS consult and follow this skill first before building, editing, reviewing, or auditing any financial model, cash flow forecast, budget, FP&A planning model, valuation model, headcount model, capex/depreciation schedule, or any other Excel-based financial planning deliverable — even for a model type with no dedicated builder skill yet. This is the shared foundation that every model-specific builder skill and financial-modeling-auditor depends on and must reference rather than duplicate."
---

# Financial Modeling Foundation

This skill defines the standards that apply to **every** financial model, regardless of type. Model-specific builder skills and `financial-modeling-auditor` both depend on this skill and must not duplicate or contradict it. If you are working from a model-specific skill, treat its instructions as *additive* to this one, not a replacement.

Models built under this standard are expected to hold up to board-room and private-equity-diligence-level scrutiny. Assume the person you're working with has prior modeling literacy — do not explain modeling basics unless asked.

## Guiding principle: scoped confidence

Only act with authority where this skill (or a model-specific skill layered on top of it) actually gives you the knowledge to do so.

- If the modeler asks for a model type that has a dedicated builder skill available, follow that skill's specific guidance in addition to this one.
- If the modeler asks for a model type with **no dedicated builder skill**, do not improvise structure or methodology from general training knowledge. Say so directly, and ask the modeler what conventions, structure, and level of detail they want — the same way a careful analyst would ask a manager before guessing. Still apply this Foundation skill's universal standards (colors, tabs, formulas, checks) to whatever gets built.
- Never guess at business-specific mechanics (collection curves, payment terms, granularity) that vary from company to company. Ask.

## Cell color conventions

Font color communicates **the source of truth** for a cell's value at a glance:

| Color | Meaning |
|---|---|
| **Blue** | Hardcoded input / constant |
| **Black** (default) | In-sheet formula |
| **Green** | Link to another tab in the *same* workbook |
| **Purple** | Link to an *external* file (bank extract, another model, a source system). This should be rare — most models should be self-contained. |
| **Yellow fill** | Flagged cell — something the modeler should look at or fill in |

**Never use red anywhere except the Checks tab.** Red implies an error state, and error signaling is reserved exclusively for the Checks tab — where it is the correct choice for the conditional formatting on a failing check and the master ERROR flag. Red appearing anywhere else in a model means something is signalling an error outside the one place errors are supposed to surface.

Inputs are simply constants, wherever they naturally live in the model. Do not assume all inputs are centralized in one tab — a schedule's own assumptions can live at the top of that schedule. Blue coloring alone is the traceability mechanism for *what is an input*; do not add data validation or cell comments to inputs by default. This is separate from documenting **where an input came from** — where a builder skill requires provenance, that belongs in a notes block, not in cell comments.

## Tab structure and naming

**Naming convention:** use underscores instead of spaces in every tab name (e.g. `Macro_Assumptions`, not `Macro Assumptions`). This avoids needing `'Sheet Name'!` quoting in cross-sheet formulas. Do not number tab names (no `1.0_README`).

**Standard tab order:**
1. **Cover / README** — first tab in the workbook. Contains the model title, purpose, and the Change Log (see Documentation below).
2. **Macro_Assumptions** — global assumptions used throughout the model, not scoped to a single schedule: company name, currency denomination, forecast start date, fiscal year end, and any other cross-cutting controls (e.g. number of periods to forecast, minimum cash threshold). This is also where period-rolling/extension controls live (see Time Period Handling).
3. **Checks** — sits immediately behind Macro_Assumptions, not at the back of the workbook. Contains the master ALL CLEAR / ERROR flag and individual PASS/FAIL check rows (see Built-In Error Checks below). Placing it early makes it the first thing anyone auditing the model encounters.
4. **Model-specific tabs** — the actual schedules. Order and structure depend on the model type (see the relevant builder skill). As a default pattern: a compact model can keep summary + supporting schedules stacked vertically on one tab; a large model should split by statement/function across multiple tabs, with supporting detail stacked beneath each schedule's own summary row, in the same top-to-bottom order the line items appear in the parent schedule above them. If a supporting schedule grows large enough to be "an abundance of related data" in its own right (e.g. a full AR aging build), promote it to its own dedicated tab rather than stacking it.

**Visual separation between stacked schedules on the same tab:** use a labeled section header row (bold/shaded row naming the schedule) — not blank spacer rows alone or borders.

## Sign convention

- **P&L (income statement):** revenue, costs, and expenses are all shown as **positive** numbers. Profit itself may display as negative if there's a loss, but the line items feeding it are not sign-flipped.
- **Cash flow statements:** cash **in** is positive, cash **out** is negative.
- **Balance roll-forwards** (A/R, A/P, debt, inventory, fixed assets): every line is signed by its effect **on the balance**, not by its effect on cash. Additions positive, reductions negative — so the closing balance is a single `SUM` down the block rather than a chain of alternating signs.

**The roll-forward convention collides with the cash flow convention, and the collision is one-directional.** A collections line reduces A/R (negative in the roll forward) but is cash in (positive in the cash flow); a payments line reduces A/P (negative) and is also cash out (negative). **Flip the sign exactly once, at the point a roll-forward line feeds the cash flow statement, and label the row so nobody flips it twice.** Where a builder skill specifies which of its lines flip, follow it.

## Units and scale

Default to **actual dollars**, not $000s or $mm, unless the modeler specifies otherwise.

## Hardcoding discipline

Formulas must **never** contain embedded hardcoded numbers (e.g. `=B5*1.05` is not acceptable; `=B5*(1+$B$6)` is). Inputs are hardcoded in exactly one cell (colored blue) and referenced everywhere they're used. Do not assume all inputs live in one central location — they may be distributed across the schedules they belong to.

## Source data and provenance

Applies wherever a model is built from client extracts, system exports or third-party files rather than from figures the modeler supplies directly. Builder skills define *which* source roles matter for their model type and *what* each assumption should be derived from; the discipline below governs both, regardless of model type.

### Classify sources by shape, not by name

**Never route on a filename, a tab name, or the system a file came out of.** A file called "aging" may be a summary with no dates in it, and the file that actually carries the behaviour the model needs may be called anything at all. Open every source, read its header row, and classify it by the columns it holds.

- One upload may fill more than one role; one role may be split across several uploads; a role may be absent entirely.
- Classify what is there, name what is missing, and **never assume a role is filled because a file exists**.
- Where a role is unfilled, say what the model must assume instead, and what extract would remove the assumption.

### Derive each assumption from the best available source

A builder skill's derivation table gives, for each assumption, the source it should be derived from and the fallbacks in descending order of quality. **Work down each row until a source exists, take the first one that does, and record which one it was.**

Three rules govern any such table:

- **A derived figure outranks a supplied one.** Where the modeler offers a number the sources can produce, derive it anyway and show both — the gap between them is usually the most interesting thing on the page.
- **Every assumption carries its provenance** into the notes block, naming the source it came from and the tier it landed on. A reader must be able to tell a measured number from an assumed one without opening a formula.
- **A fallback is not a substitute.** Where an assumption drops to a lower tier, state what would have been needed to do better, so the modeler knows what to ask the client for.

Where no builder skill defines a derivation table for the model type, ask the modeler what each assumption should be derived from rather than choosing silently — per Scoped Confidence above.

### Reconcile the sources before building

**Sources assembled independently rarely agree. Test them against each other before a single forecast cell is written.** Every hour spent modelling on top of a broken tie is wasted, and a model that cannot reconcile to its own inputs will not survive its first review. Builder skills define the specific tests for their model type; the protocol below applies to any of them.

**Where a test fails, stop and put it to the modeler:**

> "Before I build anything: [name the test] does not reconcile. [State the figures.] That is not something the model can absorb — it means the sources describe different things. Do you want me to go back for a consistent extract, adjust one source to the other and document the adjustment, or proceed on the sources as given and carry the break as a disclosed reconciling item?"

Those three are the only options. Adapt the wording to the situation — "go back" means the client in advisory work and the source system owner in an internal model — but do not add a fourth option, and do not resolve it without the modeler.

**Do not choose silently, and do not quietly scale one file to fit another.** A model built over an undisclosed plug is worse than no model, because it looks finished.

**Record the outcome of every test, passed or failed, in the notes block**, and carry any accepted break as a named reconciling item rather than folding it into an assumption.

## Formula construction rules

- **No rigid formula hierarchy.** There is no mandatory "always use X before Y." The goal is formulas that are **short and auditable**. `SWITCH`, `CHOOSE`, and `XLOOKUP` are generally strong choices for categorical logic and lookups — reach for them when they keep a formula clean. If logic becomes more complex or needs internal documentation, use `LET` to break it into clearly labeled steps rather than writing one dense, unreadable formula.
- **Never nest IFs.** Use `SWITCH`, `CHOOSE`, `IFS`, or a lookup instead.
- **Never merge and center cells.** "Merge across selection" is fine where it aids formatting; merge-and-center is not.
- **Avoid volatile functions where reasonably possible** (`OFFSET`, `INDIRECT`, etc.), but this is a qualitative judgment call, not a hard limit to track or report against. If a model leans on volatile functions heavily enough to risk performance issues (a rough mental benchmark: on the order of 100+ instances), that's a signal to reconsider the approach — but don't spend effort counting instances as you build.
- **Formula consistency across a row/period matters enormously.** A single formula, copied consistently across every period, with no mid-row exceptions, is one of the most important discipline points in this entire standard — it's the most common source of silent, hard-to-catch modeling errors. Where a formula appears to *need* an exception in the first period, that is a structural problem with a structural fix — see The Opening Column under Time Period Handling. Do not declare the exception; remove the need for it.

## No named ranges — LET only

**Never create named ranges in Excel's Name Manager for cells, ranges, or formulas.** This applies globally, across every cross-reference in every model — not just specific cases like array-slicing.

The **only** exception is naming actual Excel **Tables** (structured references like `sales_fcst[weekly_sales]` are fine — that's a native Table mechanic, not a manually created named range).

All in-formula labeling is done through `LET`, so a formula documents its own logic without relying on the Name Manager.

## Circularity handling

**No circularity, switches, or circularity breakers anywhere in the model.** Where a scenario would normally produce circularity — most commonly revolver/financing activity tied to a cash balance — structure the formula so it only ever references **beginning-of-period** balances, never the current period's own ending result. A revolver draw sized off opening cash, opening borrowings and the period's forecast movement resolves without iteration; where a builder skill defines a model-specific variant, follow it.

## Dynamic arrays — this is surface-dependent. Read carefully.

This is one of the most consequential rules in this skill, because getting it wrong doesn't just produce a suboptimal model — it can produce a file that's actively broken.

**The constraint:** when Claude builds a file in Claude.ai chat or Cowork (no live, already-open workbook), the file is written with `openpyxl` and then run through a **mandatory** recalculation step powered by **LibreOffice**. LibreOffice does not natively support genuine Excel dynamic-array spill syntax. When it encounters formulas built around `SEQUENCE`, `HSTACK`, `VSTACK`, `TAKE`, `DROP`, `FILTER`, `SORT`, `UNIQUE`, `SCAN`, `REDUCE`, `MAKEARRAY`, `BYROW`, or `BYCOL`, it either fails to evaluate them correctly, silently truncates a spilling result to a single cell while still reporting zero errors, or — confirmed directly from a real file during this skill's development — **rewrites genuine Excel spill references into `ANCHORARRAY(...)` calls**, which is not a real Microsoft Excel function. A file corrupted this way throws `#NAME?` errors the moment it's opened in real Excel 365.

**Never write `ANCHORARRAY` under any circumstance.** If you ever see it in an existing file you're auditing or editing, treat it as a sign the file has been touched by LibreOffice and flag it — the underlying spill reference should be genuine Excel syntax (the `#` operator, e.g. `=SUM(G8#)`), not a function call.

**The rule this produces:**

- **In Claude for Excel** (a live, already-open Excel 365 workbook — no LibreOffice in the loop): the full dynamic-array toolkit is fully available and encouraged. Genuine Excel evaluates everything natively and correctly.
  - Use `SEQUENCE`-driven header rows for period generation.
  - Combine periods and totals into a **single `HSTACK` formula** (e.g. `=LET(array, SEQUENCE(1,periods,1,1), total, "Total", HSTACK(array, total))`) so the whole range — periods and total together — spills and resizes as one unit. This is the correct way to avoid the total-column collision problem; do not solve it by repositioning the total or hardcoding a fixed range.
  - Reference a spilled range using the genuine `#` operator or array-slicing functions (`TAKE`, `DROP`, `INDEX`) — never a named range.
  - Use `SCAN`/`REDUCE` with `LAMBDA` specifically for **stateful roll-forwards** — a balance that carries period-to-period (cash, revolver, AR/AP roll-forward, depreciation/NBV, headcount) — as a single formula rather than a chain of individual cells each referencing the prior period. This is more robust than the traditional chained-cell approach, not just more elegant: there's no chain of individual cells to accidentally sever with an inserted column or an inconsistent mid-row edit. Always pair a `SCAN`/`REDUCE` formula with a brief plain-English note nearby describing what accumulates and what resets it, since this is the one real cost of the technique (harder to read at a glance than a simple chained formula).
  - Use `MAKEARRAY`+`LAMBDA` freely for generating dynamically-sized blank/spacer structures that match another array's dimensions — this is a formatting utility, not modeling-logic complexity, and carries no real audit downside.
  - For **independent, stateless calculations** (the majority of line items — most revenue, cost, and disbursement forecasting, where a period's value doesn't depend on the prior period's result), prefer a single array-argument formula (e.g. a `SUMIFS` whose criteria argument is an entire spilled period range) over `SCAN`/`REDUCE`. Reserve the stateful techniques for genuinely stateful calculations.
- **In Claude.ai chat or Cowork** (no live workbook — file delivered as a download, mandatory LibreOffice recalculation in the pipeline): **do not attempt true spilling dynamic arrays.** Use the traditional pattern instead — one formula, copied identically across every period column via relative references, extended by adding columns rather than by an automatic spill. `LET` and `XLOOKUP` are safe in this environment (they resolve to single values, not spill ranges) and should still be used freely for readability and lookups. Only the spilling functions listed above are off-limits here.

**Before building any model, determine which surface you're operating in, and choose the build technique accordingly. This is not optional — attempting the wrong technique for the surface can silently ship a broken file.**

## Built-in error checks

The Checks tab (positioned right behind Macro_Assumptions) contains two elements:

1. **A master ALL CLEAR / ERROR flag** — one cell, readable at a glance. Reads ALL CLEAR only when every individual check below it passes.
2. **Individual PASS/FAIL rows**, one per check, each labeled clearly enough that what it's checking is self-evident without further explanation, with conditional formatting so failures are visually obvious.

Keep the check set reasonable and targeted — a handful of meaningful checks that catch real breakage, not an exhaustive audit (that's what `financial-modeling-auditor` is for). Model-specific builder skills define what belongs on this tab for their model type.

## Time period handling

Time periods should support a fixed starting shape (e.g. 13 weeks) that can be rolled forward (change the start date, the periods shift accordingly) and extended or shortened (change the number of periods) via a **dedicated input on the Macro_Assumptions tab** — never by manually rebuilding the model. See the surface-dependent Dynamic Arrays section above for how this is technically achieved depending on where you're building.

### The opening column

**Every model with a period spine carries an opening column immediately to the left of the first forecast period.** It is not a forecast period and is never presented as one. It exists for a single reason: so that no formula in the model needs a first-period exception.

Two exceptions arise in almost every model without one, and both are structural rather than unavoidable:

- A row that references the **prior period** — a mid-period transaction date, a growth rate off the previous period, a lag — has no prior period to reference in the first forecast column, so the modeler writes a variant formula there.
- A **roll-forward's beginning balance** takes an opening figure from an input in the first period, and the prior period's ending balance in every period after it.

The opening column removes both. It carries:

- **Period number** — `0`, or the label `Opening`.
- **Period beginning date** — one period before the first forecast period. Derive it the same way the spine derives every other period boundary (subtract the period length for daily or weekly models; use `EOMONTH`-style logic for monthly or quarterly, where the length varies). Do not hardcode a day count.
- **The opening balance of every roll-forward in the model**, placed in that roll-forward's *ending balance* row — as a link to wherever the input lives, not as a typed value.

With those in place, `beginning balance = prior column's ending balance` and every prior-period reference resolve identically in the first forecast column as in all the others. Each such row becomes one formula copied across, which is what makes the formula-consistency rule achievable rather than aspirational.

**Three rules govern it:**

- **Never include it in a total.** Sums, averages and period counts run from the first forecast period, not from the opening column.
- **Never let a date-driven gather match against it.** Its date sits one period before the forecast window and any `SUMIFS` or lookup that catches it will double count.
- **Make it visually distinct** — shade it, grey the font, or group it — so a reader sees at a glance that it is an anchor rather than a period.

Leave every other row in the opening column blank. It anchors; it does not forecast.

**Declaring a first-period exception is not an acceptable substitute.** A declared exception is still an exception: it breaks formula consistency, and it forces every audit of the model to carry a permanent known-exception carve-out, which is exactly how real findings get waved through.

## Documentation and version control

Keep this lightweight — it lives **inside the Cover/README tab itself**, not a separate tab:
- A short **Change Log**: date, version, author, what changed.
- A **version number** on the same tab, updated on material structural changes.

Avoid filename-based versioning — files get renamed and copied, and filename versions drift out of sync with the file's actual content.
