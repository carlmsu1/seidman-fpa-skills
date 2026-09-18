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
| **financial-modeling-foundation** | The universal standards every model follows: colour conventions, tab structure, sign conventions, hardcoding discipline, formula rules, source classification, reconciliation protocol, check-tab design. Load-bearing for every other skill below. |
| **financial-modeling-auditor** | Audits an existing model — yours or a client's — by re-deriving figures, testing provenance and reconciliation, and reviewing structural hygiene. Reports findings; doesn't change your file. |
| **variance-analysis** | Builds a flexed-budget variance bridge (actual vs. budget/prior forecast) — separates volume variance from rate/spending variance, with an optional price/volume/mix decomposition and an EBITDA bridge waterfall. |
| **cash-flow-builder-ar** | Builds the accounts receivable module of a monthly direct-method cash flow forecast — aging waterfall, collection curves, credit memos, uncollectibles. |
| **cash-flow-builder-ap** | Builds the accounts payable module of the same forecast, including payment prioritization when cash is constrained. |
| **payroll-schedule-builder** | Builds a roster-driven payroll forecast — computes at the payroll's true pay frequency, rolls up to a monthly spine. |
| **capex-depreciation-builder** | Builds an asset-by-asset capex and depreciation roll-forward (book methods: straight-line, declining balance, units-of-production). |
| **debt-amortization-builder** | Builds a tranche-by-tranche term loan amortization schedule — fixed or floating rate, fully amortizing, interest-only, or bullet. |
| **master-cash-flow-builder** | Orchestrates A/R, A/P, payroll, and optionally capex and debt, into one combined monthly cash flow model on a shared spine — one interview, one set of tabs, a `Consolidated_Cash_Flow` roll-forward, and a light `PL_Memo`. |

## Try it

Ask Claude something like:

> Build me a monthly cash flow forecast. I have an A/R aging, an A/P aging, and payroll census data.

> Audit this model for me. [attach a workbook]

> Build a variance bridge — actual vs. budget for Q3.

Claude will interview you before building — which files you have, which conventions you want, and which of several methods fits your data. That interview is the point: the questions are the ones a careful analyst would ask before touching a spreadsheet.

## Where these work best

The builders detect whether they're running in **Claude for Excel** (a live workbook) or in **Cowork / chat** (a file you download), and change technique accordingly. Both work. Excel gets the more capable version.

## Status

Early — these skills are internally consistent and cross-checked against each other and against `financial-modeling-foundation`, but you should treat findings from real builds as expected refinements, not failures. [Open an issue](https://github.com/carlmsu1/seidman-fpa-skills/issues) with anything that does something unexpected.

## License

See [LICENSE](LICENSE).
