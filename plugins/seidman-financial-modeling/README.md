# Seidman Financial Modeling

Standards and builders for financial models in Excel, written to hold up to board-room and private-equity-diligence scrutiny.

**Status: v0.1 — unvalidated.** These skills are internally consistent and cross-checked against each other, but have not yet been used to build a model end to end. Treat findings from the first real builds as expected, not as failures.

## What's here

| Skill | Role |
|---|---|
| `financial-modeling-foundation` | Universal standards for **every** model: colour conventions, tab structure, sign conventions, hardcoding discipline, formula rules, circularity, dynamic arrays and the surface rule, the opening column, source classification and provenance, reconciliation protocol, check-tab design, documentation. |
| `financial-modeling-auditor` | Independent audit of an existing model against Foundation — numeric re-derivation, provenance and reconciliation testing, structural hygiene, severity and materiality. Works on models Claude did not build. |
| `13-week-cash-flow-builder-ar` | The accounts receivable module of a 13-week direct-method cash flow forecast. |
| `13-week-cash-flow-builder-ap` | The accounts payable module of the same forecast, including holdbacks and payment prioritization under a cash constraint. |

## How they fit together

```
                financial-modeling-foundation
                 (universal layer — always first)
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
   builder-ar            builder-ap        financial-modeling-auditor
  (A/R module)          (A/P module)      (reviews the finished model
        │                     │            against Foundation + whichever
        └────── shared ───────┘            builder applies)
            header/footer spine
```

**Foundation is a hard dependency.** Both builders and the Auditor state it in their descriptions and refuse to proceed without it. Where a builder appears to contradict Foundation, Foundation wins and the conflict gets reported rather than resolved silently.

**The two builders share one spine.** Build the header and footer spine once; whichever module runs second reads off it rather than rebuilding it.

## Known gap

**There is no cash module.** A/R produces receipts and A/P produces disbursements, but nothing here builds the cash roll-forward, non-trade disbursements (payroll, taxes, debt service, capex), the minimum cash floor, or the revolver.

This matters most for A/P's pay-to-constraint policy, which explicitly runs *after* the cash build and needs opening cash and A/R receipts already in place. Until a cash module exists, that policy assumes an artifact this plugin does not produce.

## Building order

Both builders follow the same five phases:

1. **Interview** — two phases: facts the modeler knows without opening a ledger, then sources and methodology.
2. **Classify and reconcile the sources** — by column shape, never by filename; reconcile before a single forecast cell is written.
3. **Choose the method** — A/R has one fork (invoice-level vs. percentage profile); A/P has two (the same timing fork, plus payment policy: to behaviour, to terms, or to constraint).
4. **Build** — `Macro_Assumptions` globals → the spine → schedule assumptions → roll forward → method mechanic.
5. **Checks** — on the `Checks` tab, per Foundation.

Steps 1 and 2 of the build phase depend only on interview answers, so they can be built while waiting for source extracts.

## Surface matters

Foundation's dynamic-array rule is surface-dependent and non-optional:

- **Claude for Excel** (live workbook) — full dynamic-array toolkit available and encouraged.
- **Claude.ai chat or Cowork** (file delivered as a download) — no spilling arrays. LibreOffice recalculation in the pipeline will corrupt them, sometimes silently.

Both builders ask which surface they're on before anything else.

## Open items

- The **Transaction Adjustment** formula resolves to the Friday of the *prior* week. Whether that is the intended midpoint convention has not been confirmed.
- The **two-phase interview** in each builder was constructed from the assumptions lists and derivation tables rather than transcribed from practice. Review before relying on it.
- Neither builder has been run against real client data.

## Changelog

**v0.1** — Initial version. Four skills, cross-referenced, validated for internal consistency, build-sequence resolution, and input coverage. Not yet validated by use.
