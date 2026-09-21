# Seidman FP&A Skills

Financial modeling skills for Claude, from [Carl Seidman](https://github.com/carlmsu1).

These teach Claude to build and review financial models the way an experienced FP&A practitioner would — with the conventions, checks, and documentation discipline that survive a board meeting or a diligence room.

## Install

### Claude Cowork or Claude Desktop

1. Open the **Customize** menu and go to the **Plugins** tab.
2. Under **Personal plugins**, click **+** and choose **Add marketplace**.
3. Enter: `carlmsu1/seidman-fpa-skills`
4. Click **Sync**, then install **seidman-financial-modeling**.

The skills are then available automatically — Claude picks the right one based on what you ask.

### Claude for Excel, PowerPoint, and Word

Install the plugin as above. Inside the add-in, the **+** button below the chat input lists your plugins and the skills each one carries.

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

An installed plugin stays on the version it was installed at until it syncs. If the skill names you see don't match this README, you're on an older build — sync the marketplace.

## What you get

Hub-and-spoke: Foundation is the universal layer everything else depends on; the master skill is the hub; the Auditor and the four cash-flow modules are the spokes.

| Skill | What it does |
|---|---|
| **financial-modeling-foundation** | The universal standards every model follows: colour conventions, tab structure, sign conventions, hardcoding discipline, formula rules, source classification, reconciliation protocol, check-tab design. Load-bearing for everything below it. |
| **financial-modeling-auditor** | Audits an existing model — yours or a client's — by re-deriving figures, testing provenance and reconciliation, and reviewing structural hygiene. Reports findings; doesn't change your file. Works on models this plugin didn't build. |
| **variance-analysis** | Builds a flexed-budget variance bridge — actual vs. budget or a prior forecast — separating volume variance from rate/spending variance, with an optional price/volume/mix decomposition and an EBITDA bridge waterfall. Looks backward at completed periods. |
| **13-week-cash-flow-builder-ar** | The accounts receivable module of a 13-week direct-method cash flow forecast — aged and new-sales collections, aging waterfall, credit memos, uncollectibles by bucket, customer segmentation. |
| **13-week-cash-flow-builder-ap** | The accounts payable module of the same forecast — a consolidated payables roll-forward, holdback accrual and release, and payment prioritization under a cash constraint. |
| **13-week-cash-flow-builder-payroll** | A bottom-up, roster-driven payroll forecast — gross wages, employer payroll taxes with wage-base caps, benefits, PTO liability roll-forward, bonus accrual and payout. |
| **13-week-cash-flow-master** | The hub. Orchestrates the A/R, A/P, and payroll modules in sequence on one shared weekly spine, consolidates their outputs into a single combined cash roll-forward, and routes to variance-analysis once actuals exist. |
| **deconstruct-finance-workflow** | Turns a rough description of a recurring finance or accounting process — month-end close, flux analysis, reconciliations, board reporting — into a structured Workflow Requirements document an AI build step can act on. |

## Try it

Ask Claude something like:

> Build me a full 13-week cash flow forecast. I have an A/R aging, an A/P aging, and a payroll roster.

> Build me the A/R module of a 13-week cash flow forecast. I have an open A/R aging and a settlement history.

> Audit this model for me. [attach a workbook]

> Build a variance bridge for last month — actual vs. budget.

Claude will interview you before building — which files you have, which conventions you want, and which of several methods fits your data. That interview is the point: the questions are the ones a careful analyst would ask before touching a spreadsheet.

The first question the cash flow builder asks is the **Week 1 start date**. The model runs Monday-to-Sunday, and every date in it is derived from that anchor. If you can't answer it, the build stops there — by design. Building through the master skill, that question is asked once and shared by every module.

## Where these work best

The builders detect whether they're running in **Claude for Excel** (a live workbook) or in **Cowork / chat** (a file you download), and change technique accordingly. Both work. Excel gets the more capable version, because the full dynamic-array toolkit is available there.

## Status

Early — these skills are internally consistent and cross-checked against each other and against `financial-modeling-foundation`, but you should treat findings from real builds as expected refinements, not failures. [Open an issue](https://github.com/carlmsu1/seidman-fpa-skills/issues) with anything that does something unexpected.

## License

See [LICENSE](LICENSE).
