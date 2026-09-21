---
name: financial-modeling-auditor
description: "Audit an existing financial model in Excel — re-deriving key figures for correctness, testing provenance and reconciliation discipline, and reviewing structural/formatting hygiene against financial-modeling-foundation standards. Works on models Claude did not build (client files, diligence targets, inherited workbooks) as readily as on its own output. Use whenever the user says 'audit this model,' 'audit the model,' 'review this model,' 'check this model for errors,' or otherwise asks for quality assurance, a health check, a diligence review, or a second look at a financial model, cash flow forecast, budget, or valuation model. This skill REQUIRES financial-modeling-foundation — always load and follow it first, and ask the user to enable it if it's not available."
---

# Financial Modeling Auditor

Requires `financial-modeling-foundation`. The audit is performed *against* that skill's standards — don't invent standards not defined there. If the model being audited is a type covered by a dedicated builder skill, load that skill and also check the model against its model-specific requirements (line items, checks, structure). Where no builder skill covers the model type, audit against Foundation alone and say so in the report — do not improvise model-specific standards.

**The primary case is a model Claude did not build.** A client's workbook, a diligence target's file, something an analyst produced, a model inherited from a predecessor. A Claude-built model passes most hygiene checks by construction, so the yield there concentrates in the numeric and provenance sections. Do not assume the model came from these skills, and do not assume anything about it the file does not show.

This is a genuinely separate role from building. **Auditing is not modifying.** Report findings; only take remediation action if the modeler explicitly asks.

## Independence

Where the model *was* built by these skills, do not audit it by re-reading the builder skill and confirming each instruction was followed. A reviewer who checks the work against the same instructions the builder used inherits the builder's blind spots, and the audit becomes a formatting review wearing a correctness costume.

**Re-derive from the source data and from first principles.** Ask what the number should be, compute it independently, then compare. The builder skill tells you what the model *claims* to do; the audit tests whether it does.

## Before starting: establish scope

Ask the modeler, briefly:

> Three things before I start:
> 1. **What am I auditing against** — the Foundation standards alone, or is there a model type with its own builder skill I should also check?
> 2. **Do I have the source data** the model was built from? Without it I can test internal consistency but cannot verify the model against reality — those items get marked "cannot verify" rather than passed.
> 3. **How deep?** A structural pass, or full numeric re-derivation across the sample below?

Never audit silently against assumed standards.

---

# Scope

Perform all four. Sections 1 and 2 are the correctness audit; 3 and 4 are hygiene and judgment.

1. **Numeric spot-checks** — independently re-derive sampled figures to verify formula correctness, not just formula presence.
2. **Provenance, reconciliation and method-matching** — test whether the model's claims about itself hold.
3. **Structural / hygiene audit** — the model against every Foundation standard.
4. **Materiality weighting** — every finding sized against the model, not judged in the abstract.

---

# 1. Numeric spot-checks — where to sample

*"A sample" with no method produces an audit nobody can repeat and whose coverage nobody can state. Sample from the list below in order, and record which priorities were covered and which were not.*

Errors cluster in structurally predictable places. These are they:

| Priority | Where | Why it breaks there |
|---|---|---|
| 1 | **First and last forecast periods, and the opening column** | Boundary conditions. Where Foundation's opening column is present, test that it is excluded from every total and caught by no date-driven gather — a summed or gathered opening column double counts silently. Where it is absent, expect first-period exceptions and treat each as a finding |
| 2 | **Any column where a row's formula differs from its neighbours** | Either deliberate and undocumented, or the bug. Scan every row for pattern breaks before sampling anything else — the highest-yield single test in the audit |
| 3 | **Handover points between pools** | Where one source of a line runs out and another takes over: an aged pool giving way to new business, a catch-up ending, a ramp starting. False gaps and double counts live here |
| 4 | **Period boundaries inside the horizon** | Month-end and quarter-end flags in a weekly or daily model. Off-by-one is the norm, not the exception |
| 5 | **Both sides of any method or scenario switch** | A model offering two methods usually has one that was tested and one that was not |
| 6 | **The largest lines by absolute value** | Materiality. An error here outweighs a dozen elsewhere |
| 7 | **Anything the model's own checks do not cover** | The check set defines what the builder was worried about. The gaps define what they were not |

**How to re-derive.** For each sampled cell: state what the figure should be from the source data or the model's stated logic, compute it independently, then compare. Report the delta, not just pass/fail. A cell that is right for the wrong reason is a finding.

**Trace the reference, not just the result.** A formula returning a plausible number can still point at the wrong row, the wrong period, or the wrong tab. Follow at least one reference chain per sampled row back to its source.

---

# 2. Provenance, reconciliation and method-matching

These test the model's claims about itself. Cheap, high-yield, and the ones most often missing from an audit.

**Provenance.** Foundation requires inputs to be traceable. For a sample of assumptions: is there a note naming where the figure came from? Is a placeholder identified as a placeholder? An assumption presented with no stated source, sitting among assumptions that have one, reads as derived when it is not — the specific failure a diligence team finds.

**Reconciliation.** Where the model was built from multiple source files, were they tested against each other? Is every break either resolved or carried as a *disclosed* reconciling item? **An undisclosed plug is Critical.** It appears as a hardcoded adjustment, an unexplained difference row, or a total that ties only because something was forced.

**Method-matching.** Where a model documents which of several approaches it took, verify the formulas actually do that. Notes claiming one method while the formulas run another is Critical — the reader is being told something untrue about how the numbers were produced.

**Check-tab integrity.** Do the checks pass? More importantly: does each check test what its label claims? A check reading PASS because it compares a cell to itself is worse than no check, because it buys false confidence.

---

# 3. Structural / hygiene audit

Against every Foundation standard: color convention adherence; no hardcoded numbers inside formulas; no named ranges in Name Manager (Tables excepted); formula consistency across each row and period; correct tab order and naming; no merge-and-center; no nested IFs; no circularity or circularity breakers; appropriate use or correct surface-gating of dynamic array functions; no stray `ANCHORARRAY` references; the period horizon driven by a `Macro_Assumptions` input rather than hardcoded; presence and correct treatment of the opening column; presence and correctness of the Checks tab and the Cover tab's Change Log.

**A declared first-period exception is still a finding.** Where a model documents a variant formula in its first period rather than removing the need for one, bucket it as a Warning: it breaks formula consistency, and Foundation specifies the structural fix. Say what the fix is rather than only naming the breach. Do not wave it through because it was documented — a documented exception forces every future audit to carry a permanent carve-out, which is how real findings get missed.

**`ANCHORARRAY` warrants its own mention.** It is not a real Excel function. Its presence means the file has been through LibreOffice and genuine spill references have been rewritten — the file throws `#NAME?` the moment it opens in Excel 365. Always Critical, regardless of whether the model otherwise computes correctly.

---

# 4. Severity and materiality

Every finding gets a bucket, with the reasoning stated explicitly — not just the label.

- **Critical** — actively produces wrong numbers, breaks on the next update, or fails immediately under scrutiny. Hardcoded numbers inside formulas, mid-row formula inconsistency, circularity, an undisclosed plug, notes that misdescribe the method, a stray `ANCHORARRAY`.
- **Warning** — doesn't break the model today but creates real risk. A named range, missing check coverage for a material line, inconsistent color coding, an assumption with no stated provenance.
- **Suggestion** — cosmetic or best-practice polish that doesn't threaten correctness. Tab naming inconsistency, a formula `LET` would make more readable but which isn't wrong.
- **Cannot verify** — the finding can't be resolved from what's available: source data absent, an external link that can't be followed, a figure whose derivation isn't documented anywhere. **Say so plainly rather than forcing it into another bucket.** Overstating it as Critical misleads as much as burying it, and a diligence reader needs to know what was not tested. State what would be needed to resolve it.

**Weight severity by materiality.** A hardcoded figure inside a formula is Critical in principle, but a $50 hardcode in a $50M model and a $50 hardcode in a $500K model are not the same finding. State the magnitude alongside the bucket — the amount at risk, or the line affected — so the modeler can triage rather than working an undifferentiated list. Where a finding is structurally Critical but immaterial in amount, say both.

---

# Output

**Default: a separate audit report, not the workbook.** The audit does not modify the file it audits — writing findings into the model is itself a modification, and a client or diligence file often has no Checks tab to write to.

Structure the report as:

```
Summary        — model audited, standards applied, date, what was and wasn't covered
Scorecard      — count by severity, and whether the model's own checks pass
Critical       — findings, in materiality order
Warning        — findings, in materiality order
Suggestion     — findings
Cannot verify  — items, with what would be needed to resolve each
Coverage       — which sampling priorities were tested, which were not, and why
```

Every finding carries: **location** (tab and cell), **the finding**, **severity and why it's bucketed there**, **materiality**, and **suggested remedy**.

The Coverage section is not optional. An audit that doesn't state its own limits invites the reader to assume it was exhaustive.

**Where the modeler wants findings in the workbook instead**, offer it — numeric spot-check results as additional PASS/FAIL rows in the existing checks section, structural findings as a new section below it. Do this only when asked, and say plainly that it modifies the file.

---

# Remediation

The audit reports and explains. It does not change anything in the workbook by default.

If the modeler asks Claude to remedy a finding, offer **both** and let them choose:

(a) an explanation of how they could fix it themselves, and
(b) an offer for Claude to make the fix directly.

Never make a fix without the modeler explicitly choosing (b) **for that specific finding**. Don't bundle unrelated fixes into one approval — remediate what was actually asked for. After any fix, re-run the affected checks and report whether they now pass.
