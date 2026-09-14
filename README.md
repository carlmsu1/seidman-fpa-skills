# Seidman FP&A Skills

Financial modeling skills for Claude, from [Carl Seidman](https://github.com/carlmsu1).

These teach Claude to build and review financial models the way an experienced FP&A practitioner would — with the conventions, checks, and documentation discipline that survive a board meeting or a diligence room.

## Install

### Claude Cowork or Claude Desktop

1. Open the **Customize** menu and go to the **Plugins** tab.
2. Under **Personal plugins**, click **+** and choose **Add marketplace**.
3. Enter: `carlmsu1/seidman-fpa-skills`
4. Click **Sync**, then install **seidman-financial-modeling**.

The skills are then available in Cowork automatically — Claude picks the right one based on what you ask.

### Claude Code

```
/plugin marketplace add carlmsu1/seidman-fpa-skills
/plugin install seidman-financial-modeling@seidman-fpa
```

### Staying current

Updates reach you automatically. To pull them immediately:

```
/plugin marketplace update seidman-fpa
```

## What you get

| Skill | What it does |
|---|---|
| **financial-modeling-foundation** | The universal standards every model follows: colour conventions, tab structure, sign conventions, hardcoding discipline, formula rules, source classification, reconciliation protocol, check-tab design. Load-bearing for the others. |
| **financial-modeling-auditor** | Audits an existing model — yours or a client's — by re-deriving figures, testing provenance and reconciliation, and reviewing structural hygiene. Reports findings; doesn't change your file. |
| **13-week-cash-flow-builder-ar** | Builds the accounts receivable side of a 13-week direct-method cash flow forecast. |
| **13-week-cash-flow-builder-ap** | Builds the accounts payable side, including holdbacks and payment prioritization when cash is constrained. |

## Try it

Ask Claude something like:

> Build me a 13-week cash flow forecast. I have an A/R aging and about a year of settlement history.

> Audit this model for me. [attach a workbook]

Claude will interview you before building — which files you have, which conventions you want, and which of several methods fits your data. That interview is the point: the questions are the ones a careful analyst would ask before touching a spreadsheet.

## Where these work best

The builders detect whether they're running in **Claude for Excel** (a live workbook) or in **Cowork / chat** (a file you download), and change technique accordingly. Both work. Excel gets the more capable version.

## Status

**v0.1 — early.** These are internally consistent and cross-checked, but young. If a skill does something unexpected, [open an issue](https://github.com/carlmsu1/seidman-fpa-skills/issues) — that feedback is how they improve.

Known gap: there is no cash module yet. A/R produces receipts and A/P produces disbursements, but nothing here builds the cash roll-forward, non-trade disbursements, or the revolver.

## License

See [LICENSE](LICENSE).
