# Seidman Financial Modeling

Standards and builders for financial models in Excel, written to hold up to board-room and private-equity-diligence scrutiny.

**Status: early.** These skills are internally consistent and cross-checked against each other, but treat findings from real builds as expected refinements, not failures.

## What's here

| Skill | Role |
|---|---|
| `financial-modeling-foundation` | Universal standards for **every** model: colour conventions, tab structure, sign conventions, hardcoding discipline, formula rules, circularity, dynamic arrays and the surface rule, the opening column, source classification and provenance, reconciliation protocol, check-tab design, documentation. |
| `financial-modeling-auditor` | Independent audit of an existing model against Foundation — numeric re-derivation, provenance and reconciliation testing, structural hygiene, severity and materiality. Works on models Claude did not build. |
| `variance-analysis` | Builds a flexed-budget variance bridge — actual vs. budget or a prior forecast — separating volume variance from rate/spending variance, with an optional price/volume/mix decomposition and an EBITDA bridge waterfall. Looks backward at completed periods rather than forecasting forward. |
| `cash-flow-builder-ar` | The accounts receivable module of a monthly direct-method cash flow forecast. |
| `cash-flow-builder-ap` | The accounts payable module of the same forecast, including payment prioritization under a cash constraint. |
| `payroll-schedule-builder` | A roster-driven payroll forecast — computes at the payroll's true pay frequency (weekly/bi-weekly/semi-monthly) via a pay-period ledger, rolls up to monthly output. |
| `capex-depreciation-builder` | An asset-by-asset (not pooled) capex and depreciation roll-forward — book methods only: straight-line, declining balance, units-of-production. |
| `debt-amortization-builder` | A tranche-by-tranche term loan schedule — fixed or floating rate, fully amortizing, interest-only, or bullet/balloon, on a monthly periodicity. Term loans only; no revolver. |
| `master-cash-flow-builder` | Orchestrates `cash-flow-builder-ar`, `cash-flow-builder-ap`, and `payroll-schedule-builder` (always), plus `capex-depreciation-builder` and/or `debt-amortization-builder` (where in scope), into one combined monthly cash flow model on a shared spine. Runs one consolidated interview, resolves tab-naming collisions across modules, builds a `Consolidated_Cash_Flow` roll-forward with correct-sign pulls from each module, and a light `PL_Memo` tab. |

## How they fit together

```
                    financial-modeling-foundation
                     (universal layer — always first)
                                  │
        ┌───────────┬────────────┼────────────┬──────────────┐
        │            │            │            │              │
   auditor      variance-    cash-flow-   cash-flow-      payroll-schedule-
  (reviews an    analysis    builder-ar   builder-ap           builder
  existing model  (backward-      │            │                  │
  against          looking,       └─────────┬──┴──────────────────┘
  Foundation)      standalone)               │  shared spine
                                              │
                              ┌───────────────┴────────────────┐
                              │                                  │
                    capex-depreciation-builder          debt-amortization-builder
                        (overlay, if in scope)             (overlay, if in scope)
                              │                                  │
                              └───────────────┬──────────────────┘
                                               │
                                  master-cash-flow-builder
                        (orchestrates AR + AP + payroll, always;
                         capex and/or debt, where in scope)
```

**Foundation is a hard dependency for everything else.** Every builder, the Auditor, and Variance Analysis state it in their descriptions and refuse to proceed without it. Where a builder appears to contradict Foundation, Foundation wins and the conflict gets reported rather than resolved silently.

**A/R, A/P, and payroll share one spine.** Whichever module builds it first, the others read off it rather than rebuilding it. `master-cash-flow-builder` is what actually runs all of them together on that shared spine and reconciles their outputs into one workbook — it's an orchestration layer, not a sixth set of financial mechanics, so it never re-derives DSO, wage-base caps, depreciation methods, or amortization math; each module skill still owns its own domain in full.

## Building order

The forward-looking builders (A/R, A/P, payroll, capex, debt) each follow the same shape:

1. **Interview** — two phases: facts the modeler knows without opening a ledger, then sources and methodology.
2. **Classify and reconcile the sources** — by column shape, never by filename; reconcile before a single forecast cell is written.
3. **Choose the method** — where the module has more than one mechanical route (e.g. A/R's invoice-level vs. percentage-profile fork, A/P's timing-and-policy fork).
4. **Build** — assumptions tab → spine → schedule assumptions → roll forward → method mechanic.
5. **Checks** — on the shared `Checks` tab, per Foundation.

`master-cash-flow-builder` adds a scope question before any of that ("which of A/R, A/P, payroll, capex, debt does this engagement need?"), then runs one consolidated version of steps 1–2 across every in-scope module before building each one in turn, followed by `Consolidated_Cash_Flow` and `PL_Memo`.

`variance-analysis` and `financial-modeling-auditor` sit outside this forward-looking chain — variance analysis explains periods that already happened, and the auditor reviews a finished model rather than building one.

## Surface matters

Foundation's dynamic-array rule is surface-dependent and non-optional:

- **Claude for Excel** (live workbook) — full dynamic-array toolkit available and encouraged.
- **Claude.ai chat or Cowork** (file delivered as a download) — no spilling arrays. LibreOffice recalculation in the pipeline will corrupt them, sometimes silently.

Every builder asks which surface it's on before anything else.

## License

See the repository [LICENSE](../../LICENSE).
