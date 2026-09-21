# Seidman Financial Modeling

Standards and builders for financial models in Excel, written to hold up to board-room and private-equity-diligence scrutiny.

**Status: early.** These skills are internally consistent and cross-checked against each other, but treat findings from real builds as expected refinements, not failures.

## What's here

| Skill | Role |
|---|---|
| `financial-modeling-foundation` | Universal standards for **every** model: colour conventions, tab structure, sign conventions, hardcoding discipline, formula rules, circularity, dynamic arrays and the surface rule, the opening column, source classification and provenance, reconciliation protocol, check-tab design, documentation. |
| `financial-modeling-auditor` | Independent audit of an existing model against Foundation — numeric re-derivation, provenance and reconciliation testing, structural hygiene, severity and materiality. Works on models Claude did not build. |
| `variance-analysis` | Builds a flexed-budget variance bridge — actual vs. budget or a prior forecast — separating volume variance from rate/spending variance, with an optional price/volume/mix decomposition and an EBITDA bridge waterfall. Looks backward at completed periods rather than forecasting forward. |
| `13-week-cash-flow-builder-ar` | The accounts receivable module of a 13-week direct-method cash flow forecast — aged and new-sales collections, aging waterfall, credit memos, uncollectibles by bucket, and customer segmentation. Routes between an invoice-level gather and a percentage-profile method depending on what the sources support. |
| `13-week-cash-flow-builder-ap` | The accounts payable module of the same forecast — a consolidated payables roll-forward splitting the opening balance into significantly aged and existing pools, new purchase disbursements net of holdback, holdback accrual and release, and payment prioritization under a cash constraint. Separates observed payment behaviour from stated terms so the current stretch is visible. |
| `13-week-cash-flow-builder-payroll` | A bottom-up, roster-driven payroll forecast — gross wages, employer payroll taxes with wage-base caps tracked off cumulative YTD wages, benefits, PTO liability roll-forward, and bonus accrual and payout. Separates the pay period the cost accrues in from the pay date the cash actually moves. |
| `13-week-cash-flow-master` | Orchestrates `13-week-cash-flow-builder-ar`, `-ap`, and `-payroll` in sequence against one shared spine, consolidates their outputs into a single combined cash roll-forward, and routes to `variance-analysis` once actuals exist. The hub sequences and consolidates; it never invents a collection curve, a payment policy, or a wage calculation of its own — every mechanic belongs to its spoke skill. |
| `deconstruct-finance-workflow` | Turns a rough description of a recurring finance or accounting process — month-end close, flux analysis, reconciliations, budgeting, board reporting — into a structured Workflow Requirements document that an AI build step can act on. A finance-specialized adaptation of the AI Workflow Framework's Deconstruct step. |

## How they fit together

Hub-and-spoke. Foundation is the universal layer; the master skill is the hub; the three modules are the spokes.

```
                    financial-modeling-foundation
                     (universal layer — always first)
                                  │
        ┌────────────┬────────────┼────────────┬─────────────┐
        │            │            │            │             │
   auditor      variance-    builder-ar   builder-ap   builder-payroll
  (reviews an    analysis         │            │             │
  existing model  (backward-      └────────────┼─────────────┘
  against          looking,                    │
  Foundation)      standalone)         shared weekly spine
                        ▲                      │
                        │           13-week-cash-flow-master
                        └───────────  (sequences the three modules,
                        once actuals   builds the one combined
                        exist          cash roll-forward no
                                       module builds alone)

   deconstruct-finance-workflow — standalone; runs before any build,
   to document the process a model or automation is meant to serve.
```

## The spine

Every cash flow build is anchored on a **Week 1 start date**. The model runs Monday-to-Sunday, thirteen columns. Each module derives every date from that anchor, which is what lets the hub consolidate them without reconciling three different calendars.

The modules ask for the anchor in their own Phase 1 interview when run standalone. Run through the hub, it is asked once.

## Build surface

Each builder determines its surface before anything else, because it changes the technique for every period-driven row:

- **Claude for Excel** (live Excel 365 workbook) — the full dynamic-array toolkit. Header spine, profile rows, and roll-forward are built as spilling formulas.
- **Claude.ai chat or Cowork** (file delivered as a download, LibreOffice recalculation in the pipeline) — no spilling arrays. One formula copied identically across every period column.

Building for the wrong surface silently ships a broken file, so the skills ask when it isn't obvious.
